---
date: '2025-05-02T17:16:19+08:00'
draft: false
summary: '记录了如何使用Tampermonkey脚本和百度文心AI为在线作业批改（特别是简答题）提供自动化辅助，以减轻教师重复性评分和评语工作的负担。'
title: '用Tampermonkey和AI为在线作业批改减负'
tags: [ "tampermonkey", "userscript", "ai", "baidu-ernie", "llm", "homework-grading", "grading-automation", "education-technology", "edtech", "javascript", "dom-manipulation", "api-integration", "gm-xmlhttprequest", "cors-workaround", "prompt-engineering", "json", "browser-automation", "teacher-tools" ]
---

今天和大家分享一个有点特别的编程项目——它源于一个简单的想法：帮我妈妈减轻一些工作负担。

我妈妈是一位老师，随着在线教育的普及，她的很多工作，包括作业批改，都转移到了线上。这无疑提高了教学的灵活性，但也带来了一些新的挑战。尤其是批改某些线上平台的作业，重复性的操作相当多。

这次的目标是她教学平台上的“简答题”批改环节。这类题目往往需要老师阅读学生的回答，然后给出分数和评语。虽然最终的判断和个性化反馈无可替代，但在初步评分和基础评语生成上，似乎有自动化的空间。

于是，一场结合了浏览器脚本、DOM操作、API调用和一点点人工智能的自动化探索之旅开始了。

## 从浏览器控制台开始

万事开头难，最直接的想法就是：“能不能在浏览器里运行一段代码，模拟鼠标点击和键盘输入？”
答案是肯定的。浏览器开发者工具（按F12打开）中的“控制台”（Console）就是一个强大的武器。

### 定位目标

要实现自动化，首先得让代码“认识”页面上的元素。就像人批改作业需要找到题目、答题框、分数框、评语框和提交按钮一样，代码也需要通过特定的“地址”——也就是DOM选择器——来找到这些元素。

开始探索的第一步，是仔细研究一个典型作业批改页面的HTML结构。通过开发者工具，我们发现了一些规律性的设计：每个简答题的内容和答题区域，通常被整个包裹在一个列表项
`<li>` 元素之内。为了区分题目类型或为每个题目提供唯一标识，这些 `<li>` 元素往往带有特定的属性，例如 `data-questiontype="5"`
可能用来标记这是一个简答题，同时通常还会有一个独特的 `id` 属性。

在每个代表题目的 `<li>` 元素内部，可以找到用于输入分数的 `<input type="number">` 标签。这个输入框不仅可以通过 `max`
属性得知该题的满分值，其 `name` 属性（往往是像 `questions[0].studentScore` 这样的格式）也暗示了后台处理数据时可能的索引方式。

评语功能的实现则更显动态。初始状态下，页面上只显示一个“点评”按钮，这通常是一个 `<span>` 标签（例如带有 `class="modify"`
）。用户需要点击这个按钮后，才会动态地显示出用于输入评语的文本框，通常是一个 `<textarea>` 元素（可能带有
`class="teacherWrite comments"` 等类名），以及一个用于确认评语输入的“完成”按钮（可能是一个 `<div>` 标签）。

最后，在页面的底部，一般会存在一个全局的操作按钮，例如“提交并批阅下一个”，它的作用应该是将当前页面上所有题目的批改结果（分数和评语）一次性保存，并加载下一个需要批改的内容。对这些关键元素的定位和理解，是后续自动化脚本设计的基础。

### 模拟操作

明确了需要交互的关键页面元素后，自动化的基本流程也逐渐清晰起来。首先，需要让脚本识别出页面上所有需要处理的简答题。这可以通过
`document.querySelectorAll` 方法，结合之前分析得到的选择器（例如 `li[data-questiontype="5"]`）来实现，从而获得一个包含所有简答题
`<li>` 元素的列表。

接下来，脚本需要按顺序处理这个列表中的每一个题目元素，这通常意味着一个循环操作。在循环的每一步中，针对当前的题目，脚本需要执行两个核心任务：填写分数和添加评语。

对于填写分数，初步的想法是找到题目内部的分数输入框，读取其 `max` 属性以获取满分值，然后直接将输入框的 `value`
设置为这个满分值。当然，这只是一个基础策略，后续可以根据更复杂的逻辑进行调整，但作为起点是可行的。

