---
date: '2025-05-02T17:16:19+08:00'
draft: false
summary: 'Discover how a Tampermonkey userscript was developed using Baidu Ernie AI to automate scoring and commenting for online short-answer homework, significantly reducing repetitive grading tasks for educators.'
title: 'Automating Online Grading with Tampermonkey and AI'
tags: [ "tampermonkey", "userscript", "ai", "baidu-ernie", "llm", "homework-grading", "grading-automation", "education-technology", "edtech", "javascript", "dom-manipulation", "api-integration", "gm-xmlhttprequest", "cors-workaround", "prompt-engineering", "json", "browser-automation", "teacher-tools" ]
---

Today I want to share a somewhat special programming project – one not born out of a desire for flashy tech or
commercial application, but from a simple idea: helping my mom lighten her workload a bit.

My mother is a dedicated teacher, and with the rise of online education, much of her work, including grading
assignments, has moved online. While this undoubtedly increases teaching flexibility, it also brings new challenges.
Grading homework on certain online platforms, in particular, involves a significant amount of repetitive actions, which
is both time-consuming and mentally draining. Seeing her often busy late into the night, as her son, I always thought
about whether I could use the technical skills I've learned to do something for her.

The specific target this time was the grading process for "short-answer questions" on her teaching platform. These types
of questions typically require the teacher to read the student's response, then assign a score and provide written
feedback. While the final judgment and personalized feedback are irreplaceable, there seemed to be room for automation
in the initial scoring and basic comment generation.

Thus began an exploratory journey combining browser scripting, DOM manipulation, API calls, and a touch of artificial
intelligence.

## Starting with the Browser Console

Getting started is always the hardest part. The most direct thought was: "Can I run some code in the browser to simulate
mouse clicks and keyboard input?" The answer is yes. The browser's developer tools (opened by pressing F12) and
specifically the "Console" tab are powerful weapons for this.

The first step in automation is teaching the code to "recognize" the elements on the page. Just as a person grading
needs to find the question, the answer box, the score field, the comment area, and the submit button, the code needs
specific "addresses" – DOM selectors – to locate these elements.

Opening a typical assignment grading page, the initial exploration involved carefully studying its HTML structure using
the developer tools. We discovered some consistent design patterns: the content and response area for each short-answer
question were usually entirely wrapped within a list item `<li>` element. To differentiate question types or provide
unique identifiers, these `<li>` elements often carried specific attributes, such as `data-questiontype="5"` perhaps
marking it as a short-answer question, along with a unique `id` attribute.

Inside each `<li>` representing a question, we could find an `<input type="number">` tag used for entering the score.
This input field not only revealed the maximum possible score for the question via its `max` attribute but also hinted
at how backend data processing might be indexed through its `name` attribute (often formatted like
`questions[0].studentScore`).

The commenting functionality appeared more dynamic. Initially, only a "Comment" button was visible, typically
implemented as a `<span>` tag (perhaps with a class like `modify`). A user click on this button would dynamically reveal
the text area for entering the comment – usually a `<textarea>` element (possibly with classes like
`teacherWrite comments`) – along with a "Done" or "Confirm" button (maybe a `<div>` tag) to finalize the comment input.

Finally, near the bottom of the page, there was generally a global action button, such as "Submit and Grade Next,"
designed to save all the grading results (scores and comments) for the current page in one go and then load the next
assignment requiring attention. Locating and understanding these key elements formed the foundation for the subsequent
automation script design.

With the interactive page elements identified, the basic automation workflow began to take shape. First, the script
needed to identify all the short-answer questions on the page requiring attention. This could be achieved using the
`document.querySelectorAll` method combined with the selectors identified earlier (like `li[data-questiontype="5"]`),
yielding a list of all the relevant `<li>` elements.

Next, the script would need to process each question element in this list sequentially, typically involving a loop.
Within each iteration of the loop, focusing on the current question, the script would perform two core tasks: filling in
the score and adding a comment.

For scoring, the initial idea was to locate the score input field within the current question element, read its `max`
attribute to get the maximum possible score, and then directly set the input's `value` to this maximum score. This was
conceived as a basic starting strategy, open to later refinement with more complex logic, but viable as a starting
point.

