---
date: '2025-11-09T22:32:33Z'
draft: false
title: '将我的大学课程表导入个人日历：一次手动 API 探索与 iCalendar 文件生成实践'
summary: '本文详细介绍如何利用浏览器开发者工具分析大学课程表网站的内部 API，通过 Python 脚本将获取到的 JSON 格式课表数据转换为通用的 iCalendar (.ics) 文件，从而一键导入到任何个人日历应用中。'
tags: ['python', 'icalendar', 'ics', 'json', 'api', 'reverse-engineering', 'web-scraping', 'developer-tools', 'network-analysis', 'http-request', 'curl', 'automation', 'data-conversion', 'timetable', 'calendar-import', 'student-project', 'time-management', 'university-life']
---

作为一名学生，高效的时间管理至关重要。我一直希望能将学校的课程表集成到我日常使用的个人日历应用中，这样就能在一个地方统一查看所有日程。然而，学校官方的课程表系统并没有提供“导出到日历”或任何形式的 `.ics` 订阅链接。更麻烦的是，访问这个课程表页面需要经过一个复杂的多因素认证（MFA）流程。

起初，这似乎是一个棘手的自动化问题。任何需要模拟登录的脚本都将因为MFA而变得异常复杂且脆弱。但是，我很快意识到一个关键事实：课程表一旦确定，在整个学期内几乎不会发生变动。这意味着我可能并不需要一个能够实时同步的动态解决方案。一个一次性的、手动的导入过程就完全足够了。

这个认知彻底改变了我的解决思路。问题从“如何自动化一个复杂的登录流程”转变为“如何在一次成功登录后，获取到整个学期的数据，并将其转换为标准日历格式”。这篇博客将详细记录我解决这个问题的全过程，从网络请求的侦查，到 API 的分析，再到最终利用 Python 脚本生成 `.ics` 文件的每一步。

## 深入浏览器开发者工具

我的第一步是弄清楚课程表数据是如何加载到网页上的。现代网页应用大多采用前后端分离的架构，即前端页面只是一个空壳，所需的数据是通过异步 API 请求（通常是 AJAX）从后端服务器获取并动态渲染的。如果真是这样，我或许可以直接找到这个 API，从而绕过复杂的页面解析。

我打开了学校的课程表页面，并按下了 `F12` 键（在 macOS 上是 `Option + Command + I`）启动了浏览器的开发者工具。这是一个强大的工具箱，允许我们审查和调试网页的几乎所有方面。我将重点放在“网络”（Network）选项卡上，它能够记录浏览器与服务器之间的所有通信。

为了触发数据加载，我需要在页面上执行一个操作。课程表页面默认显示当前周的日程。我点击了“下一周”的按钮，切换到下一周的课程安排。正如预期的那样，“网络”面板中立刻出现了一条新的网络请求。

筛选所有请求后，我锁定了一个名为 `get-events` 的请求。它的 URL 类似于：

`https://timetable.university.ac.uk/services/get-events?start=2025-11-10T00%3A00%3A00&end=2025-11-15T00%3A00%3A00&_=1762726887267`

这个发现令我振奋。请求的命名（`get-events`）和参数都极具暗示性。我立即开始剖析这个请求的每一个细节。

### 请求方法与 URL 结构分析

请求使用的是 `GET` 方法，这是客户端向服务器请求数据的标准 HTTP 方法。URL 的结构也相当直观：

*   **主机名**: `timetable.university.ac.uk`，这是课程表服务的域名。
*   **路径**: `/services/get-events`，清晰地表明了这个端点（Endpoint）的功能是获取事件。
*   **查询参数 (Query Parameters)**: 这是 URL 中 `?` 之后的部分，是解开谜题的关键。

### 解构查询参数

我仔细查看了查询参数，它们是以 `key=value` 的形式通过 `&` 分隔的。

*   `start=2025-11-10T00%3A00%3A00`: `start` 参数的值经过了 URL 编码。解码后是 `2025-11-10T00:00:00`。这显然是我所请求周的第一天（周一）的开始时间。`T` 分隔了日期和时间，这符合 ISO 8601 日期时间格式标准。

*   `end=2025-11-15T00%3A00%3A00`: 类似地，`end` 参数解码后是 `2025-11-15T00:00:00`。有趣的是，这似乎是那一周的周六的开始时间，而不是周五的结束时间。这表明该 API 的时间范围查询可能是左闭右开区间 `[start, end)`，即包含 `start` 时间点，但不包含 `end` 时间点。这是一种在编程中非常常见的处理时间范围的方式，可以避免很多边界问题。

