### Product Manager Review 

Here is the Product Manager review of the "Learning Platform-Assignment" PRD.

**Overall Assessment:**
This is a remarkably focused and well-scoped PRD. The decision to strip away the "Student" user role to ensure the MVP can be delivered by a small team in a single semester is a tough but excellent product decision.

Here is the evaluation based on the requested criteria:

* **Is the problem specific and supported by a clear target user?**
**Yes.** The problem of scattered assignment details causing administrative overhead is clearly defined. The target user is aggressively scoped to **Instructors only**, leaving no ambiguity about who we are building this for in the MVP phase.


* **Does the MVP solve the core journey before adding extra features?**
**Yes.** The main journey strictly follows the most critical path: Sign Up/Log In --> Create/Edit Assignment. Extraneous features like webhooks, notifications, and student submission workflows are correctly deferred.


* **Are priorities and out-of-scope items explicit?**
**Yes.** Section 2 clearly lists what is out of scope (e.g., student accounts, public access, mobile apps). This protects the engineering team from scope creep.


* **Are requirements testable and acceptance criteria measurable?**
**Mostly Yes, but with minor gaps.** Acceptance criteria like "At least 6 REST API operations" and "7 automated tests" are highly measurable. However, some Non-Functional Requirements could be tighter.


* **Do business rules reflect real product decisions?**
**Yes.** BR-01 through BR-03 dictate strict data isolation and the absolute lack of unauthenticated access, aligning perfectly with the security needs of an MVP.



**Specific Findings & Recommendations:**

* **PM-01 — Problem & Target User:** Excellent clarity. The problem statement directly correlates with the "Target Users" (Instructors) and "Product Goal". No changes needed.


* **PM-02 — MVP Journey:** The user journey is lean. However, it skips mentioning "Cohort creation" in the Main Journey steps (Section 4), even though Cohorts are required to create Assignments (Section 8). **Recommendation:** Update Section 4 to include a step for creating a Cohort before creating an Assignment.


* **PM-03 — Scope Definition:** The out-of-scope items are explicit and realistic for a 3-5 person student team. Moving student views to Future Improvements is a smart PM call.


* **PM-04 — NFR-01 (Performance):** NFR-01 states pages should load in under 2 seconds on a "typical broadband connection". **Recommendation:** Define "typical" with a specific metric (e.g., 10 Mbps down / 3G latency) so QA can accurately test it.


* **PM-05 — FR-03 (Assignment Creation):** FR-03 mentions "criteria/rubric text", but it does not specify if this is plain text or rich text (Markdown/HTML). **Recommendation:** Clarify the data type format to ensure front-end and back-end teams are aligned on text formatting constraints.



### Frontend UX/UI Review 


Here is the review of the "Learning Platform-Assignment" PRD from the perspective of a Frontend UX/UI Designer.


**Overall Assessment:**
The PRD provides a clear, stripped-down feature set focusing purely on the Instructor, which is great for keeping the UI clean. By identifying exactly 3 core screens (Login, Home, Create/Edit Assignment), it gives a strong starting point. However, to design a truly user-friendly experience, we need to address several gaps regarding UI states, responsiveness, and destructive actions.

Here is the evaluation based on the UX/UI checklist:

* **Can users complete the core journey with understandable screens and states?**
**Partially.** The 3 core screens cover the happy path, but the navigation flow between viewing a list of Cohorts and viewing Assignments inside a specific Cohort is not fully detailed. Does the Home show all assignments across all cohorts, or just the cohorts themselves?


* **Are empty, loading, error, success, and permission-denied states identified?**
**Only Error states are well-defined.** Section 13 does a great job outlining expected API errors (401, 403, 404) and field-level validation (400). However, empty states, loading skeletons/spinners (despite mentioning a 2-second load time), and success toasts are missing.


* **Is the interface usable on relevant screen sizes?**
**Unclear.** The PRD explicitly moves "Native mobile apps" to Out of Scope, but it does not specify whether the React web frontend must be responsive for mobile web browsers.


* **Could a user make an irreversible or confusing mistake?**
**Yes.** Instructors can delete Cohorts (FR-02) and Assignments (FR-04), but there are no requirements for confirmation modals to prevent accidental deletions.



**Specific Findings & Recommendations:**

* **UX-01 — Empty States:** The PRD defines a Home, but does not provide design requirements for what an Instructor sees immediately after their first login when they have 0 Cohorts, or when they view a Cohort that has 0 Assignments.


* **UX-02 — Destructive Actions:** FR-02 and API-04 state an instructor can delete a Cohort. The PRD lacks a requirement for a confirmation state. Furthermore, it is unclear from a UX/Backend perspective if deleting a Cohort cascades to delete all assignments inside it, which could be a catastrophic user error.


* **UX-03 — Responsiveness:** While native apps are out of scope, instructors often check platforms via their mobile phone browsers. The PRD needs to clarify if mobile-responsive web design is required for the 3 core screens.