填写评语的操作则稍微复杂一些，因为它涉及到动态元素的出现。脚本需要先找到并模拟点击“点评”按钮。点击之后，不能立刻去寻找评语框，而必须
**加入一个等待环节**，因为评语输入框和“完成”按钮是动态加载或显示的，需要给页面一些反应时间。等待结束后，脚本再定位到新出现的评语
`<textarea>` 元素，将其 `value` 设置为一个预先准备好的通用评语文本。最后，找到并模拟点击对应的“完成”按钮，以确认这条评语的输入。

为了让整个自动化过程更稳定，并且更贴近人类的操作习惯，在处理完一个题目的所有步骤后，加入一个短暂的延时是很有必要的。这可以避免因操作过快导致页面脚本响应不及，或者触发某些网站的反爬虫机制。

当所有题目都按照上述流程处理完毕后，理论上最后一步应该是模拟点击页面底部的那个全局提交按钮，将所有的批改结果保存下来。不过，考虑到自动化的风险和给老师留出最终检查的机会，这一步可以暂时设为待定，或者让用户手动触发。这个基于直接DOM操作的初步设想，为后续的编码实现奠定了基础。

这个初步设想主要依赖于**直接的DOM操作**（找到元素 -> 修改属性/触发点击）。在浏览器控制台里，这通常是可行的。

```javascript
// 伪代码示例：初步设想
function gradeShortAnswer(questionElement) {
	// 找到分数框并设置为满分
	const scoreInput = questionElement.querySelector('input.student-score');
	const maxScore = scoreInput?.max;
	if (scoreInput && maxScore) {
		scoreInput.value = maxScore;
		console.log(`Set score for ${questionElement.id} to ${maxScore}`);
		// 触发事件，让页面知道分数变化（重要！）
		scoreInput.dispatchEvent(new Event('input', {bubbles: true}));
		scoreInput.dispatchEvent(new Event('change', {bubbles: true}));
	}

	// 找到“点评”按钮并点击
	const commentButton = questionElement.querySelector('.comment span.modify');
	if (commentButton) {
		commentButton.click();
		// 需要等待评语框出现...
		setTimeout(() => {
			const commentArea = questionElement.querySelector('.comment textarea.teacherWrite');
			const confirmButton = questionElement.querySelector('.comment div.confirm');
			if (commentArea && confirmButton) {
				commentArea.value = "同学回答得不错！"; // 预设评语
				confirmButton.click();
				console.log(`Comment added for ${questionElement.id}`);
			}
		}, 1000); // 假设等待1秒
	}
}

// 获取所有简答题并处理
// document.querySelectorAll('#shiti-content li.subjective[data-questiontype="5"]')
//    .forEach(el => gradeShortAnswer(el));

// 注意：实际应用需要更复杂的异步处理和错误处理
```

这个思路看起来很美好，但在实践中，尤其是在复杂的、动态加载内容的现代网页上，往往会遇到第一个拦路虎。

## 探索API之路（及富文本编辑器的坑）

在实际的教学平台（包括之前探索的讨论区场景和现在的作业批改场景）中，评语输入框往往不是一个简单的 `<textarea>`，而是一个*
*富文本编辑器**（Rich Text Editor），比如我们熟悉的UEditor、CKEditor、TinyMCE等。

这些编辑器通常会在原始的 `<textarea>`（有时甚至是一个`<script>`标签）位置渲染出一个复杂的`<iframe>`或者带有
`contenteditable`属性的`<div>`，并提供一个工具栏。

然而，这种直接操作简单`<textarea>`的设想，在面对实际教学平台常用的富文本编辑器时，便遇到了挑战。这些编辑器的引入，使得原本简单的交互变得复杂起来。首先，目标元素的
**DOM结构发生了改变**。我们不再能直接操作一个简单的文本输入框，而是需要深入到编辑器生成的、通常更为复杂的结构中，比如一个内嵌的
`<iframe>`元素，并需要进一步定位到`<iframe>`内部的`<body>`元素，或者是一个设置了`contenteditable`属性的`<div>`。