*   `_=1762726887267`: 这个参数看起来像是一个 Unix 时间戳（以毫秒为单位）。这是一种常见的技术，被称为“缓存破坏者”（cache buster）。浏览器和代理服务器可能会缓存 `GET` 请求的结果。通过在每次请求时附加一个唯一的、当前时间的参数，可以确保每次请求都被视为一个新的、不同的请求，从而强制服务器返回最新的数据，而不是从缓存中读取。对于我的用例来说，这个参数可能不是必需的，但为了忠实地重放请求，最好还是保留它。

### 请求头的重要性

除了 URL，HTTP 请求头也包含了至关重要的信息。我特别关注了以下几个请求头：

*   `Cookie`: 这一长串看似随机的字符串，是整个操作能够成功的基石。当我通过学校复杂的 MFA 流程登录后，服务器会生成一个会话（Session）并将一个唯一的会话 ID 存储在浏览器的 Cookie 中。之后，浏览器在向该域名发送的每一个请求中都会自动携带这个 Cookie。服务器通过验证这个 Cookie 来确认我的身份，并返回只属于我的课程表数据。这意味着，只要我能拿到这个有效的 Cookie，我就可以在任何能够发送 HTTP 请求的工具（如 `curl` 或 Python 脚本）中模拟我的登录状态，而无需再次进行 MFA 认证。

*   `X-Requested-With: XMLHttpRequest`: 这个请求头是一个事实上的标准，用于标识这是一个 AJAX 请求。一些服务器端的框架会根据是否存在此头来判断请求来源，并可能因此改变响应的行为（例如，一个 AJAX 请求可能返回 JSON 数据，而一个普通的页面请求则返回完整的 HTML）。在模拟请求时，包含这个头是一个好习惯。

*   `Accept: application/json, text/javascript, */*; q=0.01`: 这个头告诉服务器，客户端期望接收什么样格式的响应。在这里，我们明确表示优先接受 `application/json` 格式的数据。

### 分析响应

服务器对这个 `GET` 请求返回了 `200 OK` 状态码，表示请求成功。响应的主体（Response Body）正是我所期望的 JSON 数据。它是一个数组，数组中的每个对象都代表一个课程事件。

我检查了其中一个事件对象的结构，其中包含了所有我需要的信息：

```json
{
  "start": "2025-11-10T10:00",
  "end": "2025-11-10T11:00",
  "activitydesc": "COURSE101 - Introduction to Programming [ON CAMPUS LECTURE]",
  "locationdesc": "Science Building, Lecture Theatre A",
  "uniqueid": "caf4ff456c0dba721bafcfe27f526214",
  "activityname": "COURSE101/LEC/A/02",
  "staffs": [
    { "FullName": "Dr. A. Smith" }
  ]
}
```

*   `start` 和 `end`: 事件的精确开始和结束时间。
*   `activitydesc`: 事件的标题，非常适合作为日历项的摘要（Summary）。
*   `locationdesc`: 上课地点。
*   `uniqueid`: 一个看起来像是哈希值的唯一标识符。这对于日历事件至关重要，因为它可以作为事件的唯一 ID（UID），便于未来的更新或删除操作，避免重复导入。
*   `activityname`: 课程的内部代码，可以作为补充信息。
*   `staffs`: 一个包含教师信息的数组，可以提取出来丰富事件的描述。

至此，第一阶段的侦查工作圆满完成。我已经完全理解了数据是如何被获取的，并且掌握了获取数据所需的所有要素：一个 API 端点、两个关键的查询参数（`start` 和 `end`）以及一个包含身份认证信息的 `Cookie`。

## 获取全学期数据

我的假设是，既然 `start` 和 `end` 参数可以控制请求的时间范围，那么我只需要将这个范围扩大到整个学期，就可以一次性获取所有课程数据。

首先，我需要确定学期的起止日期。这通常可以在学校的校历（Academic Calendar）中找到。我查到了本学期的起止日期，例如，从 2025 年 9 月 22 日到 2025 年 12 月 12 日。

接下来，我需要一种方式来重放（replay）之前捕获到的网络请求，并修改其中的参数。有很多工具可以做到这一点，例如 Postman 或 Insomnia。但对于这种一次性的简单任务，我选择了浏览器开发者工具内置的功能：“Copy as cURL”。

