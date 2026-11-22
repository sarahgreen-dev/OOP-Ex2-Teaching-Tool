# Testing Strategy: ClassPulse

Version 0.1 · 22/11/2026 · Sarah Green, Nerya Azani

This document explains how we test ClassPulse. It is organized by the components and classes in [architecture.md](architecture.md), and it checks the requirements in the [SRS](SRS.md) and the acceptance criteria in the [SoW](SoW.md).

## 1. Goals

- Prove that the main feature, attendance with a spreadsheet export (F1), works from start to finish.
- Check the rules that are easy to get wrong: no duplicate check-in, one vote per student, a code that expires after 30 seconds, and access limited to the lecturer.
- Make sure the app works for a whole class on phones, because this is what we show in the live demo.

## 2. Approach

ClassPulse is built on Base44, which provides the database, login and hosting. We do not write a separate server, so most of our tests are **manual test cases** that we run in the browser and on real phones. Each test case has an ID, steps and an expected result, so another person can repeat it.

| Level | What it checks | How |
|-------|----------------|-----|
| Class and module rules | A single rule, such as "a second scan creates no duplicate" | Manual test case on one screen, with the data checked in the database view of the platform |
| Component integration | Several parts together, such as scan, save and live list | Two devices: laptop as lecturer, phone as student |
| End to end | A full use case from the [use cases](use-cases.md) | Run the use case as written |
| Acceptance | The four criteria in the SoW | Run before the demo |
| LLM output | Photo count, summary, homework review | Compare with known data, and read the result as a human |

## 3. What we test, by component and class

| Component or class | What to test | Requirements |
|--------------------|--------------|--------------|
| RoleService, `User` | A user can choose Student or Lecturer. A wrong lecturer code is refused. After 5 wrong attempts at first login the user becomes a student. The role can be changed from the header | FR-1.10 |
| CourseService, `Course`, `Enrollment` | A course can be created. Students can be added by hand and from a CSV. A CSV with a missing column is refused | FR-1.1 |
| LessonService, `Lesson` | A lesson opens with a code. The code changes every 30 seconds. Closing a lesson stops check-ins | FR-1.2, FR-1.3 |
| AttendanceService, `AttendanceRecord` | A valid scan is recorded with a time. A second scan creates no duplicate. An expired code is refused. The lecturer can mark and unmark a student | FR-1.4 to FR-1.6 |
| ExportService | The CSV opens in Excel and Google Sheets. It has the right columns and rows. Hebrew names appear correctly. A non-lecturer cannot export | FR-1.7, NFR-6 |
| HeadcountService, `HeadcountCheck` | The count from a photo is shown next to the check-in count. A warning appears when the difference is above the threshold | FR-1.8, FR-1.9 |
| FeedbackService, `Comment` | A comment of up to 280 characters is saved. Longer text is refused. An anonymous comment shows no name. "+1" sorts the list | FR-2.1 to FR-2.4 |
| PollService, `Poll`, `Vote` | A poll with 2 to 5 options can be created. Each student votes once. Results update live. Closing stops voting | FR-3.1 to FR-3.6 |
| SummaryService, `LessonSummary` | A `.txt` or `.vtt` transcript gives a summary with topics, concepts and questions. The lecturer can edit it before publishing | FR-4.1 to FR-4.3 |
| HomeworkService, `Assignment`, `Submission`, `Review` | A review follows the rubric. A student sees only their own review and anonymous class statistics | FR-5.1 to FR-5.5, NFR-5 |

## 4. Test cases for attendance (F1)

Attendance is the main feature, so it has the most detailed tests.

| ID | Test | Steps | Expected result |
|----|------|-------|-----------------|
| TC-1 | Open a lesson | Log in as lecturer, create a lesson | A QR code and a short code are shown |
| TC-2 | Code rotation | Watch the code for one minute | The code changes every 30 seconds |
| TC-3 | Valid check-in | Scan the QR with a phone | A confirmation appears in under 2 seconds. The student appears on the lecturer list in under 5 seconds |
| TC-4 | Duplicate scan | Scan again with the same student | The system says it was already recorded. The list still shows one row |
| TC-5 | Expired code | Open a link from an old QR after more than 30 seconds | The system refuses and asks for the current code |
| TC-6 | Manual unmark | Lecturer unmarks a student | The student disappears from the present list |
| TC-7 | Manual mark | Lecturer marks an absent student | The student appears as present |
| TC-8 | Export one lesson | Click Export | A CSV downloads and opens in Excel or Google Sheets with the right rows |
| TC-9 | Hebrew names | Check in with a Hebrew name, then export | The name appears correctly in the file |
| TC-10 | Student cannot export | Log in as a student and look for the export | No export is available, or the system refuses |
| TC-11 | Many students | 20 or more phones check in within 2 minutes | All appear on the list. No errors |
| TC-12 | Headcount | Upload a class photo from the course folder | The photo count and the check-in count are shown. A warning appears if they differ by more than 2 |

## 5. Test cases for the other features

