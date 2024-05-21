---
title: "使用 Puppeteer 渲染并提取 Anki 卡片内容的方法"
date: 2024-05-21T15:05:44+08:00
draft: false
tags: ["article", "anki", "typescript", "puppeteer", "nodejs", "jsdom"]
math: true
---

Anki 是一款流行的学习软件，通过卡片记忆法帮助用户有效地记忆信息。然而，许多Anki卡片使用了混淆技术，以防止用户直接导出卡片内容，从而保护版权和提高用户使用平台的依赖性。这种混淆技术使得用户在尝试提取卡片内容时遇到困难，无法直接获取完整的卡片信息。为了解决这一问题，我们可以使用 Puppeteer，这是一款强大的无头浏览器自动化工具，通过渲染卡片的HTML内容来绕过混淆技术，获取所需的卡片数据。

Anki卡片通常包含混淆技术，如CSS样式的隐藏、JavaScript的动态加载等。这些技术使得卡片内容在普通的HTML解析工具中无法正确显示。直接导出卡片内容的方法无法应对这些混淆技术，需要借助更高级的工具来处理。Puppeteer 作为一个无头浏览器，可以完全渲染网页，就像用户在浏览器中打开网页一样。通过这种方式，我们能够准确地获取到卡片的真实内容。此外，结合 JSDOM，可以进一步解析渲染后的HTML，提取具体的选项和答案信息。这种方法不仅能够绕过混淆技术，还可以确保提取到的数据完整准确，为后续的数据处理和分析提供了可靠的基础。通过本文，我们将详细介绍如何使用 Puppeteer 渲染并提取 Anki 卡片内容的方法，帮助用户有效地获取所需信息。

<!--more-->

我们将使用以下工具和库来实现Anki卡片内容的提取。首先是Node.js，它是一个基于Chrome  V8引擎的JavaScript运行环境，适用于服务器端开发。其次是Puppeteer，这是一个Node库，提供了一个高层次的API来控制无头Chrome或Chromium，可以模拟用户在浏览器中的操作。最后是JSDOM，一个轻量级的JavaScript实现，用于解析和操作HTML文档。通过结合这三个工具，我们能够自动化渲染Anki卡片的内容并提取其中的关键信息，克服混淆技术带来的障碍。

## 代码实现

```typescript
import {addNote, findNotes, notesInfo} from "./ankifetch";
import fs from 'fs/promises';
import puppeteer from "puppeteer";
import {JSDOM} from "jsdom";

async function render(xuhao:string,biji:string,timu:string,xuanxiang:string,daan:string,yuanshujiexi:string,kdywhg:string,kdfpzy:string,kdnsbj:string,dingwei:string):Promise<string> {
    // Read the content of the front.html file
    let template = await fs.readFile('pol.html', 'utf-8');
    // Replace the placeholder in the front template
    template = template.replace('{{序号}}', xuhao);
    template = template.replace('{{笔记}}', biji);
    template = template.replace('{{题目}}', timu);
    template = template.replace('{{选项}}', xuanxiang);
    template = template.replace('{{答案}}', daan);
    template = template.replace('{{原书解析}}', yuanshujiexi);
    template = template.replace('{{考点原文回顾}}', kdywhg);
    template = template.replace('{{考点复盘指引}}', kdfpzy);
    template = template.replace('{{考点浓缩笔记}}', kdnsbj);
    template = template.replace('{{定位}}', dingwei);
    // Save modified front template to temp file
    const path = '/tmp/pol.html';
    await fs.writeFile(path, template);

    const browser = await puppeteer.launch();
    try {
        const page = await browser.newPage();
        // Load the front template file with page.goto
        await page.goto(`file://${path}`);
        // Wait for the first <li class="option"> to have content, with a timeout of 15 seconds
        await page.waitForFunction(
            () => {
                const firstOption = document.querySelector<HTMLLIElement>('.options .option');
                return firstOption !== null && firstOption.textContent !== null && firstOption.textContent.trim() !== '';
            },
            { timeout: 15000 } // 15 seconds timeout
        );
        return await page.content();
    } catch (e) {
        console.error(e)
        return Promise.reject(e)
    } finally {
        await browser.close()
    }
}