我在 `get-events` 请求上右键点击，选择了“Copy” -> “Copy as cURL (bash)”。这会将整个 HTTP 请求，包括方法、URL、所有请求头以及 Cookie，转换成一个可以在命令行中直接运行的 `curl` 命令。

粘贴到文本编辑器中，我得到了类似这样的长命令：

```bash
curl 'https://timetable.university.ac.uk/services/get-events?start=...' \
  -H 'User-Agent: ...' \
  -H 'Accept: application/json, ...' \
  -H 'Accept-Language: ...' \
  --compressed \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H 'Connection: keep-alive' \
  -H 'Referer: https://timetable.university.ac.uk/' \
  -H 'Cookie: session_auth_cookie=...' \
  -H 'Sec-Fetch-Dest: empty' \
  -H 'Sec-Fetch-Mode: cors' \
  -H 'Sec-Fetch-Site: same-origin'
```

现在，我只需要对这个命令做两处修改：

1.  将 `start` 参数的值修改为学期的开始日期，例如 `2025-09-22T00:00:00`。
2.  将 `end` 参数的值修改为学期的结束日期，例如 `2025-12-13T00:00:00`（为了覆盖 12 月 12 日全天，我将结束日期设为后一天的零点）。

修改后的 URL 部分看起来像这样：

`'https://timetable.university.ac.uk/services/get-events?start=2025-09-22T00%3A00%3A00&end=2025-12-13T00%3A00%3A00&_=...`

然后，我在命令的末尾加上 `> timetable.json`，这将把 `curl` 命令的输出（也就是服务器返回的 JSON 数据）直接重定向并保存到名为 `timetable.json` 的文件中。

我将修改后的完整命令粘贴到我的终端里，按下回车。几秒钟后，命令执行完毕，没有报错。我检查了当前目录，一个名为 `timetable.json` 的文件赫然在列。打开文件，里面是满满当当的 JSON 数据，一个包含了整个学期所有课程事件的巨大数组。

这个时刻是整个项目的突破点。我已经成功地将需要的数据从一个受保护的、不提供导出功能的 web 系统中完整地提取了出来，并保存为结构化的本地文件。现在，剩下的工作就是将这份数据转换成日历应用可以识别的格式。

## 理解 iCalendar (.ics) 格式

我的目标是生成一个 `.ics` 文件，这是 iCalendar 格式的标准文件扩展名。iCalendar 是一种被广泛支持的开放标准（RFC 5545），几乎所有的日历应用，无论是 Google Calendar、Apple Calendar 还是 Outlook，都能识别和导入它。

在开始编写代码之前，我必须先深入了解 `.ics` 文件的语法和结构。它本质上是一个纯文本文件，内容遵循特定的键值对格式。

一个最基础的 `.ics` 文件结构如下：

```ics
BEGIN:VCALENDAR
VERSION:2.0
PRODID:-//My App//EN
...
BEGIN:VEVENT
...
END:VEVENT
BEGIN:VEVENT
...
END:VEVENT
...
END:VCALENDAR
```

*   整个文件由 `BEGIN:VCALENDAR` 和 `END:VCALENDAR` 包裹，定义了一个日历对象。
*   `VERSION:2.0` 是必须的，指明了 iCalendar 的版本。
*   `PRODID` 是一个产品标识符，用于说明是哪个软件生成了这个文件。我可以自定义这个值。
*   日历中可以包含一个或多个事件，每个事件由 `BEGIN:VEVENT` 和 `END:VEVENT` 包裹。

### 时区的重要性

时间处理是日历格式中最复杂也最容易出错的部分。我的课程表时间都是基于英国当地时间的，而英国实行夏令时（BST, British Summer Time）和格林尼治标准时间（GMT, Greenwich Mean Time）。为了确保导入日历后时间不会因为时区转换而出错，我必须在 `.ics` 文件中明确定义时区信息。

iCalendar 规范允许我们通过 `VTIMEZONE` 组件来定义一个完整的时区规则，包括标准时间偏移、夏令时偏移以及切换规则。对于伦敦时区（`Europe/London`），一个标准的 `VTIMEZONE` 定义如下：

