# Software Requirements Specification (SRS): ClassPulse

Version 0.1 · 22/11/2026 · Sarah Green, Nerya Azani

## 1. Introduction

### 1.1 Purpose
This document defines what ClassPulse must do (functional requirements) and to what standard (non-functional requirements).

### 1.2 Terms
| Term | Meaning |
|------|---------|
| Lesson | A single session of a course, on a specific date |
| Check-in | A student marking themselves present in a lesson |
| Check-in code | A short code embedded in the QR that changes every 30 seconds |
| Headcount | The number of people the LLM counts in a photo of the classroom |
| LLM | Large language model (via the platform's built-in AI call) |

### 1.3 Users (actors)
There are two human users. The lecturer creates courses and lessons, starts attendance and polls, reads feedback, and uses the LLM tools. The student checks in, answers polls, writes feedback, submits homework, and reads reviews.

The third actor is the LLM service, an external system that counts people in photos, summarizes transcripts, and reviews homework.

## 2. General description
A responsive web app, used mainly from the phone (students) and from the laptop connected to the projector (lecturer).
Data is stored in the platform's database. Login is by email (provided by the platform).

## 3. Functional requirements

### F1: Attendance (the main feature)
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1.1 | The lecturer can create a course and add students (manually or by uploading a CSV with name and email). | Must |
| FR-1.2 | The lecturer can open a lesson in a course; the system displays a QR code on the screen. | Must |
| FR-1.3 | The QR contains a link with a check-in code that changes every 30 seconds, so a photo of the QR sent to a friend stops working quickly. | Should |
| FR-1.4 | A student who scans the QR is checked in to the lesson with a timestamp. Scanning again does not create a duplicate. | Must |
| FR-1.5 | The lecturer sees a live list: present / absent / late (after N minutes). | Must |
| FR-1.6 | The lecturer can manually mark or unmark a student's attendance. | Must |
| FR-1.7 | The lecturer can export the attendance of the whole course (students × lessons) as an Excel file, with a legend explaining the symbols V (present), L (late) and X (absent). The export contains attendance only. | Must |
| FR-1.8 | The lecturer can take or upload a photo of the classroom; the system asks the LLM to count and shows the number in the photo versus the number checked in. | Should |
| FR-1.9 | If the difference exceeds a threshold (for example, 2), the system shows a warning. | Should |
| FR-1.10 | On first login the user chooses Student (listed first) or Lecturer. Choosing Lecturer requires the secret lecturer access code, which is verified on the server and never displayed. After 5 wrong attempts at first login, the user becomes a student. At any time a user can change role from the header: lecturer to student freely, student to lecturer with the code (no attempt limit). | Must |
| FR-1.11 | A lecturer can have only one open lesson at a time. A lesson that stays open for more than 4 hours is closed automatically. | Should |
| FR-1.12 | The lecturer can export a single lesson as one Excel file: an attendance sheet with the legend, plus a poll results sheet and a comments sheet, each only if the lesson had polls or comments. Anonymous comments appear as "Anonymous". | Should |
| FR-1.13 | As an alternative to scanning, a student can type the 6-character check-in code shown on the lecturer's screen. The same rules apply as for scanning (enrolled students only, no duplicates, code expires). The student's check-in screen explains both options. | Should |




### F2: Feedback and suggestions
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-2.1 | A student can write a short comment (up to 280 characters) for a lesson while it is open. | Should |
| FR-2.2 | A student can choose to post anonymously (the name is hidden from the lecturer's view). | Should |
| FR-2.3 | Students can "+1" a comment; the lecturer's view is sorted by number of +1s. | Could |
| FR-2.4 | The lecturer can mark a comment as "answered". | Could |

### F3: Quick polls
| ID     | Requirement | Priority |
|--------|-------------|----------|
| FR-3.1 | The lecturer can create a poll with one question and 2 to 5 answers in under 30 seconds. | Must |
| FR-3.2 | Open polls appear automatically on the screens of students in that lesson. | Must |
| FR-3.3 | Each student can vote once per poll. | Must |
| FR-3.4 | The lecturer sees live results as a bar chart and can close the poll. | Must |
| FR-3.5 | (Optional) The LLM suggests a poll question based on a topic the lecturer types. | Could |
| FR-3.6 | After a poll is closed, the system shows the lecturer the checked-in students who did not vote, and the lecturer can unmark their attendance. | Could |
| FR-3.7 |A lesson can have only one open poll at a time. Opening a new poll requires closing the current one. Closed polls remain visible as read-only results. | Should |


### F4: Video summary
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-4.1 | The lecturer can upload a transcript file (.txt / .vtt exported from Zoom / Meet) for a lesson. | Could |
| FR-4.2 | The system returns an LLM summary: main topics, key concepts, open questions, suggested review questions. | Could |
| FR-4.3 | The summary can be published to the students of the course. | Could |

### F5: Homework review
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-5.1 | The lecturer creates an assignment with a description and a short rubric. | Could |
| FR-5.2 | A student submits text or a link to a repo / file. | Could |
| FR-5.3 | The LLM reviews the submission against the rubric and returns strengths, issues, and a suggested grade. | Could |
| FR-5.4 | The lecturer can run a comparison across all submissions: common mistakes, standout solutions, and suspiciously similar pairs. | Could |
| FR-5.5 | A student sees only their own review and anonymous class-wide statistics. | Could |

## 4. Non-functional requirements
| ID | Requirement |
|----|-------------|
| NFR-1 Usability | A student completes check-in in at most 2 taps after scanning; no installation. |
| NFR-2 Performance | Check-in confirmation arrives within 2 seconds; the lecturer's list updates within 5 seconds. |
| NFR-3 Capacity | Supports a class of at least 60 students checking in within 2 minutes. |
| NFR-4 Compatibility | Works on current Chrome / Safari on Android and iPhone. |
| NFR-5 Privacy | National ID numbers are not stored. Anonymous comments do not reveal the name to the lecturer. Students cannot see other students' homework or reviews. |
| NFR-6 Security | Only the course lecturer can export data or see the full attendance. The lecturer role requires the secret lecturer access code (FR-1.10), checked on the server. Security is at demo level (see Constraints). A check-in code expires after 30 seconds. |
| NFR-7 AI reliability | Every LLM result is shown as a suggestion the lecturer can edit; nothing changes automatically based on the result. |
| NFR-8 Language | English interface (Hebrew optional). |

## 5. Data model (summary)
`Course`, `Student`, `Enrollment`, `Lesson`, `AttendanceRecord`, `HeadcountCheck`, `Comment`, `Poll`, `Vote`, `LessonSummary`, `Assignment`, `Submission`, `Review`. See [architecture.md](architecture.md#3-class-diagram).

## 6. Constraints
- Built on Base44 (managed database, login, and LLM integration); no server of our own.
- No external Zoom / Moodle API.
- Security is at demo level. The lecturer access code is a shared secret, and after the first login there is no limit on wrong attempts, so a determined user could guess it. This is an accepted limitation for a classroom demo; production hardening is out of scope (see the SoW).
- Anonymous comments are anonymous in the display only: the author's name and email are still stored in the database and are not shown to the lecturer or other students. Full anonymity is out of scope for the demo.