Adding the comment was slightly more complex due to the dynamic nature of the elements involved. The script would first
need to find and simulate a click on the "Comment" button. Crucially, after the click, it couldn't immediately search
for the comment box; **a waiting period** was essential because the comment input area and the "Done" button were
dynamically loaded or displayed, requiring time for the page to react. Once the wait was over, the script would locate
the newly appeared `<textarea>` element for comments and set its `value` to a predefined, generic comment text. Finally,
it would find and simulate a click on the corresponding "Done" button to confirm the entry of this comment.

To enhance the stability of the automation process and better mimic human interaction patterns, incorporating a short
delay after completing all steps for one question was deemed necessary. This pause helps prevent issues caused by
excessively rapid operations that might outpace the page's script responsiveness or potentially trigger anti-bot
mechanisms on some websites.

After processing all questions according to this flow, the logical final step would be to simulate a click on the global
submit button at the bottom of the page to save all the grading results. However, considering the risks associated with
full automation and the importance of allowing the teacher a final review opportunity, this final submission step was
initially designated as optional or to be triggered manually by the user. This preliminary plan, primarily relying on
direct DOM manipulation, laid the groundwork for the subsequent coding implementation.

This initial concept relied heavily on **direct DOM manipulation** (finding an element -> modifying its
attributes/triggering clicks). Within the browser console, this is often feasible.

```javascript
// Pseudocode Example: Initial Concept
function gradeShortAnswer(questionElement) {
	// Find score input and set to max score
	const scoreInput = questionElement.querySelector('input.student-score');
	const maxScore = scoreInput?.max;
	if (scoreInput && maxScore) {
		scoreInput.value = maxScore;
		console.log(`Set score for ${questionElement.id} to ${maxScore}`);
		// Trigger events so the page knows the value changed (Important!)
		scoreInput.dispatchEvent(new Event('input', {bubbles: true}));
		scoreInput.dispatchEvent(new Event('change', {bubbles: true}));
	}

	// Find and click the "Comment" button
	const commentButton = questionElement.querySelector('.comment span.modify');
	if (commentButton) {
		commentButton.click();
		// Need to wait for the comment box to appear...
		setTimeout(() => {
			const commentArea = questionElement.querySelector('.comment textarea.teacherWrite');
			const confirmButton = questionElement.querySelector('.comment div.confirm');
			if (commentArea && confirmButton) {
				commentArea.value = "Student's response is good!"; // Preset comment
				confirmButton.click();
				console.log(`Comment added for ${questionElement.id}`);
			}
		}, 1000); // Assuming a 1-second wait
	}
}

// Get all short-answer questions and process
// document.querySelectorAll('#shiti-content li.subjective[data-questiontype="5"]')
//    .forEach(el => gradeShortAnswer(el));

// Note: Real application requires more complex async handling and error management
```

This approach seems appealing, but in practice, especially on complex, dynamically loaded modern web pages, it often
encounters its first major roadblock.

## Exploring the API Route (and Rich Text Editor Pitfalls)

In the context of real-world teaching platforms (including both the discussion forum scenarios explored earlier and the
current homework grading task), the comment input field is frequently not a simple `<textarea>`. Instead, it's often a *
*Rich Text Editor (RTE)**, such as the familiar UEditor, CKEditor, or TinyMCE.

These editors typically render a complex structure, often involving an `<iframe>` or a `<div>` with the
`contenteditable` attribute, in place of the original `<textarea>` (or sometimes even a `<script>` tag), and provide a
toolbar for formatting.

However, this assumption of directly manipulating a simple `<textarea>` encounters challenges when faced with the Rich
Text Editors commonly used on these platforms. The introduction of these editors complicates the seemingly
straightforward interaction. Firstly, the **DOM structure itself changes**. We can no longer directly target a simple
text input; instead, we must navigate into the more complex structure generated by the editor, such as an embedded
`<iframe>` element, and then further locate the editable `<body>` within that iframe, or perhaps a specific `<div>`
element configured with the `contenteditable` attribute.

