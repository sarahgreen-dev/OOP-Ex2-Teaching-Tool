# Architecture: ClassPulse
 
Version 0.1 · 22/11/2026 · Sarah Green, Nerya Azani
 
This document describes the main components of ClassPulse, the classes behind the data, and the sequence of messages in the main use cases. It follows the [SRS](SRS.md) and the [use cases](use-cases.md). The diagrams are written in Mermaid, so GitHub renders them directly.
 
## 1. Overview
 
ClassPulse is a responsive web app. Students use it on a phone, and the lecturer uses it on the laptop that is connected to the projector. The app is built on Base44, which provides the database, login, hosting and the call to the LLM. We do not run a server of our own (see SRS, Constraints).
 
## 2. Components
 
```mermaid
flowchart LR
    subgraph Clients["Browsers"]
        LV["Lecturer view<br/>laptop and projector"]
        SV["Student view<br/>phone"]
    end
 
    subgraph Platform["Base44 platform"]
        AUTH["Authentication<br/>and roles"]
        SVC["Backend functions"]
        DB[("Database")]
        LLM["LLM integration"]
    end
 
    LV -->|"HTTPS"| AUTH
    SV -->|"HTTPS"| AUTH
    LV -->|"create lesson, read list, export"| SVC
    SV -->|"check in, vote, comment"| SVC
    SVC --> DB
    SVC -->|"photo, transcript, homework"| LLM
```
 
### 2.1 Component responsibilities
 
| Component | Responsibility | Features |
|-----------|----------------|----------|
| Lecturer view | Shows the QR code, the live list, the poll results and the LLM suggestions. Starts the export | F1 to F5 |
| Student view | Check-in confirmation, polls, comments, homework | F1 to F5 |
| Authentication and roles | Email login, role choice, the lecturer access code (checked on the server) | FR-1.10 |
| Backend functions | All the rules of the app (see 2.2) | F1 to F5 |
| Database | Stores courses, lessons, attendance, polls, comments and reviews | All |
| LLM integration | Counts people in a photo, summarizes a transcript, reviews homework | F1b, F4, F5 |
 
### 2.2 Backend modules
 
We group the backend functions into modules. Each module has one job, and the classes in section 3 are the data they work on.
 
| Module | Job | Requirements |
|--------|-----|--------------|
| RoleService | Chooses the role, verifies the lecturer code, counts wrong attempts | FR-1.10 |
| CourseService | Creates courses, adds students manually or from a CSV | FR-1.1 |
| LessonService | Opens and closes a lesson, creates the rotating check-in code | FR-1.2, FR-1.3, FR-1.11 |
| AttendanceService | Records a check-in once, lets the lecturer mark or unmark a student | FR-1.4 to FR-1.6, FR-1.13 |
| ExportService | Builds the Excel file (.xlsx) of a lesson or of a course | FR-1.7, FR-1.12 |
| HeadcountService | Sends a photo to the LLM and compares the count to the check-ins | FR-1.8, FR-1.9 |
| PollService | Creates polls, accepts one vote per student, closes polls | FR-3.1 to FR-3.7 |
| FeedbackService | Saves comments, anonymous display, +1 and "answered" | FR-2.1 to FR-2.4 |
| SummaryService | Sends a transcript to the LLM and publishes the summary | FR-4.1 to FR-4.3 |
| HomeworkService | Stores submissions, asks the LLM for a review and a comparison | FR-5.1 to FR-5.5 |
 
## 3. Class diagram
 
```mermaid
classDiagram
    class User {
        +String id
        +String name
        +String email
        +Role role
    }
    class Role {
        <<enumeration>>
        student
        lecturer
    }
    class Lecturer
    class Student
    class Course {
        +String id
        +String name
    }
    class Enrollment {
        +String id
    }
    class Lesson {
        +String id
        +String title
        +Date date
        +Status status
        +String currentCode
        +DateTime codeExpiresAt
        +rotateCode()
        +close()
    }
    class AttendanceRecord {
        +String id
        +DateTime checkedInAt
        +AttendanceStatus status
    }
    class HeadcountCheck {
        +String id
        +String photoUrl
        +int llmCount
        +int checkInCount
        +boolean warning
    }
    class Comment {
        +String id
        +String text
        +boolean anonymous
        +int plusOnes
        +boolean answered
    }
    class Poll {
        +String id
        +String question
        +List options
        +PollStatus status
    }
    class Vote {
        +String id
        +int optionIndex
    }
    class LessonSummary {
        +String id
        +String text
        +boolean published
    }
    class Assignment {
        +String id
        +String description
        +String rubric
    }
    class Submission {
        +String id
        +String content
    }
    class Review {
        +String id
        +String strengths
        +String issues
        +String suggestedGrade
    }
 
    User --> Role
    User <|-- Lecturer
    User <|-- Student
    Lecturer "1" --> "*" Course : teaches
    Course "1" --> "*" Lesson : has
    Course "1" --> "*" Enrollment : has
    Student "1" --> "*" Enrollment : enrolled in
    Lesson "1" --> "*" AttendanceRecord : has
    Student "1" --> "*" AttendanceRecord : checks in
    Lesson "1" --> "*" HeadcountCheck : has
    Lesson "1" --> "*" Comment : has
    Student "1" --> "*" Comment : writes
    Lesson "1" --> "*" Poll : has
    Poll "1" --> "*" Vote : receives
    Student "1" --> "*" Vote : casts
    Lesson "1" --> "0..1" LessonSummary : has
    Course "1" --> "*" Assignment : has
    Assignment "1" --> "*" Submission : receives
    Student "1" --> "*" Submission : submits
    Submission "1" --> "0..1" Review : gets
```
 