* **UX-04 — Success Feedback:** Section 13 covers failure scenarios, but does not define success feedback (e.g., a toast notification when an assignment is successfully saved or updated).


* **UX-05 — Form UX (File Upload):** For API-05/API-08, the PRD notes that file upload happens client-side before the form is submitted. The UI needs a specific state requirement to show upload progress so the user doesn't submit the form while the 5MB attachment is still uploading.



###  Backend API & Database Review


Here is the review of the "Learning Platform-Assignment" PRD from the perspective of a Backend API and Database Developer.


**Overall Assessment:**
The PRD outlines a solid and highly secure architectural approach. By restricting all Firestore interactions to the authenticated Cloud Functions (REST API) and relying on the Firebase Admin SDK, it effectively centralizes business logic and prevents malicious client-side writes. The data model is suitably flat and appropriate for a NoSQL database like Firestore.

However, there are critical gaps in database constraints, API response payloads, and data lifecycle management that need to be addressed before backend implementation can begin.

Here is the evaluation based on the backend checklist:

* **Is every requirement supported by a clear data owner and API responsibility?**
**Yes.** The data owner is strictly the authenticated Instructor. API responsibility is clearly delegated to Firebase Cloud Functions acting as a REST API.


* **Are entity relationships and required fields sufficient?**
**Mostly Yes, but missing constraints.** The relationships (User --> Cohort --> Assignment) are clearly defined. However, it is not explicitly stated which fields (like Description or Criteria) are mandatory versus optional (apart from `attachmentUrl` being optional).


* **What prevents duplicates, invalid states, and race conditions?**
**Not fully addressed.** There are no business rules specifying if an Instructor can create two Cohorts with the exact same name, or two Assignments with the same title. Missing requirements could lead to duplicate data states.
* **Which rules must be enforced on the backend rather than the frontend?**
**Well defined.** The PRD explicitly mandates that ownership authorization (checking `InstructorID` against token `uid`) must happen server-side on every request.


* **Are API inputs, outputs, and error cases clear enough to implement?**
**Partially.** Error cases (400, 401, 403, 404) are beautifully documented. API inputs are lightly sketched (e.g., body parameters for API-05). However, **API response payloads are entirely missing**.



**Specific Findings & Recommendations:**

* **BE-01 — Cascading Deletes:** API-04 allows an Instructor to delete a Cohort. The PRD does not define the database strategy for the child Assignments. **Recommendation:** Specify whether deleting a Cohort requires a cascading delete of all its Assignments via the backend, or if the API should reject the deletion (return 400 or 409) until the Cohort is empty.


* **BE-02 — Missing API Response Schemas:** Endpoints like API-02 (`GET /cohorts`) and API-06 (`GET /assignments`) define the request, but do not specify the JSON response structure. **Recommendation:** Define the exact JSON outputs (e.g., returning an array of objects) so the frontend team can build their interfaces concurrently.


* **BE-03 — Uniqueness Constraints:** Section 8 notes that Email is unique for Users. However, it lacks uniqueness constraints for Cohort names. **Recommendation:** Define whether the backend needs to validate that a `Cohort.Name` is unique per `InstructorID`.


* **BE-04 — Orphaned Storage Files:** API-08 (Update Assignment) and API-09 (Delete Assignment) involve assignments that may have an `attachmentUrl`. **Recommendation:** Specify if the Cloud Function should trigger a deletion of the actual file in Firebase Storage to save free-tier space when an assignment is deleted or an attachment is replaced.


* **BE-05 — Pagination/Limits:** API-02 and API-06 return lists of data. While NFR-01 assumes low concurrent usage, returning unbounded arrays in Firestore can lead to higher read costs. **Recommendation:** Clarify if a maximum limit (e.g., 50 assignments per request) should be enforced by the backend.



### Quality & Security Review 

Here is the review of the "Learning Platform-Assignment" PRD from the perspective of a Quality and Security (QA/Sec) Engineer.

**Overall Assessment:**
From a security standpoint, the PRD establishes a very strong foundation. The "defense-in-depth" approach—using Cloud Functions as a mandatory REST API layer while backing it up with Firestore Security Rules to block direct client access—is an excellent architectural decision that prevents client-side bypasses. Relying on Firebase Auth completely offloads the risk of storing passwords. However, quality assurance processes (like performance testing) and edge-case security controls (like rate limiting and file type restrictions) need tightening.

Here is the evaluation based on the QA/Security checklist:

* **How will each acceptance criterion be tested?**
**Partially addressed.** The PRD specifies using the Firebase Emulator Suite for local testing and requires at least 7 automated tests covering CRUD operations and security rules. However, it does not specify how NFR-01 (page load under 2 seconds) will be tested or measured.


* **Are authentication and authorization clearly separated?**
**Yes.** Section 12 explicitly separates them: Firebase Auth handles authentication (issuing ID tokens), and Cloud Functions handle authorization (verifying the token's UID matches the data owner's ID).


