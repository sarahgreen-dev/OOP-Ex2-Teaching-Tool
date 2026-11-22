# Use Cases: ClassPulse

Version 0.1 · 22/11/2026 · Sarah Green, Nerya Azani

This document describes the main ways people use ClassPulse. Each use case links to the requirements in the [SRS](SRS.md). Attendance (F1) is the main feature, so it has the most detail.

## 1. Actors

| Actor | Description |
|-------|-------------|
| Lecturer | Creates courses and lessons, shows the QR code, reads attendance, runs polls, and uses the LLM tools |
| Student | Checks in, answers polls, writes comments, and (optionally) submits homework |
| LLM service | External service that counts people in a photo, summarizes a transcript, and reviews homework |

## 2. Use case list

| ID | Use case | Actor | Feature | Priority |
|----|----------|-------|---------|----------|
| UC-1 | Choose a role at first login | Lecturer, Student | F1 | Must |
| UC-2 | Create a course and add students | Lecturer | F1 | Must |
| UC-3 | Open a lesson and show the QR code | Lecturer | F1 | Must |
| UC-4 | Check in to a lesson | Student | F1 | Must |
| UC-5 | Watch and correct live attendance | Lecturer | F1 | Must |
| UC-6 | Export attendance to a spreadsheet | Lecturer | F1 | Must |
| UC-7 | Check the headcount from a photo | Lecturer, LLM service | F1b | Should |
| UC-8 | Post a comment or suggestion | Student | F2 | Should |
| UC-9 | Run a quick poll | Lecturer, Student | F3 | Must |
| UC-10 | Summarize a lesson from a transcript | Lecturer, LLM service | F4 | Could |
| UC-11 | Review homework | Lecturer, Student, LLM service | F5 | Could |

## 3. User stories

- As a **lecturer**, I want students to check in by scanning a code, so that I do not spend five minutes reading names.
- As a **lecturer**, I want the code to change every 30 seconds, so that a photo of it sent to a friend stops working.
- As a **lecturer**, I want to export attendance to a spreadsheet, so that I can keep the records and submit them.
- As a **lecturer**, I want to compare the number of check-ins to a photo of the class, so that I can spot someone who signed in without being present.
- As a **student**, I want to check in with two taps and without installing anything, so that it does not disturb the lesson.
- As a **student**, I want to ask a question anonymously, so that I can ask even if I am shy.
- As a **lecturer**, I want to create a poll in under 30 seconds, so that I can check whether the class followed a topic.

## 4. Detailed use cases

### UC-1: Choose a role at first login
- **Actor:** Lecturer or Student
- **Requirement:** FR-1.10
- **Precondition:** The user opens ClassPulse for the first time and is logged in by email.
- **Main flow:**
  1. The system asks the user to choose Student or Lecturer. Student is listed first.
  2. The user chooses Student. The system opens the student view.
- **Alternative flow, lecturer:**
  1. The user chooses Lecturer and the system asks for the lecturer access code.
  2. The user enters the code. The server verifies it, and the lecturer view opens.
  3. After 5 wrong attempts at first login, the user becomes a student.
- **Postcondition:** The user has a role. The role can be changed later from the header.

### UC-2: Create a course and add students
- **Actor:** Lecturer
- **Requirement:** FR-1.1
- **Precondition:** The user is a lecturer.
- **Main flow:**
  1. The lecturer creates a course with a name.
  2. The lecturer adds students manually (name and email), or uploads a CSV with name and email.
  3. The system saves the course and its students.
- **Alternative flow:** The CSV has a missing column. The system shows an error and adds nothing.
- **Postcondition:** The course has an enrolled student list.

### UC-3: Open a lesson and show the QR code
- **Actor:** Lecturer
- **Requirements:** FR-1.2, FR-1.3
- **Precondition:** The lecturer has a course.
- **Main flow:**
  1. The lecturer opens a new lesson in the course.
  2. The system shows a QR code on the screen, together with a short code.
  3. The code changes every 30 seconds. The lecturer projects the screen to the class.
- **Postcondition:** The lesson is open for check-in.

### UC-4: Check in to a lesson
- **Actor:** Student
- **Requirements:** FR-1.4, NFR-1, NFR-2
- **Precondition:** The lesson is open and the student has a smartphone.
- **Main flow:**
  1. The student scans the QR code with the phone camera.
  2. The system checks that the code is still valid.
  3. The system records the student as present with a timestamp and shows a confirmation within 2 seconds.
- **Alternative flows:**
  - The student scans again. The system says the check-in was already recorded and creates no duplicate.
  - The code has expired (older than 30 seconds). The system rejects it and asks the student to scan the current code.
  - The student is not logged in. The system asks the student to log in, then completes the check-in.