其次，更关键的问题在于**对API的依赖**。富文本编辑器通常有自己的一套JavaScript API来管理内容和状态。如果我们绕过API，直接修改
`<iframe>`的`innerHTML`或者`contenteditable`元素的`innerText`
，编辑器自身的内部状态可能并不会随之更新。这带来的直接后果是，当触发表单提交或者点击“完成”按钮这类保存操作时，编辑器可能仍然认为内容是空的或者未改变，因为它依赖其API调用的方式来同步和获取最终内容（常常是同步到一个隐藏的表单域中）。因此，仅仅修改了视觉呈现，并不意味着数据能被正确捕获。

最后，**实例管理**
也成了一个需要考虑的因素。在一个包含多个简答题的页面上，通常每个题目的评语框都会是一个独立的富文本编辑器实例。这意味着，如果我们想通过API来操作，就必须能够准确地识别并获取到当前正在处理的那个题目的编辑器实例对象，才能调用其特定的方法（如设置内容、获取焦点等），这无疑增加了脚本逻辑的复杂度。这些问题的出现，表明直接的DOM操作可能不足以应对富文本编辑器的场景，需要探索调用编辑器API的可能性。

面对富文本编辑器带来的挑战，我们自然想到了尝试使用编辑器本身提供的API来进行交互，这通常是更规范和可靠的方式。以常见的UEditor为例，其标准的操作流程大致是这样的：首先，需要准确识别出目标编辑器在页面DOM中的唯一标识符，这通常是初始化编辑器时绑定的容器元素的ID，这个容器有时是一个
`<script>`标签，有时则是在页面渲染后生成的包裹编辑器的最外层`<div>`的ID。获取到这个ID之后，就可以调用UEditor提供的全局方法
`UE.getEditor('编辑器ID')`来获得该编辑器的JavaScript实例对象。有了这个实例对象，我们就能像操作一个对象一样调用它的各种方法，例如使用
`editorInstance.setContent('你的评语HTML', false)`方法来设定编辑器的内容（第二个参数`false`通常表示覆盖现有内容），或者调用
`editorInstance.focus()`让编辑器获得输入焦点。不过，在调用这些操作内容的方法之前，为了确保编辑器已经完全初始化并准备就绪，通常还需要使用
`editorInstance.ready(callback)`方法，将真正的操作代码放在这个`ready`方法提供的回调函数中执行，这样可以避免在编辑器未完全加载时调用API而导致错误。

这套标准的API操作流程听起来似乎相当完善和可靠，足以应对富文本编辑器的交互需求。然而，理论与实践之间有时存在鸿沟。在我们之前的自动化探索，无论是在讨论区场景还是这次的作业批改页面上，尝试通过API与UEditor实例交互时，却遇到了意想不到且令人困惑的麻烦。最主要的问题在于
**编辑器实例的注册似乎存在延迟甚至失败的情况**。我们通过脚本逻辑，能够准确地定位到页面上由UEditor渲染出来的编辑器容器
`<div>`元素，例如一个ID为`edui78`的`<div>`。但紧接着尝试使用`UE.getEditor('edui78')`来获取这个ID对应的实例时，却经常返回
`null`或`undefined`，表明实例获取失败。为了进一步探究原因，我们检查了UEditor用于管理所有已初始化实例的全局对象`UE.instants`
，结果惊讶地发现，我们找到的那个`edui78`的ID，竟然根本**没有被记录在这个全局实例列表里**
！这意味着，尽管编辑器在视觉上已经渲染并显示在页面中，但其对应的JavaScript控制实例并未按照预期的方式进行注册，导致我们无法通过官方API获取到它。

除此之外，有时还会遇到**ID不匹配**的问题。在某些情况下，编辑器实际注册到`UE.instants`中使用的ID，与其在页面上渲染出的最外层容器
`<div>`的ID并不相同。这使得即使实例成功注册了，我们也可能因为使用了错误的ID而无法准确获取，进一步增加了通过API进行自动化操作的难度和不确定性。这些实际遇到的坑，使得原本看似理想的API调用方案变得困难重重。

多次尝试增加延时、使用不同的选择器、检查`UE.instants`，最终的结论是：**在这个特定的动态加载场景下，依赖UEditor的API来注入评语是不可靠的。
** 编辑器的初始化过程可能存在一些特殊机制或者bug，导致我们无法稳定地获取并控制目标评语框的实例。

