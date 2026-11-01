  The panel gave you very clear, constructive, and actionable feedback. Below is a complete, structured breakdown of
  the panel’s critiques, suggestions, and the exact updates you need to make to your system and manuscript.
  ──────
   📋 Summary of Panel Feedback & Actionable System Updates

  ## 1. System Access & Security: Teacher Registration

  • Panel's Observation: On the public homepage, there is a "Register as Teacher" button. The panel strongly pointed
  out that DepEd/school systems cannot have open self-registration for teachers. If anyone can register as a teacher,
  unauthorized users could access sensitive student data.
  • Required Updates:
      [ ] Remove public Teacher Registration: Disable or remove the self-registration form for Mobile Teachers on the
      landing page.
      [ ] Admin Provisioning: Only the Super Admin (or DepEd SDO coordinator) should create, invite, or approve
      teacher accounts.
      [ ] Strict Role-Based Access Control (RBAC):
          • Super Admin (SDO): Manages teacher accounts, centers, announcements, and division-wide reports.
          • Mobile Teachers: Assigned to specific learning centers; manages learners, grades modules, and reviews
          requirements.
          • Learners: Can sign up or be enrolled by teachers during the intake process.


  ──────
  ## 2. Re-evaluating the "Gamified" Concept & Research Title

  • Panel's Observation: Your title and presentation emphasize a "Gamified" system, but the panel pointed out:
      • What you currently built is not a game—it is an interactive quiz / assessment module (multiple choice with
      points). A real game requires graphics, animations, game loops, and interactive game mechanics.
      • ALS Demographics: Many ALS learners are young adults and mature individuals (e.g., 20s, 30s, or older), not
      just small children. A simplistic children's game theme may not fit their learning needs.
  • Required Updates:
      [ ] Manuscript & Title Revision: Either change the title/description to reflect what it actually is (e.g.,
      "Interactive E-Learning and Assessment Platform" or "Computer-Based Learning System") instead of "Gamified", OR
      genuinely implement gamification elements (e.g., badges, unlockable levels, streaks, achievement certificates).
      [ ] In-System Terminology: In the UI, rename "Game" buttons to "Interactive Assessment", "Knowledge Check", or
      "Module Quiz".

  ──────
  ## 3. DepEd ALS Business Process & Enrollment Forms

  • Panel's Observation: The panel noted that the system's intake and enrollment workflow should mirror the actual,
  official DepEd ALS business process rather than a generic web form.
  • Required Updates:
      [ ] Official DepEd Form Integration:
          • Standardize learner intake fields based on DepEd ALS Form 2 (AF2 - Enrollment Form) and the Personal
          Information Sheet (PIS).
          • In your documentation and system, recognize the Functional Literacy Test (FLT), which DepEd uses to assess
          whether an enrollee enters at the Basic Literacy, Elementary, or Junior High School (JHS) level.
      [ ] Center Capacity & Slot Allocation:
          • In San Juan, how many Community Learning Centers (CLCs) exist and how many teachers are deployed?
          • Add logic or status tracking for center enrollment capacity (e.g., showing if a center/teacher can still
          accommodate new learners or is full).


  ──────
  ## 4. Module Submissions & Document Verification

  • Panel's Observation: The proposal where learners simply "take a photo of their handwritten paper answers" and
  upload it was critiqued:
      • Photos are prone to being blurry, illegible, manipulated, or AI-generated.
      • Government document uploads (PSA Birth Certificate, Barangay Certificate, Valid IDs) require proper
      verification and data privacy protection.
  • Required Updates:
      [ ] In-App Digital Submissions: Whenever possible, allow learners to answer module assessments directly in the
      system (automated scoring for quizzes, text input for short answers).
      [ ] Teacher Review & Verification Dashboard:
          • For required file/photo uploads (portfolios or government documents), create a dedicated teacher screen
          where teachers can zoom in, review the document, and set a status: Verified, Pending, or Needs Resubmission
          (with teacher remarks).
      [ ] Data Privacy & Watermarking: Add basic security measures for uploaded official IDs/PSA documents (restrict
      public access, store securely on the server).

  ──────
  ## 5. Mobile App Enhancements (Offline & Progress Tracking)

  • Panel's Observation: The Android mobile app felt "too basic" (just listing modules) compared to the web portal.
  • Required Updates:
      [ ] Progress Persistence & Resume State:
          • If a learner is halfway through a module and their phone closes or runs out of battery, the app must
          remember where they left off (bookmarking/last page read).
          • Add a clear Progress Bar for each module showing completion percentage (e.g., Module 1: 75% completed).
      [ ] Cross-Platform / iOS Considerations:
          • If you are staying with native Android (Kotlin), clearly state and justify this in your Scope and
          Limitations (e.g., based on survey data showing Android device ownership among target ALS learners in San
          Juan). If time permits, consider a responsive web app or Flutter for cross-platform availability.
      [ ] Accessibility:
          • Consider accessibility options for adult/visually impaired learners (adjustable font size, text-to-speech
          for module passages, high-contrast mode).


  ──────
  ## 6. AI Assistant (Gemini API) & Human Fallback

  • Panel's Observation: You integrated Gemini API for the chatbot. The panel questioned:
      • How do you prevent AI hallucinations or irrelevant responses?
      • What happens when the AI cannot answer an administrative or specific DepEd ALS question?
  • Required Updates:
      [ ] Prompt Grounding / Context Restriction: In your Gemini API system prompt, restrict the AI to answer only
      topics related to ALS modules, schedules, requirements, and general ALS policies.
      [ ] Human-in-the-Loop / Teacher Escalation:
          • When the AI detects uncertainty or when the learner requests real help, provide a button: "Ask My Teacher"
          or "Send Inquiry to Learning Center".
      [ ] Directory & Contact Person: Ensure learning center contact details, teacher names, and office contact
      numbers are complete and accurate.

  ──────
  ## 7. Dynamic News & Admin Announcements

  • Panel's Observation: The news section on the homepage was hardcoded/static.
  • Required Updates:
      [ ] Add a News / Announcement Management Module in the Admin/Teacher portal (CRUD: Create, Read, Edit, Delete
      announcements) so that schedule changes, enrollment dates, and DepEd advisories update dynamically.

  ──────
   🚀 Recommended Prioritization Roadmap

   Priority │ Task                                                                                │ Affected Area
  ──────────┼─────────────────────────────────────────────────────────────────────────────────────┼───────────────────
   High     │ Remove public "Register as Teacher" button; restrict account creation to Admin      │ Web Portal (Auth)
   High     │ Align research title & terms (replace "Gamified" with "Interactive Computer-Based   │ Manuscript & UI
            │ Learning & Assessment")                                                             │
   High     │ Add dynamic Announcements / News CMS in Admin                                       │ Admin Portal
   High     │ Add Teacher Verification Dashboard for student uploads (Approve / Reject / Remarks) │ Teacher Portal
   Medium   │ Implement module progress saving (resume where left off) and progress bars          │ Mobile App
   Medium   │ Align registration fields to DepEd ALS Form 2 (AF2) & add Center capacity limits    │ Registration
   Medium   │ Ground Gemini AI system prompt & add a fallback "Contact Mobile Teacher" option     │ AI Chatbot
   Low      │ Accessibility features (font size scaling, screen-reader compatibility)             │ Mobile & Web UI