function removePattern(input: string): string {
    const pattern = /^\s*\d+\s*[.,，．]\s*/;
    return input.replace(pattern, '');
}

function convertNumbersToLetters(input: string): string {
    // 过滤出数字字符
    const numbersOnly = input.replace(/[^0-9]/g, '');

    // 将数字字符转换为对应的大写字母
    const letters = numbersOnly.split('').map(char => {
        const num = parseInt(char, 10);
        return String.fromCharCode(num + 64); // 'A' 的 ASCII 码是 65，因此 1 -> 65 ('A')
    });

    // 将转换后的字符数组连接成字符串
    return letters.join('');
}

function extractOptionsFromHTML(htmlString: string): string[] {
    // 使用 JSDOM 解析 HTML 字符串
    const dom = new JSDOM(htmlString);

    // 获取包含选项的 div 元素
    const optionsDiv = dom.window.document.getElementById('back-options');

    // 如果没有找到选项 div，则返回空数组
    if (!optionsDiv) {
        return [];
    }

    // 获取所有 class 为 'option' 的 li 元素
    const optionElements = optionsDiv.getElementsByClassName('option');

    // 提取每个 li 元素的文本内容
    return Array.from(optionElements).map(option => option.textContent?.trim() ?? '');
}

(async () => {
    let notes: number[] = await findNotes({query: 'deck:2025名师习题集【腿徐米余杨】（肖1000赠品系列）'});
    const totalNotes = notes.length;
    const startTime = Date.now();
    for (let i = 0; i < totalNotes; i++) {
        const noteId = notes[i];
        try{
            console.log(noteId)
            let noteInfo = (await notesInfo({notes: [noteId]}))[0];
            //console.log(noteInfo)

            let xuhao = noteInfo.fields['序号'].value;
            let biji = noteInfo.fields['笔记'].value;
            let timu = noteInfo.fields['题目'].value;
            let xuanxiang = noteInfo.fields['选项'].value;
            let daan = noteInfo.fields['答案'].value;
            let yuanshujiexi = noteInfo.fields['原书解析'].value;
            let kdywhg = noteInfo.fields['考点原文回顾'].value;
            let kdfpzy = noteInfo.fields['考点复盘指引'].value;
            let kdnsbj = noteInfo.fields['考点浓缩笔记'].value;
            let dingwei = noteInfo.fields['定位'].value;
            let tags = noteInfo.tags;

            let res = await render(xuhao,biji,timu,xuanxiang,daan,yuanshujiexi,kdywhg,kdfpzy,kdnsbj,dingwei);
            let options = extractOptionsFromHTML(res)

            //console.log(res)


            const newNote = {
                deckName: "考研政治",
                modelName: "选择题",
                fields: {
                    "Question": removePattern(timu).trim(), // 题目
                    "A": options[0] ?? "",
                    "B": options[1] ?? "",
                    "C": options[2] ?? "",
                    "D": options[3] ?? "",
                    "E": options[4] ?? "",
                    "F": options[5] ?? "",
                    "G": options[6] ?? "",
                    "H": options[7] ?? "",
                    "I": options[8] ?? "",
                    "Answer": convertNumbersToLetters(daan),
                    "Remark": yuanshujiexi.trim(),
                },
                options: {
                    allowDuplicate: false,
                    duplicateScope: "deck",
                    duplicateScopeOptions: {
                        deckName: "考研政治",
                        checkChildren: false,
                        checkAllModels: false
                    }
                },
                tags: tags, // 直接使用现有 tags
                audio: [], // 如有需要，根据 selCard 添加音频
                video: [], // 如有需要，根据 selCard 添加视频
                picture: [] // 如有需要，根据 selCard 添加图片
            };
            console.log(newNote)
            // 调用 addNote 方法添加 note
            const addedNoteId = await addNote({note: newNote});
            console.log('Added note ID:', addedNoteId);

            // 计算并输出ETA
            const currentTime = Date.now();
            const elapsedTime = currentTime - startTime;
            const avgTimePerNote = elapsedTime / (i + 1);
            const remainingTime = avgTimePerNote * (totalNotes - (i + 1));
            const eta = new Date(currentTime + remainingTime);
            console.log(`ETA: ${eta.toLocaleString()}`);
        }catch(e){
            console.error(e)
        }
    }
})()
```

`render`函数的主要作用是渲染HTML模板并获取渲染后的内容。具体步骤如下：

- 读取模板文件：通过`fs.readFile`读取`pol.html`模板文件内容。
- 替换占位符：使用字符串替换函数，将模板中的占位符替换为实际的数据。
- 保存模板文件：将修改后的模板文件保存到临时路径。
- 启动无头浏览器：使用Puppeteer启动一个无头浏览器实例。
- 加载模板文件：通过`page.goto`方法加载保存的模板文件。
- 等待内容加载：通过`page.waitForFunction`方法等待页面内容加载完成。
- 返回页面内容：获取并返回渲染后的HTML内容。
- 关闭浏览器：确保在完成操作后关闭浏览器实例。

`removePattern`函数用于去除题目字符串中的序号部分。具体步骤如下：

- 定义正则表达式模式：匹配以数字开头，后跟逗号、句号等符号的序号部分。
- 替换序号：使用`replace`方法将匹配到的序号部分替换为空字符串。

`convertNumbersToLetters`函数将数字字符串转换为对应的字母选项。具体步骤如下：

- 过滤数字字符：使用正则表达式提取字符串中的数字部分。
- 转换为字母：将每个数字转换为对应的字母（1 -> A, 2 -> B, 等）。
- 连接字母：将转换后的字母数组连接为一个字符串。

**执行流程和主要逻辑：**

1. 使用`findNotes`找到所有符合查询条件的Anki卡片。
2. 遍历每个卡片，获取其详细信息。
3. 调用`render`函数渲染卡片HTML并提取选项内容。
4. 使用`removePattern`处理题目字符串。
5. 使用`convertNumbersToLetters`转换答案字符串。
6. 创建新的Anki卡片对象，并调用`addNote`方法将其添加到目标卡组中。
7. 计算并输出ETA，展示任务预计完成时间。

该流程确保每个Anki卡片都能被正确渲染和提取，并最终创建新的卡片供用户使用。

在使用Puppeteer渲染并提取Anki卡片内容的过程中，可能会遇到一些常见问题。首先是页面加载超时问题，这可能由于网络速度或页面复杂性引起。为解决此问题，可以增加`waitForFunction`的超时时间或者优化模板文件以减少加载时间。其次，Puppeteer在某些环境中可能无法正常启动，例如没有安装Chrome或Chromium的服务器。为此，可以在服务器上安装必要的浏览器或者使用Puppeteer提供的无头浏览器模式。另外，处理大量卡片时，可能会出现内存泄漏或资源耗尽的问题，建议在处理每个卡片后关闭并重新启动浏览器实例。对于JSDOM解析选项内容，确保HTML结构正确且ID、类名匹配，否则提取可能失败。最佳实践包括定期检查并更新依赖库，确保代码兼容性和性能，以及在本地环境中充分测试代码。

通过结合Node.js、Puppeteer和JSDOM，我们能够有效地绕过Anki卡片的混淆技术，自动化渲染并提取卡片内容。这一方法不仅确保了数据的完整性和准确性，还提供了一种灵活、可扩展的解决方案，适用于各类教育技术应用场景。
