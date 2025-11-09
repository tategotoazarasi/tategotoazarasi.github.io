---
date: '2025-11-09T19:33:20Z'
draft: false
summary: 'How to build a robust video downloader from scratch with TypeScript, Node.js, and Playwright, capable of handling complex anti-scraping mechanisms, dynamic content loading, and network race conditions.'
title: 'Building a TypeScript Video Downloader for Complex, Anti-Scraping Websites'
tags: ["typescript", "nodejs", "playwright", "web-scraping", "automation", "video-downloader", "anti-scraping", "race-condition", "network-interception", "browser-automation", "typescript-video-downloader", "nodejs-playwright-tutorial", "bypass-anti-scraping", "handle-dynamic-content", "playwright-network-interception", "browser-automation-scripting", "download-streaming-video", "race-condition-in-web-scraping", "page-evaluate-fetch", "robust-downloader-nodejs", "typescript-project-from-scratch", "debugging-playwright", "advanced-web-scraping-techniques", "playwright-vs-puppeteer", "content-delivery-network-cdn-scraping"]
---

In modern web development, we often encounter tasks that seem simple on the surface but are fraught with hidden challenges. I recently faced such a task: downloading a video from a specific website. This was no simple static page; it heavily utilized JavaScript for dynamic content loading and was fortified with rather sophisticated anti-scraping and content protection mechanisms. This blog post will chronicle the entire journey of how I built a robust download tool from scratch using Node.js, TypeScript, and Playwright, step-by-step, to overcome these challenges.

## The Initial Idea and Technology Selection

The mission began with a clear objective: given a webpage URL, I needed to download the main video on that page to my local machine. My first instinct was to inspect the page's source code, hoping to find a direct link to an `.mp4` file. However, reality quickly set in. Upon opening the browser's developer tools, I discovered that the initial HTML document contained no direct URLs to any video files. The video player, its source information, and everything related to it were dynamically generated and injected into the page only after a series of complex JavaScript scripts had finished executing.

This immediately ruled out using traditional HTTP clients like `curl` or `wget`, or even simple Node.js libraries like `axios` or `node-fetch`. These tools can only retrieve the initial HTML skeleton, which is devoid of any video information. They cannot emulate a real user environment, execute JavaScript, and therefore, can never "see" the dynamically generated video player.

I knew at once that the key to solving this problem was to simulate a complete browser environment. I needed a tool that could not only load a webpage but also possess a JavaScript engine to execute all the scripts on the page, render the DOM, and respond to user interactions, just like Chrome or Firefox.

My technology selection process quickly honed in on a few key components:

**Node.js**: As the backend runtime, its powerful file system capabilities and rich ecosystem make it the ideal platform for building this kind of automation tool.

**TypeScript**: I chose TypeScript over native JavaScript primarily for the long-term maintainability and robustness of the project. TypeScript's static type checking can catch a vast number of potential errors during the development phase, such as typos and type mismatches. For a project dealing with complex browser APIs and network data streams, type safety dramatically improves development efficiency and code quality. This was not just a personal preference, but a sound engineering decision.

**Playwright**: In the realm of browser automation, two main contenders stand out: Puppeteer and Playwright. While both are excellent, I ultimately opted for Playwright. As a more modern tool from Microsoft, it offers superior cross-browser support (Chromium, Firefox, WebKit), a cleaner and more powerful API, and provides incredibly flexible and reliable interfaces for handling waits, interactions, and network interception. I believed that for a site with potentially complex anti-scraping measures, Playwright's advanced network control capabilities would be crucial.

With the tech stack decided, I began setting up the project. This process itself is a showcase of an engineer's foundational skills. I created a new project directory, initialized a `package.json` with `npm init -y`, and installed the core dependency `playwright` along with development dependencies like `typescript`, `ts-node`, and `@types/node`. Next, I generated a `tsconfig.json` file using `npx tsc --init`.

I meticulously configured the `tsconfig.json` file to align with modern Node.js best practices. For instance, I set the `target` to `ES2020` to leverage modern syntax like `async/await`, configured the `module` to `CommonJS` (our initial choice on this journey, which would later evolve), and clearly defined `rootDir` and `outDir` to separate source code from compiled output. These seemingly minor configurations are the first step toward a well-structured and professional project.

## Headless Browser and a Simulated Click

With the project skeleton in place, I started writing the core logic. The core idea of my first version was straightforward: "emulate the user, listen to the network."