```javascript
// 伪代码示例：失败的API尝试
async function tryApiComment(questionElement, commentHtml) {
	const commentContainer = questionElement.querySelector('.comment');
	const editorDiv = commentContainer?.querySelector('div.edui-editor[id^="edui"]');
	if (!editorDiv || !editorDiv.id) {
		console.error("Cannot find editor div");
		return;
	}
	const editorId = editorDiv.id; // e.g., "edui78"
	console.log(`Attempting to get editor instance: ${editorId}`);

	// **** THE PROBLEM AREA ****
	// Often failed because 'edui78' wasn't registered in UE.instants
	// or UE.getEditor returned undefined/null even if the div existed.
	let editorInstance;
	try {
		editorInstance = UE.getEditor(editorId); // <--- Fails or returns unusable instance
		if (!editorInstance || typeof editorInstance.setContent !== 'function') {
			throw new Error("Instance invalid or not ready");
		}
	} catch (e) {
		console.error(`Failed to get or validate UE instance ${editorId}:`, e);
		return;
	}
	// **** END PROBLEM AREA ****

	// Code below here would likely not be reached or would fail
	await new Promise(resolve => editorInstance.ready(resolve));
	editorInstance.setContent(commentHtml, false);
	// ... click confirm ...
}
```

API的路走不通，看来只能回到更“原始”的方法。

## 重拾DOM操作

在尝试调用编辑器API屡屡碰壁之后，面对这个似乎无法通过标准途径稳定控制的编辑器实例，我们不得不重新审视最初的思路，再次回到*
*直接操作DOM**这条路径上来。当然，这次不能重蹈覆辙，需要吸取之前的教训，并基于对富文本编辑器工作原理的理解做出调整。

首先，**操作目标必须更加明确**。我们知道，富文本编辑器最终显示内容的地方，通常是它内部的一个`<iframe>`
元素。因此，脚本的核心任务不再是寻找并操作可能不存在或无法获取的`<textarea>`，而是要精准地定位到这个`<iframe>`，并进一步深入到它的
`contentDocument`中，找到那个真正承载内容、允许用户编辑的`<body>`元素（这个`<body>`元素通常会带有一个特定的类名，例如`view`
，并且其`contentEditable`属性被设为`true`）。

其次，需要处理**数据同步的问题**。既然我们无法可靠地找到并更新那个理论上应该存在的、用于表单提交的隐藏`<textarea>`
（它要么就是不出现，要么就是被编辑器的复杂机制隐藏得太深，难以定位），我们就必须做出一个关键的、也是带有一定风险的**假设**
。我们假设：当用户（或我们的脚本）点击了评语框旁边的“完成”按钮时，该按钮绑定的页面JavaScript逻辑，并不会去读取那个我们找不到的
`<textarea>`，而是会**直接从`<iframe>`内部的`<body>`元素中获取当前的`innerHTML`**
，并以此作为用户输入的评语内容进行后续的保存或提交处理。这无疑是一个基于观察和逆向猜测的假设，在无法获取编辑器实例控制权的情况下，这几乎是唯一可行的路径，尽管带着一丝无奈。基于这个假设，我们才能够绕开API，通过直接修改
`<iframe>`内容来模拟评语输入。

基于这个关键的假设——即页面的“完成”按钮会直接读取`<iframe>`内容——我们重新调整了自动化的核心流程。脚本的大致执行步骤变为：首先，仍然是找到页面上所有的简答题列表项
`<li>`元素；接着，按顺序遍历这些题目。对于每一个题目，先定位到其分数输入框，设定好分数（比如先设为满分）并确保触发相关的更新事件。然后，关键的一步是找到并模拟点击该题的“点评”按钮。点击之后，必须
**耐心等待**，给予页面足够的时间来动态加载或显示出富文本编辑器的`<iframe>`以及旁边的“完成”按钮。一旦这些元素出现，脚本就集中精力
**找到目标`<iframe>`**——它通常嵌套在编辑器的主容器`<div>`（例如带有`class="edui-editor"`）内部，并且其`id`可能遵循某种模式（如
`ueditor_X`）。成功定位到`<iframe>`后，脚本会访问其`contentDocument.body`属性，获取到内部的可编辑区域，然后直接将这个区域的
`innerHTML`属性设置为我们预设的评语文本。最后一步，找到并模拟点击“完成”按钮，触发页面自身的评语保存逻辑。当然，在处理完一个题目的所有这些步骤后，依然需要加入适当的延时，再开始处理下一个题目，以保证流程的稳定。

