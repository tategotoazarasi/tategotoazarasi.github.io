---
date: '2025-11-09T22:32:33Z'
draft: false
title: 'Importing My University Timetable into a Personal Calendar: A Hands-On Journey Through Manual API Exploration and iCalendar Generation'
summary: "Learn how to reverse-engineer a university's private timetable API using browser developer tools and write a Python script to convert the JSON data into a universally importable iCalendar (.ics) file for your personal calendar."
tags: ['python', 'icalendar', 'ics', 'json', 'api', 'reverse-engineering', 'web-scraping', 'developer-tools', 'network-analysis', 'http-request', 'curl', 'automation', 'data-conversion', 'timetable', 'calendar-import', 'student-project', 'time-management', 'university-life']
---

As a student, effective time management is paramount. I've always wanted to integrate my university timetable into the personal calendar application I use daily, allowing me to view all my commitments in one unified place. However, the official university timetable system offers no "Export to Calendar" feature or any form of `.ics` subscription link. To complicate matters further, accessing the timetable webpage requires navigating a complex multi-factor authentication (MFA) process.

Initially, this seemed like a daunting automation challenge. Any script attempting to simulate a login would be made incredibly complex and fragile by the MFA requirement. However, I soon had a crucial realization: once the timetable is set for a semester, it almost never changes. This meant I probably didn't need a dynamic, real-time synchronization solution. A one-time, manual import process would be perfectly sufficient.

This insight completely reframed my approach to the problem. The challenge shifted from "how to automate a complex login flow" to "how to, after one successful login, retrieve the entire semester's data and convert it into a standard calendar format." This blog post will document my entire process for solving this problem, from investigating network requests and analyzing the API to finally generating an `.ics` file using a Python script.

## Dive into Browser Developer Tools

My first step was to understand how the timetable data is loaded onto the webpage. Most modern web applications are built with a front-end/back-end separation, where the front-end page is merely a shell. The necessary data is fetched from a back-end server via asynchronous API calls (usually AJAX) and then rendered dynamically. If this were the case, I could potentially find this API directly, bypassing the need for complex page scraping.

I opened the university's timetable page and pressed `F12` (or `Option + Command + I` on macOS) to launch the browser's developer tools. This is a powerful suite of tools that allows for the inspection and debugging of nearly every aspect of a webpage. I focused my attention on the "Network" tab, which records all communication between the browser and the server.

To trigger a data-loading event, I needed to perform an action on the page. The timetable defaults to showing the current week's schedule. I clicked the "Next Week" button to advance to the following week's timetable. As expected, a new network request immediately appeared in the Network panel.

After filtering through the various requests, I zeroed in on one named `get-events`. Its URL looked something like this:

`https://timetable.university.ac.uk/services/get-events?start=2025-11-10T00%3A00%3A00&end=2025-11-15T00%3A00%3A00&_=1762726887267`

This discovery was exhilarating. The request's name (`get-events`) and its parameters were highly suggestive. I immediately began to dissect every detail of this request.

### Analyzing the Request Method and URL Structure

The request used the `GET` method, which is the standard HTTP method for a client to request data from a server. The URL structure was also quite intuitive:

*   **Hostname**: `timetable.university.ac.uk`, the domain of the timetable service.
*   **Path**: `/services/get-events`, clearly indicating that this endpoint's function is to retrieve events.
*   **Query Parameters**: The part of the URL after the `?`, which held the keys to unlocking the data.

### Deconstructing the Query Parameters

I examined the query parameters closely. They were formatted as `key=value` pairs, separated by `&`.

*   `start=2025-11-10T00%3A00%3A00`: The value for the `start` parameter was URL-encoded. After decoding, it became `2025-11-10T00:00:00`. This was clearly the start time of the first day (Monday) of the week I had requested. The `T` separates the date and time, which conforms to the ISO 8601 date-time standard.

*   `end=2025-11-15T00%3A00%3A00`: Similarly, the `end` parameter decoded to `2025-11-15T00:00:00`. Interestingly, this appeared to be the start of Saturday for that week, not the end of Friday. This suggested that the API's time range query likely uses a left-closed, right-open interval `[start, end)`, meaning it includes the `start` timestamp but excludes the `end` timestamp. This is a very common practice in programming for handling time ranges, as it avoids many off-by-one errors and boundary condition issues.