I began by designing a `VideoDownloader` class to encapsulate all related operations. This is a good engineering practice that leads to a cleaner, more extensible, and maintainable code structure.

My first step was to implement the most basic user behavior simulation. I wrote several core private methods: `initialize` to launch Playwright and create a new browser page instance; `navigateToPage` to navigate to the user-provided target URL; and `clickPlayButton` to find and click the play button on the page.

The key assumption here was that the real video URL would only begin to load *after* the user clicks the play button. Therefore, my primary task was to locate that button. By inspecting the page's DOM, I identified the CSS selector for the play button, which was a `button` element with specific class names.

But simply clicking the button wasn't enough. The click would trigger a network request for the video file, and I needed to capture it. Playwright's powerful network listening capabilities came into play here. In the `initialize` method, immediately after creating the page instance, I registered a network response listener:

```typescript
// Core Snippet - Registering the response listener
this.page.on('response', this.handleResponse.bind(this));
```

This listener would capture every single network response generated by the page. The `handleResponse` method was where I implemented the core filtering and data processing logic.

Inside this method, I had to accurately identify the video segments from potentially hundreds of network requests. By observing the network panel in the browser's developer tools, I discovered several key characteristics of the video requests:

First, the request URL contained a specific file extension, such as `.mp4`.
Second, the video was typically loaded in chunks to support seeking and save bandwidth. This chunked loading is implemented in the HTTP protocol via the `Range` request header and the status code `206 Partial Content`. The server's response headers would include a `Content-Range` field, which explicitly states the segment's position within the entire video file and the total file size.

Based on these observations, I established my filtering logic: I would only process responses whose URLs contained a specific keyword (like `.mp4`) and had an HTTP status code of `206`.

The next challenge was how to handle these captured video chunks, which could arrive out of order. Simply pushing them into an array would likely result in a corrupted final file. The correct approach was to store them based on their position in the complete video. The `Content-Range` header provided this crucial piece of information, for example, `bytes 0-5242879/906098800`, indicating a data block starting at byte `0`.

Consequently, I decided to use a `Map` data structure to store the segments, using the starting byte of the chunk as the `key` and the `Buffer` containing the binary data as the `value`.

```typescript
// Core Snippet - Storing chunks using a Map
private videoChunks: Map<number, Buffer> = new Map();

// Inside handleResponse
const rangeMatch = contentRange.match(/bytes (\d+)-(\d+)\/(\d+)/);
if (rangeMatch) {
    const start = Number.parseInt(rangeMatch[1], 10);
    const buffer = await response.body(); // This was an early mistake, as we'll see
    this.videoChunks.set(start, buffer);
}
```

The advantage of this approach is that regardless of the order in which the chunks arrive, I can place them precisely where they belong.

The final step was file assembly. Once all chunks were downloaded, I needed to write them to a local file in the correct order. For efficiency and memory conservation, I opted against concatenating all chunks into one giant `Buffer` in memory. Instead, I leveraged Node.js's Stream API. I created an `fs.createWriteStream`, then iterated over the sorted keys of my `Map` (the chunk start bytes), and sequentially fetched the corresponding `Buffer` for each key to write to the stream. This stream-based writing approach has a very low memory footprint and can handle massive files, even tens of gigabytes, with ease.

With that, the logic for my first version was complete. It seemed perfect: simulate a user click, listen to the network, filter for video chunks, store them in order, and write them efficiently. I ran the script with confidence.

## Timeouts and the `networkidle` Trap

After running the program, the terminal output stalled at "Navigating to page...", and a minute later, a bright red error message announced the failure of my first attempt: `page.goto: Timeout 60000ms exceeded. waiting until "networkidle"`.

A timeout error. This is an extremely common issue in automation and web scraping. I immediately reviewed my `navigateToPage` method:

```typescript
// Early, incorrect code
await this.page.goto(this.url, { waitUntil: 'networkidle', timeout: 60000 });
```

The problem was `waitUntil: 'networkidle'`. This option tells Playwright to consider the navigation successful only when the page has loaded and there has been no new network activity for at least 500 milliseconds.

This option is useful for simple, static pages that become "quiet" after loading. However, for the modern, dynamic video site I was dealing with, this was an almost impossible condition to meet. The frontend applications of such sites are highly complex and engage in continuous background network activity:

*   Sending user behavior analytics and tracking data.
*   Periodically refreshing ad content.
*   Maintaining connections with the server via long-polling or WebSockets for real-time updates.
*   Pre-loading images or other media resources.

In this environment, the page's network activity almost never goes "idle" for more than 500 milliseconds. My program waited patiently for the full 60 seconds, only to inevitably time out.

This failure made me realize that my waiting strategy had to be more precise. I didn't need to wait for the entire page to fall silent; I only needed to wait until the **key element** I intended to interact with—the play button—was present and available.

I quickly adjusted my strategy. First, I relaxed the `page.goto` waiting condition from `networkidle` to `domcontentloaded`. This option tells Playwright to proceed as soon as the core HTML document is loaded and parsed, without waiting for all images, stylesheets, and async scripts to finish. This made the navigation step fast and reliable.

Then, I placed the real, precise waiting logic inside the `clickPlayButton` method. This method already contained a call to `await this.page.waitForSelector(playButtonSelector, ...)`. This was the correct form of "waiting," as it targeted a specific goal rather than an ambiguous and unattainable network state.

In the process of fixing this issue, I made an additional optimization. I realized that much of the continuous background network activity was related to ad and tracking scripts. These requests not only slowed down the page load and increased the risk of `networkidle` timeouts but were also completely useless for my download task. So, I decided to set up a request interceptor *before* navigation to proactively block these unnecessary requests.

```typescript
// Core Snippet - Blocking unnecessary requests
await this.page.route('**/*', (route) => {
    const url = route.request().url();
    if (url.includes('google-analytics.com') || url.includes('doubleclick.net')) {
        return route.abort();
    }
    return route.continue();
});
```

This small change not only made the page load faster but also made my automation script's environment cleaner and more predictable. It reflected an engineer's mindset shift from just "making it work" to "making it work better."

## A Race Condition

After resolving the navigation timeout, I ran the program again. This time, the logs showed the page loaded successfully, and the play button was clicked. I could almost hear the trumpets of victory. However, a few seconds later, a new, more bizarre error appeared: `Protocol error (Network.getResponseBody): No data found for resource with given identifier`.

The program had successfully captured the video chunk's response (`response` object), but it failed when attempting to retrieve its content (`await response.body()`). The error message was very low-level, pointing directly at the Chrome DevTools Protocol (CDP) layer.

I spent a significant amount of time debugging and understanding this error. At first, I suspected a Playwright bug or some special encryption on the website's part. But after repeated experiments and research, I finally identified the root cause: a classic and tricky "Race Condition."

My code and the browser's internal media processing pipeline were in a high-speed race, and I was losing.

Here's what was happening:
1. The browser initiates a request for a video chunk.
2. The server returns data. Playwright's backend detects this response via the CDP and notifies my Node.js process.
3. The Node.js event loop places a `'response'` event onto its event queue.
4. Meanwhile, the browser, a highly optimized C++ application, doesn't wait for my Node.js script. Its network stack receives the video data and may immediately send it via "zero-copy" or similar mechanisms directly to the GPU or media decoder for playback. For maximum performance, it clears this data from memory as soon as this operation is complete, assuming the data has been "consumed."
5. When the Node.js event loop finally gets around to processing my `'response'` event, my `handleResponse` callback function begins to execute. When the code reaches `await response.body()`, it sends a command back to the browser via CDP: "Please give me the data for that response you just told me about."
6. The browser replies: "Sorry, the thing you're asking for? I already processed and discarded it. The data is gone." And thus, the `No data found for resource` error was thrown.

The delay in this process might be mere milliseconds or even microseconds, but in the face of a high-performance browser kernel, that's more than enough time for the data to vanish. I realized that any form of "post-mortem" listening—any approach that tries to get the data *after* the response has already happened—was inherently at risk of failing.

To win this race, I needed a more proactive, more invasive method. I turned once again to `page.route`. This time, I wouldn't just use it to `abort()` requests; I would use it to completely "proxy" the video requests. My new plan was:
1. Intercept an outgoing video request.
2. **Instead of letting the browser make the request**, I would call `route.fetch()`, which instructs Playwright's backend to perform the request on the browser's behalf.
3. After getting the `APIResponse` object back from `route.fetch()`, I could safely `await response.body()` from it, because this was *my* request, and the data was fully preserved.
4. After capturing and storing the data in my `videoChunks` Map, I would then call `route.fulfill({ response })` to pass a "forged" copy of this response back to the browser, making the page's video player think everything was normal.