```javascript
// 伪代码示例：修正后的DOM操作 (Iframe Only)
async function commentViaIframe(questionElement, commentHtml) {
	const commentButton = questionElement.querySelector('.comment span.modify');
	if (!commentButton) return;
	commentButton.click();
	await delay(1000); // Wait for iframe etc.

	const commentContainer = questionElement.querySelector('.comment');
	const editorDiv = commentContainer?.querySelector('div.edui-editor');
	const editorIframe = editorDiv?.querySelector('iframe[id^="ueditor_"]');
	const confirmButton = commentContainer?.querySelector('div.confirm');

	if (editorIframe && confirmButton) {
		const iframeDoc = editorIframe.contentDocument || editorIframe.contentWindow?.document;
		if (iframeDoc?.body) {
			iframeDoc.body.innerHTML = commentHtml; // Set content directly
			console.log(`Set iframe content for ${questionElement.id}`);
			confirmButton.click(); // Trigger the page's own logic
			console.log(`Clicked confirm for ${questionElement.id}`);
		} else {
			console.error("Could not access iframe body for " + questionElement.id);
		}
	} else {
		console.error("Could not find iframe or confirm button for " + questionElement.id);
	}
}
```

令人惊喜的是，这个“只改iframe，不碰textarea”的方法，在我们的目标页面上**竟然成功了！** 这意味着页面的“完成”按钮点击事件，确实是去读取了
`<iframe>`里的内容来保存评语，而不需要我们手动去同步那个神秘的（或者说，不存在的）`<textarea>`。这真是峰回路转！

## 引入AI生成个性化评语

解决了基本的操作模拟，下一个目标是让评语不那么千篇一律。如果能根据学生的具体回答生成评语，那就更智能了。

这里自然想到了大型语言模型（LLM）。国内有若干优秀的模型可供选择，比如百度的文心（Ernie）系列。它们提供了API接口，可以通过发送请求（包含问题、学生答案等信息）来获取模型生成的内容（比如我们需要的评语）。

### 选择模型

考虑到批改作业时可能需要处理较多题目，对响应速度有一定要求，同时评语生成任务相对简单，我们选择了文心大模型中的 Ernie Speed
模型，它在速度和效果之间取得了较好的平衡。

典型的API调用流程分为两个主要步骤。首先是**获取授权凭证**，即`access_token`。这需要向百度AI平台的OAuth认证接口发起一次请求，将我们在平台上申请到的API
Key (AK) 和 Secret Key (SK) 作为参数传递过去。认证成功后，接口会返回一个具有一定时效性（例如30天）的`access_token`字符串。这个
`access_token`就像一个临时通行证，是我们后续与具体AI模型服务进行交互的身份凭证。

获取到有效的`access_token`之后，就可以进行第二步：**调用具体的AI模型对话接口**，比如我们选择的Ernie
Speed模型Chat接口。这通常是一个POST请求，请求地址中会带上刚刚获取的`access_token`作为参数。请求体（request
body）则需要按照接口规范构造，核心内容是`messages`字段，这是一个包含对话历史的数组，其中至少要有一条`role`为`"user"`的消息，其
`content`就是我们精心构造的**提示（Prompt）**，包含了问题、学生答案等信息，以及对AI生成评语的要求。此外，还可以包含一个可选的
`system`字段，用于设定AI的角色和全局指令（例如“你是一个大学助教，请生成评语”）。根据需要，也可以加入`temperature`、`top_p`
等参数来调整AI生成内容的多样性和随机性。服务器收到请求后，AI模型会根据`messages`和`system`指令进行处理，并将生成的结果返回。

### 跨域问题（CORS）

直接在浏览器控制台或普通网页脚本中使用`fetch`或`XMLHttpRequest`调用外部API（如`https://aip.baidubce.com`）会立即遇到**
CORS（跨源资源共享）**问题。浏览器出于安全考虑，默认禁止这种跨域请求，除非目标服务器在响应头中明确允许（通过
`Access-Control-Allow-Origin`等）。百度API主要面向服务器端调用，通常不会允许来自任意网页源的直接请求。