*   `_=1762726887267`: This parameter looked like a Unix timestamp (in milliseconds). This is a common technique known as a "cache buster." Browsers and proxy servers may cache the results of `GET` requests. By appending a unique, time-based parameter to each request, it ensures that every request is treated as new and distinct, forcing the server to return fresh data instead of a cached version. For my use case, this parameter probably wasn't strictly necessary, but to faithfully replay the request, it was best to keep it.

### The Importance of Request Headers

Beyond the URL, the HTTP request headers also contained crucial information. I paid special attention to the following headers:

*   `Cookie`: This long, seemingly random string of characters was the cornerstone that made the entire operation possible. When I logged in through the university's complex MFA process, the server generated a session and stored a unique session ID in my browser's Cookie. Subsequently, the browser automatically includes this Cookie in every request sent to that domain. The server validates this Cookie to confirm my identity and returns the timetable data that belongs only to me. This meant that as long as I could get my hands on this valid Cookie, I could simulate my logged-in state in any tool capable of sending HTTP requests (like `curl` or a Python script) without needing to perform MFA again.

*   `X-Requested-With: XMLHttpRequest`: This header is a de facto standard used to identify an AJAX request. Some server-side frameworks check for the presence of this header to determine the request's origin and may alter their response behavior accordingly (e.g., an AJAX request might return JSON, while a normal page request returns full HTML). It's good practice to include this header when simulating such requests.

*   `Accept: application/json, text/javascript, */*; q=0.01`: This header informs the server about the response formats the client is willing to accept. Here, we are explicitly stating a preference for data in `application/json` format.

### Analyzing the Response

The server responded to this `GET` request with a `200 OK` status code, indicating success. The response body was exactly what I had hoped for: JSON data. It was an array, and each object within the array represented a single class event.

I inspected the structure of one of these event objects and found all the information I would need:

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

*   `start` and `end`: The precise start and end times of the event.
*   `activitydesc`: The title of the event, perfect for the calendar item's summary.
*   `locationdesc`: The location of the class.
*   `uniqueid`: A unique identifier that looks like a hash value. This is critical for calendar events, as it can serve as the event's Unique ID (UID), facilitating future updates or deletions and preventing duplicate imports.
*   `activityname`: The internal code for the course, which could be included as supplementary information.
*   `staffs`: An array containing instructor information, which could be extracted to enrich the event's description.

At this point, the initial investigation was complete. I had a full understanding of how the data was being fetched and had identified all the necessary components: an API endpoint, two key query parameters (`start` and `end`), and a `Cookie` containing my authentication token.

## Fetching Data for the Entire Semester

My hypothesis was simple: if the `start` and `end` parameters control the time range of the request, I just needed to expand this range to cover the entire semester to get all the course data in one go.

First, I needed to determine the start and end dates of the academic semester. This information is typically available in the university's Academic Calendar. I found the relevant dates for the current semester—for instance, from September 22, 2025, to December 12, 2025.

Next, I needed a way to replay the network request I had captured, but with modified parameters. Many tools, such as Postman or Insomnia, are excellent for this. However, for a simple one-off task, I chose to use a feature built directly into the browser's developer tools: "Copy as cURL."

I right-clicked on the `get-events` request in the Network panel and selected "Copy" -> "Copy as cURL (bash)." This action copied the entire HTTP request—including the method, URL, all headers, and the crucial Cookie—to my clipboard as a command-line-runnable `curl` command.

Pasting it into a text editor, I had a long command that looked something like this:

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

Now, I only needed to make two small modifications to this command:

1.  Change the value of the `start` parameter to the semester's start date, e.g., `2025-09-22T00:00:00`.
2.  Change the value of the `end` parameter to the semester's end date, e.g., `2025-12-13T00:00:00` (I set it to the day after the last day of classes to ensure all events on December 12th were included).

The modified URL portion looked like this:

`'https://timetable.university.ac.uk/services/get-events?start=2025-09-22T00%3A00%3A00&end=2025-12-13T00%3A00%3A00&_=...`

Then, I appended `> timetable.json` to the end of the command. This would redirect the output of the `curl` command (the JSON data returned by the server) and save it directly into a file named `timetable.json`.