Secondly, and more critically, is the **dependency on the editor's API**. Rich Text Editors usually have their own set
of JavaScript APIs to manage content and state. If we bypass these APIs and directly modify the `innerHTML` of the
`<iframe>` or the `innerText` of a `contenteditable` element, the editor's internal state might not update accordingly.
The direct consequence is that when a save action is triggered (like clicking the "Done" button), the editor might still
consider the content empty or unchanged because it relies on its API calls to synchronize and retrieve the final
content (often synchronizing it to a hidden form field). Therefore, merely altering the visual presentation doesn't
guarantee the data will be captured correctly.

Finally, **instance management** also becomes a factor. On a page containing multiple short-answer questions, each
question's comment box is typically an independent instance of the Rich Text Editor. This means if we want to interact
via API, we must be able to accurately identify and obtain the specific instance object for the editor corresponding to
the question currently being processed. Only then can we call its specific methods (like setting content or focusing).
This undoubtedly increases the complexity of the automation script. The emergence of these issues indicated that direct
DOM manipulation might be insufficient for handling RTE scenarios, necessitating an exploration into using the editor's
API.

Facing the challenges posed by Rich Text Editors, the natural next step was to attempt interaction using the editor's
own provided API, which is generally the more standardized and reliable method. Taking the common UEditor as an example,
the standard operational workflow typically involves these steps: First, identify the unique identifier of the target
editor container within the page's DOM. This ID is usually associated with the element (sometimes a `<script>` tag,
sometimes the outermost `<div>` rendered by the editor) that the editor was initialized upon. Once this ID is obtained,
one can call UEditor's global method `UE.getEditor('editorID')` to retrieve the JavaScript instance object for that
specific editor. With this instance object in hand, various methods can be invoked, such as using
`editorInstance.setContent('your comment HTML', false)` to set the editor's content (where the second argument `false`
usually means overwriting existing content), or calling `editorInstance.focus()` to bring focus to the editor. However,
before invoking methods that manipulate content, it's crucial to ensure the editor has fully initialized and is ready to
accept API calls. This is typically achieved using the `editorInstance.ready(callback)` method, placing the actual
content manipulation code within the provided callback function to avoid errors caused by calling APIs on an
incompletely loaded editor.

This standard API workflow sounds quite robust and sufficient for interacting with Rich Text Editors. However, theory
and practice sometimes diverge. In our previous automation explorations, both in discussion forum scenarios and during
the current homework grading attempt, trying to interact with UEditor instances via the API led to unexpected and
perplexing difficulties. The primary issue encountered was that **editor instance registration seemed delayed or even
failed entirely**. Using script logic, we could accurately locate the editor's container `<div>` element rendered on the
page, for instance, one with the ID `edui78`. But subsequent attempts to fetch the instance for this ID using
`UE.getEditor('edui78')` frequently returned `null` or `undefined`, indicating failure. To investigate further, we
examined UEditor's global object `UE.instants`, which manages all initialized instances, only to be surprised that the
ID `edui78` we found was **not listed** in this global registry at all! This implied that although the editor was
visually rendered and present on the page, its corresponding JavaScript control instance wasn't being registered in the
expected manner, preventing us from accessing it through the official API.

Additionally, we sometimes encountered **ID mismatch issues**. In certain situations, the ID under which the editor
instance was actually registered in `UE.instants` did not match the ID of the outermost container `<div>` rendered on
the page. This meant that even if an instance was successfully registered, we might fail to retrieve it because we were
using the incorrect ID derived from the visible DOM element, further complicating and adding uncertainty to automation
attempts via the API. These practical pitfalls made the seemingly ideal API approach fraught with difficulty.

After multiple attempts involving increased delays, trying different selectors, and inspecting `UE.instants`, the
conclusion was clear: **in this specific dynamically loaded context, relying on the UEditor API to inject comments was
unreliable.** The editor's initialization process likely involved some specific mechanism or perhaps a bug that
prevented us from consistently obtaining and controlling the target comment box instance.