控制台日志会明确告诉你：

```
Access to fetch at 'https://aip.baidubce.com/...' from origin 'http://...' has been blocked by CORS policy...
```

### 油猴脚本（Tampermonkey/Greasemonkey）

用户脚本管理器（如Tampermonkey）提供了一个“特权通道”：`GM_xmlhttpRequest`
函数。运行在用户脚本环境中的代码，可以通过这个函数发起跨域请求，因为它是由浏览器扩展本身（而非网页）发起的，从而绕过了常规的CORS限制。

然而，在使用这个强大的工具时，有几个关键点需要特别注意，以确保脚本能够正常工作并拥有必要的权限。首先，**必须在脚本头部的元数据区域
**（即以`// ==UserScript==`开始，以`// ==/UserScript==`结束的部分）**进行授权声明**。需要明确添加一行
`// @grant GM_xmlhttpRequest`，这相当于告知Tampermonkey：“这个脚本需要使用跨域请求的功能，请授予相应权限。”缺少这一步，脚本在尝试调用该函数时会失败。

其次，出于安全考虑，Tampermonkey通常还要求脚本**明确声明它打算连接的外部域**。因此，同样在元数据区域，需要加入类似
`// @connect aip.baidubce.com` 的声明，指明脚本将要与百度AI平台的API服务器进行通信。这个声明有助于用户了解脚本的网络行为，也让脚本管理器能更好地控制权限。

最后，必须理解`GM_xmlhttpRequest`本质上是一个**异步操作**
。这意味着发起请求后，脚本不会停下来等待结果返回，而是会继续执行后续代码。因此，处理请求的结果需要采用异步编程模式。常见的方式是为
`GM_xmlhttpRequest`提供回调函数，例如`onload`参数指定请求成功时执行的函数，`onerror`指定发生网络或其他错误时执行的函数，
`ontimeout`指定请求超时后执行的函数。另一种更现代、更便于管理复杂流程的方式是将其包装在JavaScript的`Promise`对象中，然后结合
`async/await`语法糖来编写看似同步、实则异步的代码，从而更清晰地处理请求的发送和结果的接收。

```javascript
// 伪代码示例：使用 GM_xmlhttpRequest 获取 Token
function getAccessTokenGM(apiKey, secretKey) {
	const url = `https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id=${apiKey}&client_secret=${secretKey}`;
	return new Promise((resolve, reject) => {
		GM_xmlhttpRequest({
			                  method   : "POST",
			                  url      : url,
			                  headers  : {"Content-Type": "application/json", "Accept": "application/json"},
			                  onload   : function (response) {
				                  if (response.status === 200) {
					                  const data = JSON.parse(response.responseText);
					                  if (data.access_token) {
						                  resolve(data.access_token);
					                  } else {
						                  reject(new Error("Token not found in response"));
					                  }
				                  } else {
					                  reject(new Error("HTTP error getting token: " + response.status));
				                  }
			                  },
			                  onerror  : reject,
			                  ontimeout: reject
		                  });
	});
}
```

解决了CORS问题，我们就能在脚本中畅通无阻地与AI对话了。

## Prompt工程与JSON约定

仅仅能调用AI还不够，关键在于如何让AI理解我们的意图，并给出我们想要的输出。这就是**Prompt Engineering**。

### 初步Prompt

最开始，可能只是简单地把问题和学生答案拼接到Prompt里，让AI自由发挥写评语。

```
问题：请阐述空镜头的功能。
学生答案：空镜头可以交代环境，转换时空。
请给一句评语：
```

这样可能会得到类似“回答基本正确，但不够全面。”的评语，还不错。

### 引入更多上下文

仅仅向AI提供问题和学生的回答，虽然能生成初步的评语，但为了追求更准确、更贴合实际批改需求的评价，我们可以*
*在提示（Prompt）中为AI提供更丰富的上下文信息**。例如，将题目的**正确答案**也一并告知AI，让它明确知道标准的参照是什么。同时，如果页面上提供了官方的
**答案解析**或者评分说明，将这部分内容也加入到提示中，可以帮助AI更好地理解题目的考查点和评分标准。此外，告知AI该题目的**满分值
**，也能让它在后续被要求辅助打分时（如果我们决定加入这个功能），对分数的范围和量级有一个基本的概念，从而可能给出更合理的评分建议。通过整合这些额外的信息，我们能够构建一个信息更全面的提示，引导AI做出更精准、更有依据的判断和评价。