### 3.1 Rules the classes must follow
 
- An `AttendanceRecord` is unique for a pair of student and lesson, so a second scan creates no duplicate (FR-1.4).
- A `Vote` is unique for a pair of student and poll (FR-3.3).
- A `Comment` with `anonymous = true` is shown without a name. The author is still stored in the database (SRS, Constraints).
- A `Review` is visible only to its own student and to the lecturer (NFR-5).
- Results from the LLM (`llmCount`, `LessonSummary`, `Review`) are suggestions. The lecturer can edit them, and nothing changes automatically (NFR-7).
## 4. Sequence diagrams
 
### 4.1 Open a lesson and rotate the code (UC-3)
 
```mermaid
sequenceDiagram
    actor Lecturer
    participant UI as Lecturer view
    participant LS as LessonService
    participant DB as Database
 
    Lecturer->>UI: Open lesson
    UI->>LS: createLesson(courseId, title)
    LS->>DB: save Lesson (status open)
    LS->>LS: generate check-in code
    LS->>DB: save code and expiry (30 seconds)
    LS-->>UI: lesson and code
    UI-->>Lecturer: show QR code
    loop Every 30 seconds while the lesson is open
        UI->>LS: rotateCode(lessonId)
        LS->>DB: replace code and expiry
        LS-->>UI: new code
        UI-->>Lecturer: show new QR code
    end
```
 
### 4.2 Student checks in (UC-4)
 
```mermaid
sequenceDiagram
    actor Student
    participant UI as Student view
    participant AS as AttendanceService
    participant DB as Database
    participant LV as Lecturer view
 
    Student->>UI: Scan QR code
    UI->>AS: checkIn(lessonId, code)
    AS->>DB: read lesson and current code
    alt Code is wrong or expired
        AS-->>UI: error, scan the current code
        UI-->>Student: show error
    else Student already checked in
        AS-->>UI: normal confirmation, no new record
        UI-->>Student: show confirmation
    else Code is valid
        AS->>DB: save AttendanceRecord with timestamp
        AS-->>UI: success
        UI-->>Student: show confirmation
        AS-->>LV: live update of the list
    end
```
 
### 4.3 Export attendance (UC-6)
 
```mermaid
sequenceDiagram
    actor Lecturer
    participant UI as Lecturer view
    participant ES as ExportService
    participant DB as Database
 
    Lecturer->>UI: Click Export
    UI->>ES: exportExcel(lessonId)
    ES->>ES: check that the user is the course lecturer
    alt Not the course lecturer
        ES-->>UI: refused
        UI-->>Lecturer: show error
    else Allowed
        ES->>DB: read attendance records
        DB-->>ES: records
        ES->>ES: build Excel file
        ES-->>UI: Excel file (.xlsx)
        UI-->>Lecturer: download file
    end
```
 
### 4.4 Headcount from a photo (UC-7)
 
```mermaid
sequenceDiagram
    actor Lecturer
    participant UI as Lecturer view
    participant HS as HeadcountService
    participant LLM as LLM integration
    participant DB as Database
 
    Lecturer->>UI: Upload photo of the class
    UI->>HS: checkHeadcount(lessonId, photo)
    HS->>LLM: count the people in the photo
    LLM-->>HS: llmCount
    HS->>DB: count check-ins of the lesson
    DB-->>HS: checkInCount
    HS->>DB: save HeadcountCheck
    alt Difference above the threshold
        HS-->>UI: counts and a warning
    else Difference is small
        HS-->>UI: counts
    end
    UI-->>Lecturer: show both numbers
    Note over Lecturer,UI: The lecturer decides what to do. Nothing changes automatically.
```
 
### 4.5 Quick poll (UC-9)
 
```mermaid
sequenceDiagram
    actor Lecturer
    actor Student
    participant PS as PollService
    participant DB as Database
 
    Lecturer->>PS: createPoll(lessonId, question, options)
    PS->>DB: save Poll (open)
    PS-->>Student: poll appears on screen
    Student->>PS: vote(pollId, option)
    alt Student already voted
        PS-->>Student: refused
    else First vote
        PS->>DB: save Vote
        PS-->>Lecturer: live results update
    end
    Lecturer->>PS: closePoll(pollId)
    PS->>DB: set status closed
    PS-->>Lecturer: final results and students who did not vote
```
 
## 5. Design decisions
 
| Decision | Reason |
|----------|--------|
| The check-in code is valid for 30 seconds, and only the current code is accepted | A photo of the QR sent to a friend stops working quickly (FR-1.3) |
| One lesson record holds the current code and its expiry | Keeps the design simple and fits the platform database |
| The lecturer role is protected by a secret code checked on the server | Students cannot export data or see the full list (NFR-6) |
| LLM calls go through backend functions, not from the browser | The lecturer sees the result first and can correct it (NFR-7) |
| No server of our own | The platform gives us login, database and LLM calls, so we design only what we can build in a week |
 