```javascript
// Pseudocode Example: Failed API Attempt
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

With the API route seemingly blocked, the only option left was to return to the more "primitive" method.

## Revisiting DOM Manipulation

After repeatedly hitting roadblocks while trying to use the editor's API, and facing an editor instance that seemed
uncontrollable through standard means, we had to reconsider our initial approach and **return to the path of direct DOM
manipulation**. This time, however, we needed to learn from past mistakes and make adjustments based on our
understanding of how Rich Text Editors typically function.

First, the **target of manipulation needed to be more precise**. We knew that the final content displayed by an RTE
usually resides within an `<iframe>` element. Therefore, the script's core task shifted from trying to find and
manipulate a potentially non-existent or inaccessible `<textarea>` to accurately locating this `<iframe>`. It then
needed to delve deeper into its `contentDocument` to find the actual `<body>` element (often marked with a class like
`view` and having its `contentEditable` attribute set to `true`) which holds the content and allows user editing.

Second, the **issue of data synchronization** had to be addressed. Since we couldn't reliably find and update the hidden
`<textarea>` that, in theory, should exist for form submission (it either didn't appear as expected or was too deeply
buried by the editor's complex mechanisms), we had to make a **critical, albeit risky, assumption**. We assumed that
when the user (or our script) clicks the "Done" button associated with the comment box, the page's own JavaScript logic
bound to that button **reads the current content directly from the `<iframe>`'s inner `<body>` element**, rather than
from the elusive `<textarea>`. This content would then be used for subsequent processing, such as saving or submitting
the comment. This was undoubtedly an assumption based on observation and reverse-engineering guesswork, but given the
inability to gain control via the editor's instance, it seemed the only viable path forward, albeit tinged with a degree
of uncertainty. This assumption allowed us to bypass the API and simulate comment input by directly modifying the
`<iframe>`'s content.

Based on this key assumption – that the page's "Done" button reads directly from the `<iframe>` content – we readjusted
the core automation workflow. The general sequence of steps for the script became: first, as before, identify all the
short-answer list item `<li>` elements on the page; next, iterate through these questions sequentially. For each
question, initially locate its score input field, set the desired score (e.g., defaulting to the maximum), and ensure
relevant update events are triggered. Then comes the crucial step: find and simulate a click on that question's "
Comment" button. After clicking, it's vital to **patiently wait**, allowing the page sufficient time to dynamically load
or display the Rich Text Editor's `<iframe>` along with the adjacent "Done" button. Once these elements appear, the
script concentrates on **finding the target `<iframe>`** – typically nested within the editor's main container `<div>` (
perhaps with a class like `edui-editor`) and possibly having an ID following a pattern (like `ueditor_X`). Upon
successfully locating the `<iframe>`, the script accesses its `contentDocument.body` property to get the internal
editable area, and then directly sets the `innerHTML` property of this area to our predefined comment text. The final
action within the loop is to find and simulate a click on the "Done" button, thereby triggering the page's own logic for
saving the comment. Naturally, after completing all these steps for one question, incorporating an appropriate delay
before proceeding to the next remains essential for process stability.

```javascript
// Pseudocode Example: Revised DOM Manipulation (Iframe Only)
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

Surprisingly, this approach of "modify only the iframe, ignore the textarea" **actually worked** on our target page!
This implied that the "Done" button's click event handler indeed read the content from the `<iframe>` to save the
comment, without requiring us to manually sync the mysterious (or perhaps non-existent) `<textarea>`. It was a welcome
breakthrough.

## Introducing AI for Personalized Comments

Having solved the basic operational simulation, the next goal was to make the comments less repetitive. Generating
comments based on the student's specific answer would make the process more intelligent.

This naturally led to considering Large Language Models (LLMs). Several excellent models are available, including
Baidu's Ernie series in China. They provide APIs that allow sending requests (containing information like the question,
student answer, etc.) to receive model-generated content, such as the comments we needed.

### Choosing the Model

Considering the potential need to process numerous questions during grading, response speed was a requirement. At the
same time, the task of generating a short comment is relatively simple. We opted for the Ernie Speed model from the
Ernie family, as it offered a good balance between speed and performance for this task.