- **Postcondition:** The student appears in the lesson's attendance.

### UC-5: Watch and correct live attendance
- **Actor:** Lecturer
- **Requirements:** FR-1.5, FR-1.6, NFR-2
- **Precondition:** A lesson is open.
- **Main flow:**
  1. The lecturer watches the live list of present, absent and late students. It updates within 5 seconds.
  2. If needed, the lecturer marks or unmarks a student manually (for example, a student whose phone died).
  3. The lecturer closes the check-in when the lesson starts or ends.
- **Postcondition:** The attendance list is final for the lesson.

### UC-6: Export attendance to a spreadsheet
- **Actor:** Lecturer
- **Requirements:** FR-1.7, NFR-6
- **Precondition:** The lesson has attendance records.
- **Main flow:**
  1. The lecturer chooses to export a single lesson, or the whole course (students by lessons).
  2. The system creates a CSV file and downloads it.
  3. The lecturer opens the file in Excel or Google Sheets.
- **Alternative flow:** A user who is not the course lecturer tries to export. The system refuses.
- **Postcondition:** The lecturer has an attendance file.

### UC-7: Check the headcount from a photo
- **Actor:** Lecturer, LLM service
- **Requirements:** FR-1.8, FR-1.9, NFR-7
- **Precondition:** A lesson has check-ins and the lecturer has a photo of the class.
- **Main flow:**
  1. The lecturer takes or uploads a photo of the class.
  2. The system asks the LLM to count the people in the photo.
  3. The system shows the number in the photo next to the number of check-ins.
  4. If the difference is above the threshold (for example, 2), the system shows a warning.
- **Alternative flow:** The LLM count looks wrong. The lecturer ignores it, and nothing changes automatically.
- **Postcondition:** The lecturer has a result to compare, and decides what to do.

### UC-8: Post a comment or suggestion
- **Actor:** Student
- **Requirements:** FR-2.1, FR-2.2, FR-2.3, FR-2.4
- **Precondition:** The student is in a lesson, or the lesson has ended.
- **Main flow:**
  1. The student writes a comment of up to 280 characters.
  2. The student optionally chooses to post anonymously.
  3. The system saves the comment. The lecturer sees it, without the name if it is anonymous.
  4. Other students can "+1" it, and the lecturer can mark it as answered.
- **Postcondition:** The comment appears in the lecturer's list, sorted by number of +1s.

### UC-9: Run a quick poll
- **Actor:** Lecturer, Student
- **Requirements:** FR-3.1 to FR-3.6
- **Precondition:** A lesson is open.
- **Main flow:**
  1. The lecturer creates a poll with one question and 2 to 5 answers, in under 30 seconds.
  2. The poll appears on the screen of every student in the lesson.
  3. Each student votes once.
  4. The lecturer watches the live results as a bar chart, then closes the poll.
- **Alternative flows:**
  - The student tries to vote twice. The system refuses the second vote.
  - The lecturer asks the LLM to suggest a question for a topic. The lecturer can edit the suggestion (FR-3.5).
  - After the poll closes, the system lists checked-in students who did not vote (FR-3.6).
- **Postcondition:** The poll result is stored.

### UC-10: Summarize a lesson from a transcript
- **Actor:** Lecturer, LLM service
- **Requirements:** FR-4.1, FR-4.2, FR-4.3
- **Precondition:** The lecturer has a transcript (.txt or .vtt) exported from Zoom or Meet.
- **Main flow:**
  1. The lecturer uploads the transcript for a lesson.
  2. The system asks the LLM for a summary: main topics, key concepts, open questions and review questions.
  3. The lecturer reads and edits the summary, then publishes it to the students.
- **Postcondition:** Students of the course can read the summary.

### UC-11: Review homework
- **Actor:** Lecturer, Student, LLM service
- **Requirements:** FR-5.1 to FR-5.5, NFR-5
- **Precondition:** The lecturer has created an assignment with a description and a rubric.
- **Main flow:**
  1. The student submits text or a link to a repo or file.
  2. The system asks the LLM to review the submission against the rubric.
  3. The student sees only their own review: strengths, issues and a suggested grade.
  4. The lecturer can run a comparison across all submissions: common mistakes, standout solutions and suspiciously similar pairs.
- **Postcondition:** Each student has a review. Students see only anonymous class-wide statistics, never other students' work.

## 5. Notes

- Every LLM result is a suggestion. The lecturer can edit it, and nothing changes automatically (NFR-7).
- F4 and F5 are "Could" features, so the demo covers them minimally or as design only.