I pasted the complete, modified command into my terminal and pressed Enter. After a few seconds, the command completed without any errors. I checked my current directory, and there it was: a file named `timetable.json`. Opening it revealed a massive array of JSON data, containing every single course event for the entire semester.

This was the breakthrough moment of the project. I had successfully extracted the data I needed, in its entirety, from a protected web system that offered no export functionality, and saved it as a structured local file. Now, the only remaining task was to convert this data into a format that my calendar application could understand.

## Understanding the iCalendar (.ics) Format

My goal was to generate an `.ics` file, the standard file extension for the iCalendar format. iCalendar is a widely supported open standard (RFC 5545), and virtually all calendar applications—whether it's Google Calendar, Apple Calendar, or Outlook—can recognize and import it.

Before I could start writing code, I had to gain a solid understanding of the syntax and structure of an `.ics` file. It's essentially a plain text file with content that follows a specific key-value pair format.

The basic structure of an `.ics` file is as follows:

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

*   The entire file is wrapped in `BEGIN:VCALENDAR` and `END:VCALENDAR`, defining a calendar object.
*   `VERSION:2.0` is a required property that specifies the iCalendar version.
*   `PRODID` is a product identifier, indicating which software generated the file. I could set this to a custom value.
*   A calendar can contain one or more events, each wrapped in `BEGIN:VEVENT` and `END:VEVENT` blocks.

### The Role of Time Zones

Handling time is one of the most complex and error-prone aspects of calendar formats. My timetable is based on the local time in the UK, which observes both British Summer Time (BST) and Greenwich Mean Time (GMT). To ensure that the event times are correct after importing them into a calendar, I must explicitly define the time zone information within the `.ics` file.

The iCalendar specification allows for the definition of a complete time zone rule set using a `VTIMEZONE` component. This includes the standard time offset, the daylight saving time offset, and the rules for when the transitions occur. For the `Europe/London` time zone, a standard `VTIMEZONE` definition looks like this:

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

*   `TZID` defines the name of the time zone.
*   The inner `DAYLIGHT` and `STANDARD` blocks define the rules for daylight saving time and standard time, respectively.
*   `TZOFFSETFROM` and `TZOFFSETTO` define the offset from UTC.
*   `RRULE` defines the recurrence rule for when the time changes start (e.g., `BYDAY=-1SU` means "the last Sunday").

After defining the time zone, I can reference its `TZID` in each event's timestamp to ensure time accuracy.

### Deconstructing the VEVENT

Each `VEVENT` block contains several properties that describe a specific event. Based on the information I extracted from the JSON data, I needed to focus on these key properties:

*   `DTSTART` and `DTEND`: The start and end times of the event. To bind them to the time zone I defined, the format should be `DTSTART;TZID=Europe/London:YYYYMMDDTHHMMSS`. Note that the date-time format here, `YYYYMMDDTHHMMSS`, has no separators.

*   `SUMMARY`: The title of the event. This corresponds to `activitydesc` in the JSON.

*   `LOCATION`: The location of the event. This corresponds to `locationdesc`.

*   `DESCRIPTION`: A detailed description of the event. I can place additional information here, such as the course code (`activityname`) and instructor names (`staffs`). In the `.ics` format, newlines must be represented as `\n`.

*   `UID`: A unique identifier for the event. This is crucial for calendar applications to distinguish between different events, handle updates, and prevent duplicate imports. I will use the `uniqueid` field from the JSON directly, appending a custom domain to ensure it is globally unique, like `uniqueid@my-university-timetable`.

*   `DTSTAMP`: A timestamp indicating when this event object was created or last modified, formatted as a UTC time.

With a clear understanding of the `.ics` format, I was now ready to write a Python script to "translate" the data from `timetable.json` into this format.

## The Python Script Implementation

I chose Python for this conversion task due to its excellent capabilities for handling JSON and text files, along with its powerful standard library. The script's logic is very straightforward: read the JSON file, iterate over each event, format and concatenate strings according to the `.ics` specification, and finally, write the result to a new file.

### Designing the Core Function

I encapsulated all the logic within a single function, `create_ics_file`, which takes the JSON data and an output filename as its arguments.

