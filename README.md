# ClassPulse: a KISS teaching tool for the classroom

> Introduction to Software Engineering, Ex2: "Vibe Coding & Deep Designing".
> Authors: Sarah Green and Nerya Azani · Course: Object-Oriented Programming / Semester A (2026–2027) · Demo: [space-class-pulse-live.base44.app](https://space-class-pulse-live.base44.app/)

## 1. The problem

Most lectures we sit in still start the same way: someone passes around a sign-in sheet, or the lecturer reads out names for five minutes. Half the time a friend signs for someone who isn't there, and the sheet ends up in a drawer.

The rest of the lesson has similar gaps. Students who are shy about raising a hand have nowhere to ask a quick question. If the lecturer wants to know whether the class followed a topic, the only option is a full quiz in Moodle, so it usually doesn't happen. Recorded lessons run for an hour and a half, and the summary Zoom generates is pretty thin. Homework feedback comes back weeks later and doesn't tell you how your solution compares to anyone else's.

None of the tools we use today (Moodle, Zoom chat, Google Forms, paper) were built to work with LLMs, even though a lot of this is exactly the kind of thing an LLM is good at.

## 2. The idea

ClassPulse is one small web app that runs in the phone's browser, so nobody has to install anything.
Everything that happens during the lecture lives in one place. There are two roles, lecturer and student.

We tried to keep it as simple as possible. The rule we set for ourselves: once a student has joined, anything they do during the lecture should take two taps at most. If a feature can't meet that, it's probably too complicated for the middle of a lecture.

## 3. Scope

Attendance with a spreadsheet export is the main feature, so it gets built first and gets the most testing. Everything else is ranked below it.

| # | Feature | Priority | In the demo? |
|---|---------|----------|--------------|
| F1 | Attendance: students scan a QR code on the projector, the lecturer sees a live list and can export it to a spreadsheet (CSV / Google Sheets) | Must (main feature) | Yes |
| F1b | Photo headcount: the lecturer takes a photo of the class, an LLM counts the people, and the app compares that to the number of check-ins | Should | Design only |
| F2 | Comments and suggestions during or after class, optionally anonymous | Should | Yes |
| F3 | Quick polls: one question, created in under 30 seconds, results update live | Must | Yes |
| F4 | Video summary: the lecturer uploads a transcript exported from Zoom / Meet and gets a structured summary | Could | Design only |
| F5 | Homework review: an LLM reviews a submission and compares it with the other submissions | Could | Design only |

### Out of scope
These follow the assignment guidelines:
- No integration with the Moodle, Zoom or Google Classroom APIs. Transcripts and files are uploaded by hand instead.
- No ID numbers are stored. A student is identified by name and email.
- No native mobile app, just a responsive web app.

## 4. Repository structure

```
README.md                   this file: problem and scope
docs/SoW.md                 Statement of Work
docs/SRS.md                 Software Requirements Specification
docs/use-cases.md           main use cases / user stories
docs/architecture.md        components, class diagrams, sequence diagrams
docs/testing-strategy.md    testing strategy (per class)
docs/tools-report.md        tools report (Base44 vs Lovable vs Claude)
docs/conclusions.md         results and lessons learned (after the demo)
```

## 5. How to run the demo

The demo is built with Base44, an AI app builder that comes with a database, login and LLM calls. The [tools report](docs/tools-report.md) explains why we picked it.

1. Open the app: [space-class-pulse-live.base44.app](https://space-class-pulse-live.base44.app/)
2. Log in. Choose "Lecturer" and enter the lecturer access code. Create a course and add the students (by hand or with a CSV file).
3. Start a lesson and click "Show QR".
4. Students scan the QR with the phone camera, or tap "Enter lesson code", and they are checked in.
5. The lecturer creates a quick poll, and students vote. Students can also post a comment, anonymously if they want.
6. The lecturer clicks "Export" to download the attendance as a spreadsheet.

## 6. Status

- [x] README (problem and scope)
- [x] Statement of Work
- [x] Tools report
- [x] SRS, use cases, architecture, testing strategy
- [x] Demo app (attendance, polls and comments)
- [ ] Demo in class
- [ ] Conclusions