### API Call Workflow

To interact with the AI model, a typical API call process involves two main steps. The first is **obtaining an
authorization credential**, known as an `access_token`. This requires making a request to the platform's OAuth
authentication endpoint, passing along the API Key (AK) and Secret Key (SK) obtained from the platform registration.
Upon successful authentication, the endpoint returns an `access_token` string, which usually has an expiration period (
e.g., 30 days). This token acts like a temporary passport, credentialing subsequent interactions with the specific AI
model services.

Once a valid `access_token` is acquired, the second step is to **call the specific AI model's dialogue interface**, such
as the Ernie Speed Chat API. This is typically a POST request sent to an endpoint URL that includes the obtained
`access_token` as a parameter. The request body must be structured according to the API's specifications. The core
component is the `messages` field, an array representing the conversation history, which must contain at least one
message with `role: "user"`. The `content` of this user message is our carefully crafted **prompt**, containing the
question, student's answer, and instructions for the AI comment generation. Additionally, an optional `system` field can
be included to define the AI's role and provide global instructions (e.g., "You are a university TA generating
comments"). Depending on the need, other parameters like `temperature` or `top_p` can be added to control the diversity
and randomness of the generated output. Upon receiving the request, the AI model processes the input based on the
`messages` and `system` prompt and returns the generated result.

### Cross-Origin Issues (CORS)

Attempting to call external APIs (like `https://aip.baidubce.com`) directly from browser console scripts or standard
webpage JavaScript using `fetch` or `XMLHttpRequest` immediately runs into **CORS (Cross-Origin Resource Sharing)**
issues. For security reasons, browsers block such cross-domain requests by default, unless the target server explicitly
permits them via specific response headers (like `Access-Control-Allow-Origin`). Baidu's API endpoints, being primarily
designed for server-to-server communication, usually do not send the necessary headers to allow direct requests from
arbitrary web origins.

The browser console will explicitly state the block:

```
Access to fetch at 'https://aip.baidubce.com/...' from origin 'http://...' has been blocked by CORS policy...
```

### The Tampermonkey Solution

Userscript managers like Tampermonkey provide a privileged channel: the `GM_xmlhttpRequest` function. Code running
within a userscript environment can use this function to make cross-domain requests because the request originates from
the browser extension itself, not the webpage's restricted context, thus bypassing standard CORS limitations.

However, using this powerful function requires attention to a few key details to ensure the script works correctly and
has the necessary permissions. Firstly, **explicit authorization must be declared in the script's metadata block** (the
section starting with `// ==UserScript==` and ending with `// ==/UserScript==`). A line `// @grant GM_xmlhttpRequest`
must be included, essentially informing Tampermonkey: "This script needs permission to make cross-domain requests."
Without this declaration, attempts to call the function will fail.

Secondly, for security purposes, Tampermonkey usually requires the script to **explicitly declare the external domains
it intends to connect to**. Therefore, within the same metadata block, a declaration like `// @connect aip.baidubce.com`
is needed, specifying that the script will communicate with Baidu's AI platform API server. This declaration helps users
understand the script's network activity and allows the script manager to enforce finer-grained permissions.

Lastly, it's crucial to understand that `GM_xmlhttpRequest` is inherently an **asynchronous operation**. This means that
after initiating a request, the script execution doesn't pause to wait for the response; it continues with the
subsequent code. Consequently, handling the request's outcome requires using asynchronous programming patterns. Common
approaches include providing callback functions to `GM_xmlhttpRequest`—such as `onload` for successful responses,
`onerror` for network or other errors, and `ontimeout` for timeouts. A more modern and often more manageable approach
for complex workflows involves wrapping the `GM_xmlhttpRequest` call within a JavaScript `Promise` object, which then
allows the use of `async/await` syntax. This makes the asynchronous code appear more like synchronous code, leading to
clearer logic for sending requests and processing their results.