The first part of the function constructs the static header of the `.ics` file, including `BEGIN:VCALENDAR`, `PRODID`, `VERSION`, and the `VTIMEZONE` block I prepared earlier. I stored these lines in a list of strings, which would make it easy to join them together later.

```python
def create_ics_file(json_data, output_filename):
    ics_content = [
        "BEGIN:VCALENDAR",
        "PRODID:-//[Your Name]//Timetable//EN",
        "VERSION:2.0",
        # ... other header info ...
        "X-WR-TIMEZONE:Europe/London",
        # ... VTIMEZONE block ...
        "END:VTIMEZONE"
    ]
```

### Iteration and Data Extraction

Next came the core of the script: a loop to iterate through the list of events loaded from `timetable.json`. Inside the loop, I needed to safely extract the required data from each event dictionary.

Using the dictionary's `.get()` method is a best practice here. Unlike direct access with `[]`, `.get()` returns a default value (which is `None` by default) if a key doesn't exist, rather than raising a `KeyError`. This makes the code more robust.

```python
for event_data in json_data:
    summary = event_data.get('activitydesc', 'No Title')
    start_time_str = event_data.get('start')
    end_time_str = event_data.get('end')
    location = event_data.get('locationdesc', '')
    unique_id = event_data.get('uniqueid', '')
```

### Handling Instructor Info and Description

The instructor information was nested within the `staffs` list. I needed to iterate through this list, extract the `FullName`, and join them into a single string. A list comprehension is an elegant way to accomplish this.

```python
staff_list = [
    staff['FullName'] for staff in event_data.get('staffs', [])
    if staff.get('FullName')
]
```
I then combined the course code and the list of instructors into a multi-line description string. The iCalendar spec requires a literal `\n` for newlines, so in my Python string, I needed to use `\\n` to escape the backslash.

```python
description_parts = []
if activity_name:
    description_parts.append(activity_name)
if staff_list:
    description_parts.append("Staff: " + ", ".join(staff_list))
description = "\\n".join(description_parts)
```

### Formatting Dates and Times

This was the step that required the most attention to detail. The time format in the JSON was ISO 8601 (`YYYY-MM-DDTHH:MM`), while the `.ics` file required the `YYYYMMDDTHHMMSS` format. Python's `datetime` module is the perfect tool for this job.

First, I used `datetime.fromisoformat()` to parse the string into a `datetime` object. Then, I used the `.strftime()` method to format this object into the target string.

```python
start_dt = datetime.fromisoformat(start_time_str).strftime('%Y%m%dT%H%M%S')
end_dt = datetime.fromisoformat(end_time_str).strftime('%Y%m%dT%H%M%S')
```

### Assembling the VEVENT Block

With all the information extracted and correctly formatted, I could now assemble it into a complete `VEVENT` block as a series of strings and append them to my `ics_content` list. F-strings are a modern and efficient way to format strings in Python.

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

## The Final Result

After the loop finished, the `ics_content` list contained the complete calendar data. I added the final `END:VCALENDAR` marker, then used `"\n".join()` to concatenate all lines in the list into a single string, which I then wrote to the specified output file.

Using a `with open(...)` statement for file I/O is a good practice, as it ensures the file is properly closed, even if errors occur during the write process. Additionally, explicitly specifying `encoding='utf-8'` prevents potential encoding issues when dealing with non-ASCII characters (such as in some instructor or location names).

```python
ics_content.append("END:VCALENDAR")

with open(output_filename, 'w', encoding='utf-8') as f:
	f.write("\n".join(ics_content))
```

Finally, I added an `if __name__ == "__main__":` block, which is the standard entry point for a Python script. This allows the file to be both imported as a module in other scripts and run directly from the command line as the main program. Within this block, I included the logic for reading the `timetable.json` file and added error handling for cases like the file not being found or containing invalid JSON.

I placed the `timetable.json` file and my completed Python script in the same directory. Then, I ran `python your_script_name.py` in my terminal. The script executed in an instant, and a `timetable.ics` file was generated.

The final step was to verify the result. I opened my calendar application (Google Calendar), selected the "Import" function, and uploaded the `timetable.ics` file I had just created.

It worked perfectly. My entire semester's timetable, with the exact time, location, course name, and instructor for every class, appeared neatly on my calendar. I could now customize colors, set reminders, and manage my academic schedule just like any other event.