* **Can users access or change only permitted data?**
**Yes.** NFR-02 and BR-02 strictly isolate data so an Instructor can only access their own Cohorts and Assignments. Cross-instructor access is explicitly denied.


* **Are sensitive data, validation, logging, and secrets handled safely?**
**Mostly Yes, but missing logging.** Passwords are not stored in the database. Field-level validation is planned (400 errors). However, there is no mention of an audit log or error logging strategy if the Cloud Functions fail.


* **What happens during network, database, and third-party service failures?**
**Unclear.** Section 13 outlines client errors (4xx) well, but lacks definitions for server-side errors (5xx) or how the frontend should behave if Firebase services experience downtime or timeouts.



**Specific Findings & Recommendations:**

* **QS-01 — File Type Restrictions:** Section 9 and 13 mention capping file uploads to 5MB and handling file type failures, but do not explicitly define *which* file types are allowed. **Recommendation:** Explicitly allowlist safe extensions (e.g., `.pdf`, `.docx`, `.txt`) in both frontend validation and Firebase Storage Security Rules to prevent malicious uploads (like `.exe` or `.sh`).


* **QS-02 — Rate Limiting & Abuse:** The system must run on a free tier ($0 budget), but there is no mention of rate limiting. An authenticated user (or a compromised account) could spam the API, potentially exhausting the free tier limits and incurring costs on the Blaze plan. **Recommendation:** Add a requirement for basic rate limiting or quota alerts on Cloud Functions.


* **QS-03 — NFR Testing (Performance):** NFR-01 requires a 2-second load time, but Acceptance Criteria (Section 17) only focuses on functional and security tests. **Recommendation:** Add an acceptance criterion defining how UI performance will be tested (e.g., using Lighthouse or browser network tabs).


* **QS-04 — Error Logging:** The PRD lacks a strategy for monitoring production bugs. **Recommendation:** Specify that Cloud Functions should log 500-level errors to Firebase Crashlytics or Google Cloud Logging so the student developers can debug production issues.




###  Delivery and Document Review 

Here is the review of the "Learning Platform-Assignment" PRD from the perspective of Delivery and Documentation.

**Overall Assessment:**
From a delivery perspective, this PRD is exceptionally pragmatic. The aggressive scoping—specifically dropping the student-facing features to focus solely on the Instructor—makes the timeline of one semester for 3-5 student developers highly realistic. The PRD is internally consistent, and the architecture matches the team's constraints perfectly. However, there are significant risks regarding deployment operations, environment management, and the credit card requirement for the backend.

Here is the evaluation based on the delivery checklist:

* **Can the group build, deploy, test, and document this within one semester?**
**Yes.** The scope of 3 core screens, 6 API endpoints, and 7 automated tests is highly achievable for a small team in this timeframe.


* **Are hosting, domain, environment variables, free-tier limits, and deployment steps realistic?**
**Partially.** The PRD relies on Firebase's free tier (Spark plan) for hosting, auth, and database, which is realistic. However, it correctly flags that Firebase Cloud Functions require the Blaze plan (pay-as-you-go), which mandates a credit card. Deployment steps are vague (`firebase deploy` from the CLI), and environment variables are not discussed.


* **Are dependencies, risks, and ownership visible?**
**Yes.** Section 16 excellently highlights the critical risks, especially the budget constraint vs. the Blaze plan requirement, and the risk of small team size.


* **Is the PRD internally consistent and understandable by a new team member?**
**Yes.** The document flows logically. The Data Model (Section 8) perfectly aligns with the API endpoints (Section 11) and Business Rules (Section 7).


* **Is there a minimum release plan and a rollback or recovery approach where appropriate?**
**No.** While it mentions merging to 'main' after PR review, it lacks a rollback plan if a bad deployment breaks the live environment.



**Specific Findings & Recommendations:**

* **DD-01 — Deployment Conflicts (CI/CD):** Section 14 mentions deploying via `firebase deploy`. If 3-5 students are running manual deployments from their local laptops, they will overwrite each other's work and security rules. **Recommendation:** Require a basic CI/CD pipeline (e.g., GitHub Actions) to automatically deploy to Firebase Hosting/Functions when code is merged to the `main` branch.


* **DD-02 — Rollback Strategy:** The PRD relies heavily on Firestore Security Rules for data isolation. If a buggy rule is deployed, it could lock out all users or expose data. **Recommendation:** Document a rollback strategy (e.g., keeping previous versions of the `firestore.rules` file in version control and defining a clear command to revert).


* **DD-03 — Environment Configuration:** The PRD does not mention how the React frontend will securely store and share the Firebase config keys among the 3-5 developers. **Recommendation:** Add a brief note on using `.env` files and ensuring they are included in the `.gitignore`.


* **DD-04 — API Documentation:** Requiring an OpenAPI or Postman collection as a shared contract (Section 11) is an excellent practice that will prevent frontend/backend integration bottlenecks.
