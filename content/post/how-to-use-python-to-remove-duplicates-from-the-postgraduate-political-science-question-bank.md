---
title: "如何使用Python去重考研政治选择题库"
date: 2024-05-20T21:39:12+08:00
draft: false
tags: ["python", "anki", "bert", "nlp", "ai", "pol", "article"]
math: true
---

在准备考研政治时，积累大量的选择题是非常重要的。然而，重复的题目不仅浪费时间，还影响复习效率。考研政治是考研复习中的重要一环，大量的选择题可以帮助考生更好地掌握知识点，但在积累题库的过程中，往往会出现大量重复的题目。这些重复题目不仅占用存储空间，还浪费考生宝贵的复习时间，影响复习效率。因此，对题库进行去重变得尤为重要。本文将介绍两种基于Python的去重方法，帮助您轻松清理重复题目，从而更高效地备战考研。通过去重，我们可以确保题库的独特性，提高复习的针对性和效率，为考生提供更好的复习体验和更高的考试通过率。

为了解决考研政治选择题库中的重复问题，我们采用了两种不同的去重方法：基于MD5哈希和基于BERT相似度。这两种方法各有优缺点，结合使用可以更全面地清理题库中的重复题目。

<!--more-->

## 方法一：基于MD5哈希去重

MD5是一种常用的消息摘要算法，可以将任意长度的数据转换为固定长度的哈希值。通过对题目的各个部分（选项、备注、问题）进行预处理，然后计算它们的MD5哈希值，我们可以快速检测到重复的题目。对于重复的文本内容，MD5算法会生成相同的哈希值，因此，通过计算题目的MD5哈希值，可以快速识别并删除重复的题目。此方法的优点是速度快、实现简单，但对于相似但不完全相同的题目（例如选项顺序不同）可能无法检测到。

**实现步骤**

1. **预处理字符串**：为了确保对比的准确性，需要对题目内容进行标准化处理。具体步骤包括：转换为小写、移除HTML标签、替换连续的空白字符为单个空格、移除前后空白。
2. **计算MD5哈希值**：将标准化后的题目各部分（选项、备注、问题）进行编码，并计算其MD5哈希值，再将这些哈希值转换为整数并相加，生成唯一的哈希值。
3. **去重**：使用集合（set）来存储已经处理过的哈希值。如果某个题目的哈希值已经存在于集合中，则说明它是重复的，删除该题目；否则，将其哈希值添加到集合中。

```python
import hashlib
import regex
from bs4 import BeautifulSoup
from main import invoke

s_options = set()
s_remark = set()
s_question = set()

def process_string(str_):
    str_ = str_.lower()
    str_ = BeautifulSoup(str_, "html.parser").get_text()
    str_ = regex.sub(r"[^\p{Script=Han}\p{Script=Latin}\p{Number}]", "", str_)
    str_ = str_.strip()
    return str_

def calculate_md5(strings):
    result = 0
    for string in strings:
        encoded_string = string.encode()
        md5_hash = hashlib.md5(encoded_string)
        int_hash = int(md5_hash.hexdigest(), 16)
        result += int_hash
    return result

if __name__ == "__main__":
    notesId = invoke("findNotes", query="deck:考研政治")
    notesInfo = invoke("notesInfo", notes=notesId)
    for noteInfo in notesInfo:
        fields = noteInfo["fields"]
        strs = [
            process_string(fields["A"]["value"]),
            process_string(fields["B"]["value"]),
            process_string(fields["C"]["value"]),
            process_string(fields["D"]["value"]),
            process_string(fields["E"]["value"]),
            process_string(fields["F"]["value"]),
            process_string(fields["G"]["value"]),
            process_string(fields["H"]["value"]),
            process_string(fields["I"]["value"]),
        ]
        md5v = str(calculate_md5(strs))
        if md5v in s_options:
            invoke("deleteNotes", notes=[noteInfo["noteId"]])
        else:
            s_options.add(md5v)
        remark = process_string(fields["Remark"]["value"])[1:]
        if len(remark) > 16 and remark in s_remark:
            invoke("deleteNotes", notes=[noteInfo["noteId"]])
        else:
            s_remark.add(remark)
        question = process_string(fields["Question"]["value"])
        md5v = str(calculate_md5(question))
        if md5v in s_question:
            invoke("deleteNotes", notes=[noteInfo["noteId"]])
        else:
            s_question.add(md5v)
```

#### 方法二：基于BERT相似度去重