```ics
BEGIN:VTIMEZONE
TZID:Europe/London
BEGIN:DAYLIGHT
TZOFFSETFROM:+0000
TZOFFSETTO:+0100
TZNAME:BST
DTSTART:19700329T010000
RRULE:FREQ=YEARLY;BYMONTH=3;BYDAY=-1SU
END:DAYLIGHT
BEGIN:STANDARD
TZOFFSETFROM:+0100
TZOFFSETTO:+0000
TZNAME:GMT
DTSTART:19701025T020000
RRULE:FREQ=YEARLY;BYMONTH=10;BYDAY=-1SU
END:STANDARD
END:VTIMEZONE
```

*   `TZID` 定义了时区的名称。
*   内部的 `DAYLIGHT` 和 `STANDARD` 块分别定义了夏令时和标准时间的规则。
*   `TZOFFSETFROM` 和 `TZOFFSETTO` 定义了与 UTC 的时间偏移量。
*   `RRULE` 定义了夏令时和标准时间开始切换的重复规则（例如，`BYDAY=-1SU` 表示“最后一个周日”）。

在定义好时区后，我就可以在每个事件的时间戳中引用这个时区 ID，从而确保时间的准确性。

### 解构 VEVENT 事件

每个 `VEVENT` 块内部包含了描述一个具体事件的多个属性。根据我从 JSON 数据中提取的信息，我需要关注以下几个关键属性：

*   `DTSTART` 和 `DTEND`: 事件的开始和结束时间。为了绑定我们定义的时区，格式应该是 `DTSTART;TZID=Europe/London:YYYYMMDDTHHMMSS`。注意，这里的日期时间格式是 `YYYYMMDDTHHMMSS`，中间没有任何分隔符。

*   `SUMMARY`: 事件的标题。对应 JSON 中的 `activitydesc`。

*   `LOCATION`: 事件的地点。对应 JSON 中的 `locationdesc`。

*   `DESCRIPTION`: 事件的详细描述。我可以将课程代码（`activityname`）和教师姓名（`staffs`）等附加信息放在这里。在 `.ics` 格式中，换行符需要用 `\n` 来表示。

*   `UID`: 事件的唯一标识符。这是确保日历应用能够区分不同事件、处理更新和防止重复导入的关键。我将直接使用 JSON 中的 `uniqueid` 字段，并在后面附加一个自定义的域名以确保其全局唯一性，例如 `uniqueid@my-university-timetable`。

*   `DTSTAMP`: 一个时间戳，表示这个事件对象被创建或最后修改的时间，格式为 UTC 时间。

有了对 `.ics` 格式的清晰理解，我现在可以开始编写 Python 脚本，将 `timetable.json` 文件中的数据“翻译”成这种格式。

## Python 脚本的实现

我选择使用 Python 来完成这个转换任务，因为它在处理 JSON 和文本文件方面非常强大，并且拥有出色的标准库。整个脚本的逻辑非常直接：读取 JSON 文件，遍历每个事件，然后按照 `.ics` 格式的要求格式化并拼接字符串，最后将结果写入新文件。

### 核心函数设计

我将所有逻辑封装在一个名为 `create_ics_file` 的函数中。这个函数接收 JSON 数据和输出文件名作为参数。

函数的第一部分是构建 `.ics` 文件的静态头部，包括 `BEGIN:VCALENDAR`、`PRODID`、`VERSION` 以及我上面准备好的 `VTIMEZONE` 块。我将这些内容存储在一个字符串列表中，后续可以方便地进行拼接。

```python
def create_ics_file(json_data, output_filename):
    ics_content = [
        "BEGIN:VCALENDAR",
        "PRODID:-//[Your Name]//Timetable//EN",
        "VERSION:2.0",
        # ... 其他头部信息 ...
        "X-WR-TIMEZONE:Europe/London",
        # ... VTIMEZONE 块 ...
        "END:VTIMEZONE"
    ]
```

### 遍历与数据提取

接下来是脚本的核心部分：一个循环，用于遍历从 `timetable.json` 中加载的事件列表。在循环内部，我需要安全地从每个事件字典中提取所需的数据。

使用字典的 `.get()` 方法是一个很好的实践。与直接使用 `[]` 访问相比，`.get()` 在键不存在时会返回一个默认值（默认为 `None`），而不是抛出 `KeyError` 异常，这让代码更具鲁棒性。

