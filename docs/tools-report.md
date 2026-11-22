# Tools Report: Base44 vs Lovable vs Claude

Version 0.1 · 22/11/2026 · Sarah Green, Nerya Azani

## 1. Purpose

The assignment asks us to compare at least three "vibe coding" tools and to test each one in real life. This report records what happened when we gave the same request to three tools, and explains which one we chose for the ClassPulse demo.

## 2. Method

We gave each tool the same prompt, word for word, on 22/11/2026:

```
Build a simple web app called ClassPulse for classroom attendance.
The lecturer creates a lesson and sees a QR code that changes every 30 seconds.
Students scan it with their phone and get checked in (no duplicates).
The lecturer sees a live list of present students and can export it as CSV.
```

For each tool we checked the same things:

1. How long until something worked.
2. Whether the tool needed extra setup (database, login, hosting).
3. Whether the QR code rotates every 30 seconds.
4. Whether a check-in appears on the lecturer's live list.
5. Whether the CSV export downloads and opens.
6. Problems we ran into.

## 3. Results

| Criterion | Base44 | Lovable | Claude (chat) |
|-----------|--------|---------|---------------|
| What we received | A hosted web app with a preview URL | A hosted web app with a preview | A ZIP project and a screenshot |
| Time to a working result | About 8 minutes (13:04 to 13:12, measured) | About 8-10 minutes (estimated from screenshots) | About 3-5 minutes (estimated) |
| Questions asked during the build | Optional "getting to know you" questions, skippable | One question about lecturer login, with a recommended default | None that we noticed |
| Database, login, hosting | Built in, nothing to set up | Built in, nothing to set up | Not included; needs a local server |
| How a student checks in | Logs in with an account | Types name and student number, no account | Types name and student ID |
| QR rotates every 30 seconds | Yes (6-character code also shown) | Yes (countdown shown) | Yes, according to the screenshot (not run by us) |
| Live list of present students | Yes, updated without refresh | Yes, updated without refresh | Shown in the screenshot (sample data, not run by us) |
| CSV export | Yes. Columns: Name, Email, Checked in at. Hebrew names work | Yes. Columns: #, Name, Student number, Date, Check-in time, Lesson | Button shown in the screenshot (not tested) |
| Problems we saw | A "TypeError: Failed to fetch" notice appeared once; it did not stop the app | Prompts for a test user and for browser notifications | No hosted link; needs Node.js and a local address before phones can reach it |

### 3.1 Base44

Base44 built a working app in about eight minutes. It created the lecturer dashboard, the QR screen with a 6-character code that changes every 30 seconds, and the live attendance list. We created a lesson, checked in, and saw our name appear on the list with the time. The exported CSV opened correctly and contained the name, email and check-in time.

Base44 told us itself that it had not verified two things: the downloaded CSV file and a second person scanning from another phone. We tested the CSV ourselves, but we only checked in with one account.

### 3.2 Lovable

Lovable also produced a hosted app. It asked how lecturers should log in and recommended email and password, which we accepted. In its design a student does not need an account: the student enters a name and a student number. This is faster in a classroom, but the student is not identified by email. The lecturer screen shows the QR code with a countdown, a live attendance list, an "End check-in" button and "Export CSV". The exported file contained the lesson name, date and check-in time.

### 3.3 Claude

Claude returned a ZIP file with a project and an image of the lecturer screen. The image shows a QR code with a countdown, a live list and an export button, but the three students in it are sample data created by Claude, not a test we ran. The screen itself warns that the page runs on `localhost`, so phones cannot reach it unless the server is started on a reachable address. We did not install or run the project, so we cannot confirm the full flow. Claude is strong for producing code, but the result is not a ready-to-use hosted app.

## 4. Decision

We chose **Base44** for the ClassPulse demo.

- It gave a working, hosted app in the shortest measured time, with database, login and hosting built in.
- The app worked end to end in our test: QR, check-in, live list and CSV.
- It supports built-in LLM calls, which we need for the photo headcount and the summaries (F1b, F4, F5 in the SRS).
- A student account gives us an email for each student, which the SRS requires (students are identified by name and email).

Lovable is a close second. Its no-account student check-in is quicker in a classroom, and it could be a good choice if the demo only needed attendance. Claude is useful for writing design documents and code, but in this test it did not deliver an app that students could open on their phones.

## 5. Limitations

- Each tool was run once, with one prompt. Results may differ on another run.
- Times were taken from screenshots and clocks, and some are estimates (marked above).
- We checked in with a single account on each tool. We did not test a second phone, many students at once, or the 30-second code expiring.
- We did not run the project that Claude returned.