BERT（Bidirectional Encoder Representations from  Transformers）是一种强大的自然语言处理模型，能够捕捉文本的上下文语义信息，用于计算文本之间的语义相似度。通过使用BERT模型对题目进行嵌入，然后利用HNSWlib库进行高效的近似最近邻搜索，我们可以检测到语义上相似的题目并删除。此方法的优点是可以检测到语义相似的题目，但需要更多的计算资源和时间。

**实现步骤**

1. **预处理字符串**：同样需要对题目内容进行标准化处理。
2. **计算相似度**：使用预训练的BERT模型对题目进行嵌入，将文本转化为向量，并利用余弦相似度计算两个文本之间的相似性。
3. **构建索引**：利用HNSWlib库构建高效的最近邻搜索索引，将每个题目的嵌入向量添加到索引中。
4. **去重**：对于每个新题目，计算其嵌入向量，并在索引中查找最相似的题目。如果相似度超过阈值（例如0.9），则删除该题目，否则将其添加到索引中。

```python
import time
from datetime import datetime, timedelta
import hnswlib
from bs4 import BeautifulSoup
from sentence_transformers import SentenceTransformer, util
from main import invoke

model = SentenceTransformer("paraphrase-multilingual-MiniLM-L12-v2")
dim = 384
num_elements = 10000
p = hnswlib.Index(space="cosine", dim=dim)
p.init_index(max_elements=num_elements, ef_construction=200, M=16)
p.set_ef(50)
p.set_num_threads(4)
id_to_remark = {}

def process_string(str_):
    str_ = str_.lower()
    str_ = BeautifulSoup(str_, "html.parser").get_text()
    str_ = str_.strip()
    return str_

def add_remark_to_index(remark, index, id_to_remark):
    embedding = model.encode(remark)
    remark_id = len(id_to_remark)
    id_to_remark[remark_id] = remark
    index.add_items(embedding, [remark_id])
    return embedding

def get_card_abstract(fields):
    question = "问题:" + process_string(fields["Question"]["value"])
    options = "\nA." + process_string(fields["A"]["value"])
    options += "\nB." + process_string(fields["B"]["value"])
    options += "\nC." + process_string(fields["C"]["value"])
    options += "\nD." + process_string(fields["D"]["value"])
    explanation = "\n解析:" + process_string(fields["Remark"]["value"])
    return question + options + explanation

if __name__ == "__main__":
    notesId = invoke("findNotes", query="deck:25考研::政治 note:选择题 (is:new -(is:buried OR is:suspended))")
    notesInfo = invoke("notesInfo", notes=notesId)
    total_notes = len(notesInfo)
    start_time = time.time()

    for index, noteInfo in enumerate(notesInfo):
        fields = noteInfo["fields"]
        abstract = get_card_abstract(fields)

        if len(abstract) > 16:
            embedding = model.encode(abstract)
            if p.get_current_count() > 0:
                labels, distances = p.knn_query(embedding, k=1)
                if distances[0][0] < 0.1:
                    invoke("deleteNotes", notes=[noteInfo["noteId"]])
                    print(f"{noteInfo['noteId']} deleted due to similarity with {id_to_remark[labels[0][0]]}")
                    continue
            add_remark_to_index(abstract, p, id_to_remark)

        current_time = time.time()
        elapsed_time = current_time - start_time
        current_count = p.get_current_count()

        avg_time_per_note = elapsed_time / (index + 1)
        adjustment_factor = 1 + (current_count / (index + 1))
        adjusted_avg_time_per_note = avg_time_per_note * adjustment_factor

        remaining_notes = total_notes - (index + 1)
        remaining_time = remaining_notes * adjusted_avg_time_per_note
        eta = datetime.now() + timedelta(seconds=remaining_time)
        print(f"Processed {index + 1}/{total_notes} notes. ETA: {eta.strftime('%Y-%m-%d %H:%M:%S')}")
```

在实际应用中，基于MD5哈希的方法能够快速去除完全重复的题目。此方法实现简单，运行速度快，适用于题库中存在大量完全重复题目的情况。然而，对于内容相似但不完全相同的题目（例如选项顺序不同），MD5哈希方法无法检测到。

基于BERT相似度的方法则可以检测到语义相似的题目。通过BERT模型计算文本的语义嵌入，结合HNSWlib进行近似最近邻搜索，能够有效识别并删除相似题目。虽然此方法计算量较大，运行速度相对较慢，但适用于需要高精度去重的场景。

综上所述，MD5哈希适合快速初步去重，而BERT相似度适合精细去重。结合这两种方法，可以在保证效率的同时，最大限度地去除重复题目。未来，可以尝试引入更多的自然语言处理技术，如更先进的文本嵌入模型或混合方法，以进一步提升去重效果。