```javascript
// Pseudocode Example: Using GM_xmlhttpRequest for Token
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

With CORS issues resolved, we could now communicate freely with the AI from within the script.

## Prompt Engineering and JSON Agreement

Simply being able to call the AI wasn't enough; the key was instructing it effectively to get the desired output. This
is where **Prompt Engineering** comes in.

### Initial Prompt

Initially, one might just concatenate the question and student answer into the prompt and let the AI write a comment
freely.

```
Question: Please explain the function of an empty shot (kūjìngtóu).
Student Answer: Empty shots can establish the environment and transition time/space.
Please provide a one-sentence comment:
```

This might yield a comment like, "The answer is basically correct, but not comprehensive enough," which is a decent
start.

### Adding More Context

Providing only the question and student's answer, while capable of generating a basic comment, might lack the nuance
needed for more accurate evaluation. To enhance the AI's assessment capabilities, we can **enrich the prompt with
additional contextual information**. For instance, including the **correct answer** explicitly allows the AI to know the
standard benchmark. Similarly, providing the official **answer analysis** or grading rubric, if available on the page,
helps the AI better grasp the question's intended focus and evaluation criteria. Furthermore, informing the AI about the
**maximum score** for the question gives it a sense of the grading scale, potentially leading to more reasonable score
suggestions if we later task it with assisting in grading. By integrating these extra pieces of information, we
construct a more comprehensive prompt, guiding the AI towards making more precise and well-founded judgments and
comments.

```
You are a university teaching assistant. Please evaluate the student's answer based on the following information and provide a concise comment:

Question (Max Score: 8 points):
What are the specific functions and artistic values of an empty shot (kūjìngtóu)?

Student Answer:
(1) Establish the story environment (2) Serve as a means of spatiotemporal transition

Correct Answer Reference:
(1) Establish the story environment (2) Serve as a means of spatiotemporal transition (3) Render atmosphere, enhance emotion (4) Create artistic conception (yìjìng)

Answer Analysis Reference:
An empty shot is one containing only scenery, no characters... It serves multiple expressive functions and artistic values, specifically in four aspects.

Comment:
```

This allows the AI to more clearly see which points the student covered and which were missed.

### Defining Output Format (JSON)

Free-text comments still require the script to parse them. What if we also want the AI to suggest a score? Including the
score directly within the comment makes parsing more difficult and error-prone. A better approach is to have the AI
return **structured data**, such as JSON.

This requires modifying the **System Prompt** (global role/instructions for the AI) and the **User Prompt** (specific
request) to explicitly ask for a JSON object in a specific format:

```javascript
// Example System Prompt
const DEFAULT_SYSTEM_PROMPT = `
You are a university teaching assistant grading homework.
Based on the provided context, evaluate the student's answer.
Respond ONLY with a JSON object containing two keys:
1.  "score": A numerical score between 0 and the Maximum Score (inclusive).
2.  "comment": A brief, positive, and constructive comment (max 25 characters).

Example Response Format:
{
  "score": 10,
  "comment": "Accurate answer, key points clear."
}

Do NOT include any other text or markdown formatting.
`;

