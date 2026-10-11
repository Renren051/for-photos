# SDO San Juan City — ALS Dashboard Metrics Dictionary & Data Sources Specification

This document provides the formal operational specification, calculation formulas, database origins, and scoping rules for all metrics displayed on the **Executive & Mobile Teacher Dashboard** of the DepEd SDO San Juan Alternative Learning System (ALS) Portal.

This specification fulfills the requirements of **Issue #11: [Admin] Validate dashboard metrics and operational summaries** and complies with **DepEd Order No. 27, s. 2017** and **Republic Act No. 11510 (Alternative Learning System Act)**.

---

## 1. Executive Summary & Authorization Architecture

### 1.1 Access Control & Scoping Matrix
Dashboard data is restricted exclusively to authenticated personnel with authorized administrative roles:

| User Role | Authorized Scope | Data Visibility Rules |
| :--- | :--- | :--- |
| **Division Administrator** (`admin`) | Division-wide (SDO San Juan City) | Accesses all 21 barangays, all 12 Community Learning Centers (CLCs), and all registered learners and mobile teachers across San Juan City. Can filter division totals by CLC, Program Track, Date Range, and Enrolment Status. |
| **Mobile Teacher** (`mobile_teacher`) | Center & Learner-Scoped | Strictly filtered to learners assigned directly to the teacher (`enrollments.assigned_teacher_id = user.id`) OR learners enrolled in CLCs where the teacher is designated (`enrollments.clc.assigned_teacher_id = user.id`). System CLC totals reflect assigned centers. |
| **Learner** (`learner`) | Restricted | **HTTP 403 Forbidden**. Cannot access staff dashboard endpoints; redirected to Learner Portal. |
| **Unauthenticated Guest** | None | **HTTP 302 Redirect** to `/login`. |

---

## 2. Metric Dictionary: Primary Learner Status Totals

The dashboard displays six core enrolment status totals agreeing with the underlying `enrollments` table state:

### 2.1 Officially Enrolled (`totalEnrolled`)
- **Metric Code:** `total_enrolled`
- **Display Label:** Officially Enrolled
- **Official Definition:** Out-of-School Youth and Adult (OSYA) learners whose submitted Form 2 intake records, civil registry papers (PSA Birth Certificate), and prior schooling credentials have been reviewed, verified, and officially admitted into the active division masterlist.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'enrolled' 
    AND deleted_at IS NULL
    [AND assigned_teacher_id = :teacher_id OR clc_id IN (:teacher_clcs)]
    [AND calendar_year = :calendar_year]
    [AND DATE(created_at) BETWEEN :date_from AND :date_to]
    [AND clc_id = :clc_id]
    [AND program_track = :track];
  ```
- **Policy Reference:** DepEd Order No. 27, s. 2017; ALS Form 1 Masterlist eligibility.

---

### 2.2 Pending Initial Review (`totalPending`)
- **Metric Code:** `total_pending`
- **Display Label:** Pending Review
- **Official Definition:** Newly encoded or online Form 2 learner registrations submitted via the portal that are waiting for intake queue assessment, document review, and CLC teacher verification.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'pending' 
    AND deleted_at IS NULL;
  ```
- **Operational Trigger:** Incremented immediately upon digital registration wizard completion or walk-in encoding with initial status `pending`.

---

### 2.3 Under Active Review (`totalUnderReview`)
- **Metric Code:** `total_under_review`
- **Display Label:** Under Review
- **Official Definition:** Intake applications actively being validated by mobile teachers or registrars for preliminary placement, functional literacy screening, or requirement completeness.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'under_review' 
    AND deleted_at IS NULL;
  ```
- **Operational Trigger:** When staff moves status from `pending` to `under_review` during intake queue processing.

---

### 2.4 Correction Needed (`totalNeedsCorrection`)
- **Metric Code:** `total_needs_correction`
- **Display Label:** Correction Needed
- **Official Definition:** Applications flagged by reviewing staff with blurry, incomplete, or invalid requirement documents (e.g. invalid PSA or missing barangay certificate). These learners receive notification remarks on their dashboard prompting file re-uploads.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'needs_correction' 
    AND deleted_at IS NULL;
  ```
- **Operational Trigger:** Set by reviewer with mandatory descriptive review remarks.

---