| ID | Feature | Test | Expected result |
|----|---------|------|-----------------|
| TC-13 | F2 | Post a comment of 280 characters, then of 281 | The first is saved, the second is refused |
| TC-14 | F2 | Post an anonymous comment | The lecturer view shows no name |
| TC-15 | F3 | Create a poll with 2 options, then with 6 | The first is created, the second is refused |
| TC-16 | F3 | One student votes twice | The second vote is refused |
| TC-17 | F3 | Close a poll, then vote | The vote is refused. The final results stay |
| TC-18 | F4 | Upload the example Zoom transcript | A summary appears. The lecturer can edit it |
| TC-19 | F5 | Submit an assignment from the example list | A review appears, based on the rubric |
| TC-20 | F5 | A student opens another student's review | The system does not show it |

## 6. Acceptance tests

These come from the SoW. We run all four before the live demo.

| ID | Criterion | How we check it |
|----|-----------|-----------------|
| AT-1 | A student can register within 15 seconds of the QR code being displayed | Time TC-3 with a stopwatch, from the moment the QR appears |
| AT-2 | The lecturer can download a lesson's attendance as a spreadsheet file | TC-8 |
| AT-3 | A poll can be created and answered in real time during the demo | TC-15, TC-16 and TC-17 on two devices |
| AT-4 | All design documents are in the repo, and the diagrams render correctly on GitHub | Open each file on GitHub and check the diagrams |

## 7. Test data

The course lecturer shared a folder with material that we use:

- A list of students (without ID numbers) for TC-1 and the CSV upload.
- Photos of the class, with a known number of people, for the headcount test (TC-12).
- An example transcript summary made by Zoom, to compare with our summary (TC-18).
- A list of example assignments (repos) for the homework review (TC-19).

## 8. LLM outputs

Results from the LLM can be wrong (SoW, Assumptions). So we do not test them as "right or wrong" only. For each LLM feature we check three things: the result is shown as a suggestion, the lecturer can edit it, and nothing changes in the data without the lecturer's action (NFR-7). For the headcount we also compare the LLM count with the known number of people in each photo.

## 9. Test log

We ran the tests on 22/11/2026 on the demo app, with the lecturer account and two real students on their own devices. "Not run" means we did not test it yet.

| ID | Date | Result | Notes |
|----|------|--------|-------|
| TC-1 | 22/11/2026 | Pass | A lesson opens with a QR code and a 6-character code |
| TC-2 | 22/11/2026 | Partly | The code was different in each screenshot (for example GFCKQY, 855NNZ, 94ZYVE). We did not time the 30-second interval |
| TC-3 | 22/11/2026 | Pass | A second student checked in and appeared on the lecturer list with a time, and the counter rose from 0/3 to 1/3. We did not measure the 2-second limit |
| TC-4 | 22/11/2026 | Pass | A second check-in with the same student gave the normal confirmation, and the counter stayed 1/3. There is no special "already checked in" message |
| TC-5 | | Not run | |
| TC-6 | | Not run | The list says "Tap a name to change", but we did not try it |
| TC-7 | | Not run | |
| TC-8 | 22/11/2026 | Pass | The lesson export downloaded and opened. Columns: Name, Email, Status, Checked in at. The course export also worked, with one column for each lesson |
| TC-9 | 22/11/2026 | Pass | A Hebrew name appeared correctly in both files |
| TC-10 | | Not run | |
| TC-11 | | Not run | We tested with two students, not a full class |
| TC-12 | | Not in the demo | Photo headcount is design only |
| TC-13 | | Not run | The form shows a 280-character counter |
| TC-14 | 22/11/2026 | Pass | An anonymous comment appeared as "Anonymous", and a normal comment showed the name |
| TC-15 | 22/11/2026 | Partly | A poll with 2 options was created. We did not test 6 options |
| TC-16 | 22/11/2026 | Pass | After voting, the student saw "Thanks" again after a refresh, and the poll showed 1 vote |
| TC-17 | 22/11/2026 | Pass | After the lecturer closed the poll, the student could no longer vote. The results stayed on the lecturer screen |
| TC-18 to TC-20 | | Not in the demo | F4 and F5 are design only |
| AT-1 | | Not run | We did not time the check-in with a stopwatch |
| AT-2 | 22/11/2026 | Pass | See TC-8 |
| AT-3 | 22/11/2026 | Pass | A poll was created and answered in real time on two devices |
| AT-4 | 22/11/2026 | Partly | On GitHub, the component diagram, the class diagram and sequence diagrams 4.4 and 4.5 displayed correctly. We still need to check the other diagrams and files |

Notes from the export tests: the course export uses the letters V (present), L (late) and X (absent), and the file has no legend. We explain this in the demo.

Earlier, when we compared the tools, we ran a similar flow on a separate Base44 test project with one account. That test is described in the [tools report](tools-report.md). The results above are from the demo app itself.

## 10. Known limits

- The tests are manual, so they depend on us running them.
- We have not tested with a full class. TC-11 is the first test with many phones, and we will run it in the real classroom if possible.
- Security is at demo level (SRS, Constraints), so we do not test attacks such as guessing the lecturer code.