This solution was, in theory, perfect. It transformed passive listening into active proxying, fundamentally solving the race condition. I refactored the code and ran it again, full of hope.

It worked! I successfully captured the first, and then the second, chunk of data! However, the joy was short-lived. After downloading a few chunks (about 9MB), the network requests stopped dead. The program timed out after waiting 20 seconds, leaving me with an incomplete file.

Why did the download chain break? Had my `route.fulfill()` call failed to properly "deceive" the player?

To find the truth, I added extremely verbose logging to my interceptor, printing the URL, method, type of every intercepted request, and the complete headers of every video response.

The logs quickly revealed the first stunning discovery: after the play button was clicked, the player initiated **concurrent requests for multiple different video resolutions**! For example, `720P.mp4`, `1440P.mp4`, and `2048P.mp4`. It seemed to be probing to determine the best bitrate for the current network speed.

My code had a critical flaw: I was using a single `activeVideoUrl` variable to lock onto the first video stream I downloaded. When the request for `720P.mp4` arrived first, my program "decided" this was the target and ruthlessly ignored all subsequent requests for `1440P` and `2048P`. However, the player, after its brief probing, may have ultimately decided to continue streaming the `2048P` version. Because my program didn't correctly `fulfill` the `2048P` requests (since it ignored them), the player's state machine was broken, and it ceased all subsequent requests.

I immediately corrected this logic. I stopped being "first-come, first-served" and instead maintained a list of all candidate streams. After the download finished, I would then select the "best" stream to assemble. My initial criterion for "best" was "the one with the most chunks."

Yet, upon running it again, the problem persisted. The logs showed that every resolution stream was requested only for 2 chunks, and then they all stopped. My "select the most chunks" strategy degenerated into "select the first one" in this scenario, which was still wrong.

It was then that I finally grasped the true essence of the problem. It wasn't my selection logic that was flawed; it was that **`route.fulfill()` itself was the problem**. The `APIResponse` object I got from `route.fetch()`, while containing the data, was ultimately a "clone." When passed back to the browser via `route.fulfill()`, it may have lost certain low-level connection handles, timing information, or other metadata critical to the player. This "imperfect" response, while providing the initial few chunks of data, was enough to corrupt the player's delicate internal state, causing it to refuse to request subsequent segments.

## Active Control over Passive Listening

After the failures of both the `page.on('response')` race condition and the `route.fulfill()` state corruption, I was at an impasse. I seemed to be caught in a dilemma: either be too slow, or be too intrusive.

It was at that moment that a completely new idea took shape in my mind. All my previous attempts had revolved around a central theme: **how to "steal" data from requests initiated by the browser**. Whether listening or proxying, I was a "parasite" attached to the browser's behavior.

Why not flip the script? Why couldn't *I* be the one driving the download process?

This idea completely transformed my architecture. The new, final solution was born, divided into two distinct phases: **Reconnaissance** and **Takeover**.

### Reconnaissance
I still used `page.route` to intercept network requests. But this time, its purpose was incredibly simple: **look, don't touch**. When a video chunk request was intercepted, I did nothing but immediately call `route.continue()`, giving the browser an unobstructed path to conduct its own network communications. This ensured the player's state remained absolutely pure and continuous.
Simultaneously, in the background, I quietly used `page.waitForResponse()` to await the completion of the very request I had just allowed to pass. When the response arrived, I parsed from it all the information I needed: the full URL (including the dynamically generated token), the total video size, and the resolution. I stored this information in my `discoveredStreams` Map.
I set a 5-10 second reconnaissance window. During this time, the player would probe all the resolution streams as usual. My recon detector would gather information on all of them. At the end of the window, I would examine the `discoveredStreams` Map and select the stream with the highest resolution. With that, I had all the critical intelligence needed to download the full, high-quality video.

### Takeover
With the recon mission complete, I no longer cared about the browser's subsequent network activities. I initiated a brand new download loop, one that I was in complete control of. In this loop, I would execute the most core and elegant operation of the entire solution: making the browser work for me.

I used the `page.evaluate()` function. This function can execute arbitrary JavaScript code within the context of the browser page. From my Node.js side, I calculated the range of the next chunk I wanted to download, for example, `bytes=0-5242879`, and then passed this range along with the high-resolution URL from the recon phase as arguments to `page.evaluate`.

