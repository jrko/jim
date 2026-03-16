# WebFetch

* If a WebFetch or WebSearch call fails or returns a rate-limit error (429), **stop immediately and ask the user to fetch that URL manually**. Do not continue the task with missing data. Do not defer the ask to the end of the task. The user will paste the content and you can resume.
* **Do not work around failed fetches** by using alternative tools (`gh api`, `curl`, `Bash`, etc.) to download the same content. The point is to stop and ask — not to find another way to fetch it yourself.