### 2.5 Ineligible / Rejected (`totalRejected`)
- **Metric Code:** `total_rejected`
- **Display Label:** Rejected / Ineligible
- **Official Definition:** Submissions formally rejected due to ineligibility under DepEd A&E standards (e.g. age below 12 for Elementary, below 16 for Junior High School, currently enrolled in formal basic education, or duplicate records). Rejection remarks are preserved for auditability.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'rejected' 
    AND deleted_at IS NULL;
  ```
- **Operational Trigger:** Set by reviewer along with mandatory audit remarks explaining cause of ineligibility.

---

### 2.6 Completed / Graduated (`totalCompleted`)
- **Metric Code:** `total_completed`
- **Display Label:** Completed / Graduated
- **Official Definition:** Learners who have successfully finished their academic curriculum modules and fulfilled portfolio requirements, or passed the official DepEd A&E Certification Exam.
- **Database Table:** `enrollments`
- **Filter / Where Clause:**
  ```sql
  SELECT COUNT(*) FROM enrollments 
  WHERE enrollment_status = 'completed' 
    AND deleted_at IS NULL;
  ```

---

## 3. Operational & System Summaries

### 3.1 Total Processed Intakes (`totalIntakes`)
- **Metric Code:** `total_intakes`
- **Definition:** Total gross intake applications recorded in the system within the selected filter scope and calendar year.
- **Source:** `COUNT(*)` from `enrollments` matching active filters and teacher scope.

### 3.2 San Juan Community Learning Centers (`totalClcs`)
- **Metric Code:** `active_clcs`
- **Definition:** Operational CLCs and barangay learning centers providing ALS sessions in San Juan City.
- **Source:**
  ```sql
  SELECT COUNT(*) FROM community_learning_centers 
  WHERE status = 'active' 
    AND deleted_at IS NULL
    [AND assigned_teacher_id = :teacher_id];
  ```

### 3.3 Published Learning Modules (`totalModules`)
- **Metric Code:** `published_modules`
- **Definition:** Official Self-Learning Modules (SLMs) published across the 6 DepEd ALS Learning Strands for offline downloading and in-app reading.
- **Source:** `SELECT COUNT(*) FROM modules WHERE is_published = 1 AND deleted_at IS NULL;`

### 3.4 Offline Telemetry Sync Packets (`totalTelemetrySyncs`)
- **Metric Code:** `offline_telemetry`
- **Definition:** Total background progress synchronization packets uploaded from the Android LMS mobile app (offline PDF readers) upon reconnecting to Wi-Fi/cellular data.
- **Source:** `COUNT(*)` from `module_progress`.

---

## 4. Program Track Distribution

The dashboard computes distribution across the three DepEd ALS academic tracks:

1. **Basic Literacy Program (BLP):** `enrollments.program_track = 'BLP'`
2. **A&E Elementary:** `enrollments.program_track = 'A&E_Elementary'`
3. **A&E Junior High School:** `enrollments.program_track = 'A&E_Junior_High'`

### Calculation Formulas:
- **Track Percentage:**
  $$\text{Percentage}_{\text{Track}} = \begin{cases} \frac{\text{Count}_{\text{Track}}}{\sum \text{Track Counts}} \times 100 & \text{if } \sum \text{Track Counts} > 0 \\ 0\% & \text{otherwise} \end{cases}$$
- **Zero-State Protection:** When no records exist in the selected period, percentage safely resolves to 0.0% and renders an informative empty state illustration without division by zero.

---

## 5. CLC Allocations Snapshot

- **Table:** `community_learning_centers`
- **Metric Column:** `active_learners`
- **Formula:** Count of active enrolled learners (`enrollments.enrollment_status = 'enrolled'`) mapped to the center, matching active date range and calendar year criteria.
- **Sorting:** `ORDER BY active_learners DESC LIMIT 6`
- **Empty State:** When no active CLCs match the selection, displays clean center alert.

---

## 6. Dashboard Filtering & Date Range Architecture

The dashboard supports full multi-dimensional filtering via HTTP GET query parameters:

| Parameter | Type | Valid Values | Description |
| :--- | :--- | :--- | :--- |
| `date_range` | String | `all`, `today`, `7d`, `30d`, `this_month`, `custom` | Preset date ranges computed against `created_at` |
| `date_from` | Date | `YYYY-MM-DD` | Start date boundary |
| `date_to` | Date | `YYYY-MM-DD` | End date boundary |
| `calendar_year`| String | e.g. `2026`, `2025` | School / Calendar Year filter |
| `clc_id` | Integer | ID from `community_learning_centers` | Specific San Juan CLC |
| `track` | String | `BLP`, `A&E_Elementary`, `A&E_Junior_High` | Specific academic track |
| `status` | String | `enrolled`, `pending`, `under_review`, `needs_correction`, `rejected`, `completed` | Specific status focus |

### Active Filter Badges & Drill-Down:
- Each applied filter renders as a removable chip badge with a 1-click removal link.
- Action links ("View Full Queue &rarr;") pass the active filter parameters to the Enrollment Queue (`admin.enrollments.index`), preserving filter context.

---

## 7. Automated Test Coverage Mapping

All calculations and operational rules are verified by automated tests in `tests/Feature/AdminDashboardMetricsTest.php`:

1. `test_guest_cannot_access_dashboard_and_is_redirected_to_login`
2. `test_learner_cannot_access_dashboard_and_receives_forbidden_response`
3. `test_division_admin_can_access_dashboard_with_all_metrics`
4. `test_mobile_teacher_can_access_dashboard_with_scoped_metrics`
5. `test_kpi_metrics_accurately_reflect_underlying_enrollment_records_by_status`
6. `test_program_track_distribution_counts_agree_with_records`
7. `test_clc_active_learner_counts_agree_with_enrolled_records`
8. `test_dashboard_filters_by_date_range_preset_and_custom_dates`
9. `test_dashboard_filters_by_community_learning_center`
10. `test_dashboard_filters_by_program_track`
11. `test_dashboard_filters_by_enrollment_status`
12. `test_mobile_teacher_only_counts_assigned_learners_and_clcs`
13. `test_dashboard_handles_empty_dataset_gracefully_without_errors`
14. `test_soft_deleted_enrollments_are_excluded_from_metric_calculations`
15. `test_large_numbers_and_zero_percentages_render_clearly`
16. `test_metric_definitions_modal_and_dictionary_data_are_present_in_view`