```
你是大学课程的助教。请根据以下信息评价学生回答，并给一句简洁评语：

问题 (满分 8分):
空镜头具体有哪些表现功能和艺术价值？

学生回答:
（1）交代故事环境 （2）作为时空转换的手段

正确答案参考:
（1）交代故事环境 （2）作为时空转换的手段 （3）渲染气氛，烘托感情 （4）营造意境

答案解析参考:
空镜头是指只有景物没有人物的镜头...具体表现在四个方面。

评语：
```

这样AI就能更清楚地知道学生答对了哪些要点，遗漏了哪些。

### 定义输出格式（JSON）

自由格式的评语仍然需要脚本去解析。如果我们还想让AI辅助打分呢？直接让它在评语里包含分数，解析起来更麻烦且易出错。更好的方式是让AI返回
**结构化的数据**，比如JSON。

这就需要修改**System Prompt**（给AI设定的全局角色和指令）和**User Prompt**（具体请求），明确要求它返回特定格式的JSON：

```javascript
// System Prompt 示例
const DEFAULT_SYSTEM_PROMPT = `
You are a university teaching assistant grading homework.
Based on the provided context, evaluate the student's answer.
Respond ONLY with a JSON object containing two keys:
1.  "score": A numerical score between 0 and the Maximum Score (inclusive).
2.  "comment": A brief, positive, and constructive comment (max 25 characters).

Example Response Format:
{
  "score": 10,
  "comment": "回答准确，要点清晰。"
}

Do NOT include any other text or markdown formatting.
`;

// User Prompt 示例 (结合System Prompt使用)
const userPrompt = `
问题 (满分 ${maxScore}分):
${questionText}

学生回答:
${studentAnswerText}

正确答案参考:
${correctAnswerText || "无"}

答案解析参考:
${analysisText || "无"}

请根据以上信息，严格按照JSON格式返回评分和评语:
`;
```

同时，在调用API时，如果接口支持（比如某些版本的文心API），可以尝试加入`"response_format": "json_object"`参数，进一步强制JSON输出。

### 解析与应用

当我们通过API向AI模型发送请求，并要求它以JSON格式返回评分和评语后，脚本在接收到响应时还需要执行一系列处理步骤才能最终将结果应用到页面上。首先，由于AI有时可能在生成的JSON字符串前后包裹markdown的代码块标记（如
` ```json ... ``` `），我们需要进行一次**可选的清理操作**，确保移除这些额外的标记，得到一个纯净的JSON字符串。

接下来，是关键的**解析**步骤。使用JavaScript内置的`JSON.parse()`
方法，将这个清理后的字符串转换成一个标准的JavaScript对象。如果AI遵循了我们的指令，这个对象应该包含我们期望的键，例如`score`和
`comment`。

然而，不能完全信任外部服务的返回结果，因此**验证**是必不可少的一环。脚本需要检查解析得到的对象是否确实包含了`score`和
`comment`这两个属性。对于`score`，还需要进一步验证它的值是否是一个有效的数字，并且这个数字是否落在合理的范围内（例如大于等于0且小于等于该题目的满分）。对于
`comment`，则可以检查它是否是一个非空的字符串。

只有通过了这些验证，我们才能放心地将结果**应用**
到用户界面上。将验证过的分数填入对应题目的分数输入框中，并将获取到的评语通过之前确定的DOM操作方式（例如，点击“点评”按钮后找到文本区域并设置其值，再点击“完成”）写入评语框。如果在解析或验证过程中发现问题（例如返回的不是有效的JSON，或者分数超出了范围），脚本则应该采取相应的错误处理措施，比如记录警告信息，并在评语框中填入一条表示失败或无效的默认提示。

