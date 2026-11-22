# Statement of Work (SoW): ClassPulse

## 1. Goal
The goal is to design a small web tool for the classroom and demonstrate it working in a real lesson. Attendance comes first. Polls, feedback, and LLM features come after it.

## 2. Background
See the [README](https://github.com/sarahgreen-dev/OOP-Ex2-Teaching-Tool#1-the-problem).

## 3. Scope

### In scope
1. Design documents: SRS, use cases, component design, class and sequence diagrams, testing strategy.
2. A tools report comparing at least three "vibe coding" tools, each one actually tested.
3. A working demo app (Base44) that includes: QR-based attendance with export to a spreadsheet, quick polls, feedback, and (optionally) counting from a photo.
4. A 10-minute live demo in class.
5. A short summary document.

### Out of scope
- Integration with the Moodle / Zoom / Google Classroom APIs.
- National ID numbers or any other sensitive personal data.
- Production hardening (thousands of users, SLA, backups).

## 4. Deliverables

| # | Deliverable | Format | Due |
|---|-------------|--------|-----|
| D1 | Repo with README (problem and scope) | GitHub | _<18/11/2026>_ |
| D2 | Tools report | `docs/tools-report.md` | _<date>_ |
| D3 | SoW (this document) | `docs/SoW.md` | _<18/11/2026>_ |
| D4 | SRS | `docs/SRS.md` | _<date>_ |
| D5 | Use cases | `docs/use-cases.md` | _<date>_ |
| D6 | Components, class and sequence diagrams | `docs/architecture.md` | _<date>_ |
| D7 | Testing strategy | `docs/testing-strategy.md` | _<date>_ |
| D8 | Demo app | Base44 link + screenshots | 25/11/2026 |
| D9 | Live demo | In class, 10 minutes | _<date>_ |
| D10 | Summary | `docs/conclusions.md` | After the demo |

## 5. Schedule (proposed)

| Days | Work |
|------|------|
| 1-2 | README, SoW, tool evaluation (Base44, Lovable, Claude) |
| 3-4 | SRS, use cases, architecture and diagrams |
| 5-6 | Build the demo in Base44 (attendance and export first) |
| 7 | Polls, feedback, counting from a photo; testing strategy |
| 8 | Dry run of the demo, bug fixes, summary |

## 6. Roles
We work as a pair (Sarah Green and Nerya Azani) and share the design and development. The course lecturer is the "client" and reviews the work, and the classmates are the test users in the demo.

## 7. Assumptions and constraints
- Every student has a smartphone with a camera and a browser.
- The classroom has Wi-Fi or mobile data.
- The tool's free (or student) tier is enough for one class (about 40 users).
- LLM outputs (counts, summaries, reviews) can be wrong; the lecturer always sees the result and can correct it.

## 8. Acceptance criteria
- A student can register within 15 seconds of the QR code being displayed.
- The lecturer can download a lesson's attendance as a spreadsheet file.
- A poll can be created and answered in real time during the demo.
- All design documents are in the repo, and the diagrams render correctly on GitHub.