// Example User Prompt (used with the System Prompt)
const userPrompt = `
Question (Max Score: ${maxScore} points):
${questionText}

Student Answer:
${studentAnswerText}

Correct Answer Reference:
${correctAnswerText || "None provided"}

Answer Analysis Reference:
${analysisText || "None provided"}

Based on the information above, please return ONLY the JSON object with the score and comment:
`;
```

Additionally, when calling the API, if the specific endpoint supports it (like some newer Ernie API versions), one might
try adding the `"response_format": "json_object"` parameter to further enforce the JSON output structure.

### Parsing and Application

Upon receiving the `result` string containing the AI's response, the script needs to perform several processing steps
before the information can be applied to the webpage. First, since AIs sometimes wrap JSON strings in markdown code
block markers (like ` ```json ... ``` `), an **optional cleanup step** is necessary to remove these extraneous
characters, yielding a clean JSON string.

The next critical step is **parsing**. Using JavaScript's built-in `JSON.parse()` method, this cleaned string is
converted into a standard JavaScript object. If the AI adhered to the instructions, this object should contain the
expected keys, such as `score` and `comment`.

However, trusting external service responses implicitly is unwise, making **validation** an essential part of the
process. The script must check if the parsed object actually contains both the `score` and `comment` properties. For the
`score`, further validation is needed to ensure its value is a valid number and falls within the acceptable range (e.g.,
greater than or equal to 0 and less than or equal to the question's maximum score). For the `comment`, a simple check
for a non-empty string might suffice.

Only after passing these validation checks can the results be confidently **applied** to the user interface. The
validated score is inserted into the corresponding question's score input field, and the retrieved comment is written
into the comment textarea using the previously determined DOM manipulation method (which might involve clicking "
Comment," finding the textarea, setting its value, and then clicking "Done"). If issues arise during parsing or
validation (e.g., the response isn't valid JSON, or the score is out of range), the script should implement appropriate
error handling, such as logging a warning and populating the comment field with a default message indicating failure or
invalidity.

```javascript
// Pseudocode: Handling AI JSON Response
async function handleAiResponse(aiResultString, scoreInput, commentArea, confirmButton, maxScore) {
	let score = 0;
	let comment = "(Failed to process comment)";
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
				comment = `(Invalid Score) ${result.comment}`; // Prepend warning
			}
			comment = result.comment; // Use AI comment regardless of score validity (unless invalid JSON)
		} else {
			console.warn("AI response is not valid JSON or missing keys:", result);
			comment = "(Invalid comment format)";
		}
	} catch (e) {
		console.error("Error parsing AI JSON response:", e, aiResultString);
		comment = "(Error parsing comment)";
	}

	// Apply to UI
	if (scoreInput) scoreInput.value = score;
	if (commentArea) commentArea.value = comment;
	if (confirmButton) confirmButton.click();

	console.log(`Applied Score: ${score}, Comment: ${comment}`);
}
```

## Integration, Results, and Reflections

Bringing together precise DOM manipulation, cross-domain API calls via Tampermonkey, essential data extraction,
carefully designed AI interaction (including prompt engineering and JSON parsing), and a user-friendly trigger button
resulted in a functional Tampermonkey script for assisted grading. Key to its success were crucial design principles: *
*asynchronous flow control** using `async/await` to manage network requests and delays sequentially; robust **error
handling** with `try...catch` to prevent single failures from stopping the entire process; an enhanced **user experience
** through SweetAlert for confirmations and feedback; modular **code organization** into functions for readability and
maintenance; **on-demand execution** via a button click for user control; and a constant awareness of **security
implications**, limiting the script's use to personal, trusted environments due to the client-side handling of API keys.

While this script demonstrably cannot replace a teacher's nuanced judgment or personalized feedback, it effectively met
its primary objective: **significantly reducing repetitive workload**. It provides **preliminary AI-suggested scores** (
defaulting to the max or using parsed JSON values), requiring only teacher review and adjustment. It **automatically
generates basic comments** based on AI analysis, serving as a starting point for faster feedback. Most importantly, it
enables **batch processing**, handling all short-answer questions on the page sequentially with a single click, saving
considerable time.

This project offered profound insights into the **complexity of real-world web applications**, where dynamic loading and
third-party components often complicate standard approaches, sometimes making direct DOM manipulation a necessary, if
potentially fragile, alternative to unreliable APIs. It also clearly defined the **boundaries of automation**;
technology excels at efficiency and repetitive tasks but cannot replicate the deep understanding and empathy required
for high-quality educational feedback – the goal must be assistance, not replacement. Furthermore, the experience
underscored the **critical role of prompt engineering** in effectively communicating intent to AI and the value of
structuring requests for predictable, usable output like JSON. Finally, it served as a potent reminder about **security
consciousness** when handling sensitive credentials in client-side scripts.

In conclusion, though a modest personal project, the journey of tackling these challenges and creating a genuinely
helpful tool for family was deeply rewarding. Hopefully, sharing this exploration offers some practical insights. If you
undertake a similar project, remember to **analyze your specific target platform meticulously**, always **prioritize
security** (especially with API keys), and **embrace the debugging process** using your browser's developer tools.
Often, the true joy of coding lies in overcoming these practical hurdles and making tangible, positive changes in
everyday life. Thank you for reading!