```javascript
// 伪代码：处理AI返回的JSON
async function handleAiResponse(aiResultString, scoreInput, commentArea, confirmButton, maxScore) {
	let score = 0;
	let comment = "（处理评语失败）";
	try {
		// Clean potential markdown backticks
		const cleanedString = aiResultString.replace(/^```json\s*|```$/g, '').trim();
		const result = JSON.parse(cleanedString);
		if (result && typeof result.score === 'number' && typeof result.comment === 'string') {
			// Validate score
			const potentialScore = parseFloat(result.score);
			const maxScoreNum = parseFloat(maxScore);
			if (!isNaN(potentialScore) && !isNaN(maxScoreNum) && potentialScore >= 0 && potentialScore <= maxScoreNum) {
				score = potentialScore;
			} else {
				console.warn(`Invalid score from AI: ${result.score}, Max: ${maxScore}. Defaulting to 0.`);
				comment = `(分数无效) ${result.comment}`; // Prepend warning
			}
			comment = result.comment; // Use AI comment regardless of score validity (unless invalid JSON)
		} else {
			console.warn("AI response is not valid JSON or missing keys:", result);
			comment = "（评语格式无效）";
		}
	} catch (e) {
		console.error("Error parsing AI JSON response:", e, aiResultString);
		comment = "（评语解析错误）";
	}

	// Apply to UI
	if (scoreInput) scoreInput.value = score;
	if (commentArea) commentArea.value = comment;
	if (confirmButton) confirmButton.click();

	console.log(`Applied Score: ${score}, Comment: ${comment}`);
}
```

## 整合、成果与反思

将前面讨论的各项技术环节——精确的DOM定位与操作、借助Tampermonkey实现的跨域API调用、必要的信息提取、与AI模型的交互设计（包括精心构造的prompt和对JSON响应的解析处理），以及为方便使用而添加的UI按钮——整合起来，便形成了一个能够运行的Tampermonkey自动化辅助批改脚本。在这一过程中，
**异步流程控制**尤为关键，使用`async/await`确保了涉及网络请求和延时的操作能按预期顺序执行。同时，健壮的**错误处理**机制（
`try...catch`）保证了单个题目的失败不会导致整个流程中断。引入SweetAlert库则显著改善了**用户交互体验**，提供了清晰的确认、进度和结果反馈。良好的
**代码组织**，通过函数封装不同功能模块，提高了可读性和可维护性。而**按需执行**的设计（通过按钮触发）则将控制权交还给用户，避免了自动执行可能带来的风险。

最终实现的这个脚本，虽然它的目标并非完全替代人工，尤其是在深层理解和个性化反馈方面，但它确实有效地达成了初衷：**显著减轻重复性工作负担
**。它能够为简答题**预设基于AI建议的分数**（通常设置为满分或AI返回的建议值），老师只需在此基础上微调；同时**自动生成基础评语
**，为老师提供一个快速参考或修改的起点；最重要的是，它实现了**批量处理**，一次点击即可顺序完成对页面所有简答题的初步评分和评语添加，极大地节省了时间和精力。

这次实践也带来了深刻的技术反思。**Web应用的复杂性**
远超表面所见，动态加载、第三方组件（尤其是富文本编辑器）都可能让标准的API调用路径受阻，有时不得不回退到更直接但可能更脆弱的DOM操作，并基于对页面行为的假设来设计方案。这也界定了
**自动化的边界**：技术擅长处理重复性任务，提高效率，但无法取代需要深度理解、创造力和情感投入的核心工作，如高质量的教学反馈。因此，工具的定位应是辅助而非替代。与AI的协作则凸显了
**Prompt工程的核心价值**，如何清晰、准确、结构化地向AI提问，并规范其输出（如要求返回JSON），是获得可用结果的关键。最后，**安全意识
**贯穿始终，尤其是在浏览器端处理API密钥时，必须认识到其固有的风险，仅限于可信的个人环境使用。

总而言之，这个小项目虽然技术栈并不算高深，但其探索和解决问题的过程，以及最终能为家人带来实际帮助所带来的成就感，都非常有意义。希望这次分享，能为同样希望用技术改善身边小事的朋友们提供一些参考。若你想尝试类似实践，请牢记
**具体情况具体分析**（每个平台都不同）、**安全第一**（尤其注意敏感信息处理）以及**拥抱调试**
（耐心使用开发者工具）这几点关键原则。有时，编码的乐趣恰恰在于这些解决实际问题的点滴尝试和带来的微小改变。感谢阅读！