```python
for event_data in json_data:
    summary = event_data.get('activitydesc', 'No Title')
    start_time_str = event_data.get('start')
    end_time_str = event_data.get('end')
    location = event_data.get('locationdesc', '')
    unique_id = event_data.get('uniqueid', '')
```

### 处理教师信息和描述

教师信息被嵌套在 `staffs` 列表里。我需要遍历这个列表，提取出 `FullName`，并将他们拼接成一个字符串。一个列表推导式（list comprehension）可以优雅地完成这个任务。

```python
staff_list = [
    staff['FullName'] for staff in event_data.get('staffs', [])
    if staff.get('FullName')
]
```
我将课程代码和教师列表组合成一个多行的描述字符串。iCalendar 规范要求换行符是字面上的 `\n`，所以在 Python 字符串中，我需要使用 `\\n` 来转义。

```python
description_parts = []
if activity_name:
    description_parts.append(activity_name)
if staff_list:
    description_parts.append("Staff: " + ", ".join(staff_list))
description = "\\n".join(description_parts)
```
### 时间和日期的格式化

这是最需要注意细节的一步。JSON 中的时间格式是 ISO 8601（`YYYY-MM-DDTHH:MM`），而 `.ics` 文件需要的是 `YYYYMMDDTHHMMSS` 格式。Python 的 `datetime` 模块是处理这个问题的完美工具。

首先，我使用 `datetime.fromisoformat()` 将字符串解析成一个 `datetime` 对象。然后，我使用 `.strftime()` 方法将这个对象格式化为目标字符串。

```python
start_dt = datetime.fromisoformat(start_time_str).strftime('%Y%m%dT%H%M%S')
end_dt = datetime.fromisoformat(end_time_str).strftime('%Y%m%dT%H%M%S')
```

### 拼接 VEVENT 块

提取并格式化好所有信息后，我就可以将它们组装成一个完整的 `VEVENT` 字符串块，并添加到 `ics_content` 列表中。f-string 是 Python 中格式化字符串的现代且高效的方式。

```python
ics_content.extend([
	"BEGIN:VEVENT",
	f"DTSTART;TZID=Europe/London:{start_dt}",
	f"DTEND;TZID=Europe/London:{end_dt}",
	f"DTSTAMP:{dtstamp}",
	f"UID:{unique_id}@university-timetable",
	f"SUMMARY:{summary}",
	f"DESCRIPTION:{description}",
	f"LOCATION:{location}",
	"END:VEVENT"
])
```

## 成果

循环结束后，`ics_content` 列表中就包含了完整的日历数据。我最后添加 `END:VCALENDAR` 标记，然后使用 `"\n".join()` 将列表中的所有行拼接成一个单一的字符串，并将其写入指定的输出文件中。

使用 `with open(...)` 语句来处理文件 I/O 是一个好习惯，它能确保无论在写入过程中是否发生错误，文件最终都会被正确关闭。同时，明确指定 `encoding='utf-8'` 可以避免在处理非 ASCII 字符（例如某些教师或地点的名字）时可能出现的编码问题。

```python
ics_content.append("END:VCALENDAR")

with open(output_filename, 'w', encoding='utf-8') as f:
	f.write("\n".join(ics_content))
```

最后，我添加了一个 `if __name__ == "__main__":` 块，这是 Python 脚本的标准入口。它使得这个文件既可以被其他脚本导入作为模块使用，也可以作为主程序直接在命令行中运行。在这个块中，我加入了对 `timetable.json` 文件的读取和错误处理，例如文件未找到或 JSON 格式无效等情况。

我将 `timetable.json` 文件和写好的 Python 脚本放在同一个目录下，然后在终端中运行 `python your_script_name.py`。脚本瞬间执行完毕，并生成了一个 `timetable.ics` 文件。

最后一步是验证成果。我打开了我的日历应用（Google Calendar），选择了“导入”功能，然后上传了刚刚生成的 `timetable.ics` 文件。

成功了！整个学期的课程表，每一节课的准确时间、地点、课程名称和教师信息，都整齐地出现在我的日历上。颜色、提醒等都可以像对待普通日历事件一样进行自定义。

通过这次实践，我不仅解决了一个困扰我很久的实际问题，更重要的是，我完整地经历了一次逆向工程和数据转换的微型项目。这个过程涵盖了 HTTP 协议分析、API 交互、数据处理和标准格式生成等多个方面的技能，证明了通过细致的观察和技术手段，我们能够为自己创造出现有工具所不具备的便利。