```typescript
// Core Snippet - Initiating a fetch request inside the browser
const chunkData = await this.page.evaluate(async ({ url, range }) => {
    const response = await fetch(url, { headers: { 'Range': range } });
    if (!response.ok) throw new Error(`Fetch failed: ${response.status}`);
    const buffer = await response.arrayBuffer();
    return { data: Array.from(new Uint8Array(buffer)) };
}, { url, range });
```

The magic of this code lies in the fact that `fetch(url, ...)` is executed **inside the browser**. This means the request:
*   Automatically carries all the current page's cookies and session information.
*   Originates from the same IP address and browser fingerprint.
*   Has completely passed any Cloudflare or other JavaScript-based human verification checks the site might have deployed.

It was a perfect, legitimate request originating from a real user session. The server could not distinguish it from the player's own requests.

The response from `fetch` is an `ArrayBuffer`, an object that cannot be directly passed from the browser back to Node.js. Therefore, I cleverly converted it into a plain JavaScript array of numbers using `Array.from(new Uint8Array(buffer))`, a data structure that can be serialized as JSON and passed across process boundaries.

Back in my Node.js environment, I received this array of numbers, converted it back into a Node.js `Buffer` object with `Buffer.from(chunkData.data)`, and then steadily wrote it to my file stream.

I repeated this process in a `for` loop, sequentially requesting the entire video file chunk by chunk, according to the size I defined, until every last byte was downloaded.

This solution proved to be unassailable. It completely sidestepped the race condition because it was no longer passively listening. It also entirely avoided interfering with the player's state, because it let the browser's business be the browser's, and the download's be mine. I was simply borrowing the browser's "identity" to make my own requests.

## Polishing the Product

Having solved the core download challenge, I didn't stop there. A great engineer doesn't just solve problems; they deliver a great tool. I began to add a series of professional features to the script.

### Command-Line Argument Support
I removed the hardcoded `TARGET_URL` and switched to using Node.js's `process.argv` to parse command-line arguments. Now, a user could simply append any video URL they wanted to download after `npm start`, vastly improving the tool's flexibility. I also added parameter checking; if no URL was provided, the program would print a usage hint and exit gracefully.

### Rich Progress Display
A long download process without any feedback is a terrible user experience. I created a separate `DownloadTracker` helper class dedicated to handling download statistics. It records a timestamp at the start of the download and updates the number of downloaded bytes each time a data chunk is received.
To avoid spamming the console with frequent updates, I used a simple throttling technique: the progress information would update at most once per second. With each update, it would calculate:
*   **Average Download Speed (Mbps)**: `(Total Bytes Downloaded * 8) / Seconds Elapsed / 1024 / 1024`
*   **Time Remaining (seconds)**: `(Total Size - Bytes Downloaded) / Average Speed`
*   **Estimated Time of Arrival (ETA)**: `Current Time + Time Remaining`
    I then formatted this information into a single, clear, continuously refreshing status line, such as:
    `Progress: 25.13% | 216.05MB / 860.00MB | Speed: 123.45 Mbps | ETA: 19:30:45 (55s remaining)`
    This provided excellent real-time feedback to the user.

### Download Retry Mechanism
Networks are unstable. During the download of a large file, the failure of a single chunk due to a temporary network glitch should not terminate the entire task. I added a `while` loop-based retry mechanism to the `downloadChunkWithRetries` logic.
If a `fetch` failed, it wouldn't give up immediately. It would enter a `catch` block where I would increment a retry counter and calculate a wait time using an "exponential backoff" strategy (e.g., wait 2s on the 1st retry, 4s on the 2nd, 8s on the 3rd...). After a short wait, it would attempt to download the **exact same** chunk again. Only after reaching the maximum number of retries (e.g., 5) would the program finally give up and abort the entire download. This mechanism dramatically enhanced the program's robustness in unstable network environments.

### Intelligent File Naming
Finally, I implemented a feature you might have seen in another one of my projects: extracting a meaningful title from the page to use as the filename. I wrote an `extractVideoTitle` method that would search for specific elements on the page in order of priority (first `p[lang="ja"]`, then `h2.mt-16`). If neither was found, it would fall back to using a part of the URL. I also cleaned the extracted title, removing any characters that are illegal in filenames. In the end, the downloaded file was named `[Video Title]-[Resolution].mp4`, making it instantly identifiable.