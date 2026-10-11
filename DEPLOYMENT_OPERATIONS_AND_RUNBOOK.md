# DepEd SDO San Juan City — ALS SIS & LMS
# Deployment Operations, Runbook & Incident Response Manual

**Document Version:** 1.0.0  
**Target Environment:** Staging & Production (CY 2026 Rollout)  
**Maintained by:** SDO San Juan City Division IT Unit & ALS Technical Working Group  
**Related Issue:** Issue #20: `[Quality] Prepare end-to-end pilot and release readiness`

---

## 1. Environment Architecture & Prerequisites

### 1.1 Technical Stack Specifications
- **Operating System:** Ubuntu Server 22.04 LTS / Windows Server (XAMPP Enterprise)
- **Web Server:** Nginx 1.24+ or Apache 2.4+ (with `mod_rewrite` enabled)
- **PHP Version:** PHP 8.2 or 8.3 (Required extensions: `bcmath`, `ctype`, `curl`, `dom`, `fileinfo`, `filter`, `gd`, `hash`, `intl`, `mbstring`, `openssl`, `pcre`, `pdo_mysql`, `session`, `tokenizer`, `xml`)
- **Database Engine:** MySQL 8.0+ or MariaDB 10.5+ (Default collation: `utf8mb4_unicode_ci`)
- **Package Managers:** Composer 2.6+, Node.js 18.x+, npm 9.x+

### 1.2 Production Environment Variables (`.env` Checklist)

```ini
APP_NAME="SDO San Juan City ALS SIS & LMS"
APP_ENV=production
APP_KEY=base64:... (generated via artisan key:generate)
APP_DEBUG=false
APP_URL=https://als.sdosanjuan.deped.gov.ph

LOG_CHANNEL=daily
LOG_LEVEL=warning
LOG_DEPRECATIONS_CHANNEL=null

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=als_sdosanjuan_prod
DB_USERNAME=als_app_user
DB_PASSWORD=[SECURE_PASSWORD]

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=public
QUEUE_CONNECTION=database
SESSION_DRIVER=file
SESSION_LIFETIME=120
SESSION_ENCRYPT=false
SESSION_PATH=/
SESSION_DOMAIN=null
SESSION_SECURE_COOKIE=true
SESSION_HTTP_ONLY=true
SESSION_SAME_SITE=lax

SANCTUM_STATEFUL_DOMAINS="als.sdosanjuan.deped.gov.ph"
```

---

## 2. Production Deployment Runbook (Acceptance Criterion 3)

Follow this step-by-step checklist during scheduled maintenance windows:

### Pre-Deployment Checklist
1. Notify Division ALS stakeholders and mobile teachers of the maintenance window.
2. Put application into maintenance mode:
   ```bash
   php artisan down --secret="sdo-als-deploy-bypass-2026" --render="errors::503"
   ```
3. Take a pre-deployment database snapshot (see Section 3).

### Step-by-Step Deployment Commands
```bash
# 1. Pull latest verified release from git
git fetch origin
git checkout tags/v1.0.0 -b release-v1.0.0
# Or for preview deployment:
git pull origin preview-deployment

# 2. Install production PHP dependencies
composer install --no-dev --prefer-dist --optimize-autoloader --no-interaction

# 3. Compile front-end assets
npm ci
npm run build

# 4. Execute pending database migrations safely
php artisan migrate --force

# 5. Ensure symbolic link for uploaded documents is active
php artisan storage:link

# 6. Optimize framework caches for production speed
php artisan config:cache
php artisan route:cache
php artisan view:cache
php artisan event:cache

# 7. Restart background queue workers
php artisan queue:restart

# 8. Bring application back online
php artisan up
```

### Post-Deployment Verification
- Perform smoke tests:
  - Visit `/login` and verify assets load via HTTPS.
  - Authenticate as `admin_als` and verify Dashboard operational metrics load.
  - Export a sample DepEd Form 1 masterlist PDF to verify DomPDF rendering.
  - Query health endpoint `GET /api/v1/strands` to verify REST API responsiveness.

---

## 3. Backup & Disaster Recovery Runbook (Acceptance Criterion 3)

### 3.1 Automated Database Backup Script
Create an automated scheduled backup job (`/etc/cron.daily/als_backup.sh`):

```bash
#!/usr/bin/env bash
set -eo pipefail

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/var/backups/als_sdosanjuan"
DB_NAME="als_sdosanjuan_prod"
DB_USER="als_app_user"
DB_PASS="[SECURE_PASSWORD]"

mkdir -p "${BACKUP_DIR}/db"
mkdir -p "${BACKUP_DIR}/storage"

# 1. Dump database with routines and triggers
mysqldump -u "${DB_USER}" -p"${DB_PASS}" --single-transaction --routines --triggers "${DB_NAME}" | gzip > "${BACKUP_DIR}/db/${DB_NAME}_${TIMESTAMP}.sql.gz"

# 2. Archive learner uploaded credentials (non-public storage)
tar -czf "${BACKUP_DIR}/storage/uploads_${TIMESTAMP}.tar.gz" -C /var/www/als_sdosanjuan/storage/app/public documents/

# 3. Retention policy: Remove backups older than 30 days
find "${BACKUP_DIR}/db" -type f -name "*.sql.gz" -mtime +30 -delete
find "${BACKUP_DIR}/storage" -type f -name "*.tar.gz" -mtime +30 -delete

echo "[${TIMESTAMP}] DepEd ALS SDO San Juan backup completed successfully."
```

### 3.2 Step-by-Step Restoration Procedure
If a catastrophic server failure or database corruption occurs:

1. Stop web server traffic:
   ```bash
   php artisan down
   ```
2. Locate the most recent verified backup file in `/var/backups/als_sdosanjuan/db/`.
3. Restore database schema and data:
   ```bash
   gunzip < /var/backups/als_sdosanjuan/db/als_sdosanjuan_prod_20261009_000000.sql.gz | mysql -u root -p als_sdosanjuan_prod
   ```
4. Restore uploaded learner files:
   ```bash
   tar -xzf /var/backups/als_sdosanjuan/storage/uploads_20261009_000000.tar.gz -C /var/www/als_sdosanjuan/storage/app/public/
   ```
5. Clear cached application state and re-cache:
   ```bash
   php artisan cache:clear
   php artisan config:cache
   php artisan route:cache
   ```
6. Verify record integrity with division admin and bring online:
   ```bash
   php artisan up
   ```

---

## 4. Rollback Runbook (Acceptance Criterion 3)

If critical issues or blocking errors emerge immediately post-deployment:

1. Put application into maintenance mode:
   ```bash
   php artisan down --secret="sdo-als-emergency-rollback"
   ```
2. Revert Git codebase to the previous known stable release commit:
   ```bash
   git checkout tags/v0.9.8
   # Or revert commit:
   git checkout HEAD~1
   ```
3. Roll back database migrations if the migration caused schema corruption:
   ```bash
   # Check migration status first
   php artisan migrate:status
   # Roll back last batch
   php artisan migrate:rollback --step=1 --force
   # Alternatively: restore the pre-deploy DB snapshot
   ```
4. Re-install corresponding dependencies and re-cache:
   ```bash
   composer install --no-dev --optimize-autoloader
   npm ci && npm run build
   php artisan optimize:clear
   php artisan config:cache
   php artisan route:cache
   php artisan view:cache
   ```
5. Verify login and dashboard functionality.
6. Bring site back online:
   ```bash
   php artisan up
   ```

---

## 5. Monitoring, Incident Response & Support Matrix (Acceptance Criterion 4)

### 5.1 System Health Monitoring & Logs
- **Application Error Log:** `storage/logs/laravel.log` (Rotated daily, retains 14 days).
- **Web Server Access/Error Logs:** `/var/log/nginx/als_access.log` / `/var/log/nginx/als_error.log`.
- **System Resource Monitoring:** CPU, Memory, Disk Space (warning threshold: 85% disk usage on `/`).

### 5.2 Incident Classification & Response SLAs

| Severity Level | Definition & Examples | Target Response | Target Resolution | Incident Commander |
| :--- | :--- | :---: | :---: | :--- |
| **P1 - Critical** | Full system outage, database unreachable, data loss event. | **< 15 mins** | **< 2 hours** | Division IT Officer |
| **P2 - Major** | Enrollment queue inaccessible, report PDF exports failing division-wide, Android sync endpoint error. | **< 30 mins** | **< 4 hours** | Lead Software Engineer |
| **P3 - Minor** | Isolated learner login failure, UI alignment error, single file upload format issue. | **< 2 hours** | **< 24 hours** | Support Specialist |
| **P4 - Low** | Content update, non-critical report formatting request, cosmetic tweak. | **< 8 hours** | Next Release Cycle | Helpdesk Coordinator |

### 5.3 User Support & Staff Handover Responsibilities

| Role | Contact / Personnel | Primary Operational Responsibility |
| :--- | :--- | :--- |
| **Tier 1: CLC Helpdesk** | Designated ALS Mobile Teachers | First-line intake assistance for learners, walk-in encoding, password resets, basic mobile app troubleshooting. |
| **Tier 2: Division Systems Admin** | SDO San Juan IT Unit Staff | Server maintenance, SSL certificate renewal, backup verification, role assignments, queue debugging. |
| **Tier 3: Engineering Escalation** | ALS Technical Project Lead | Core application bugs, database migration patches, security patches, API contract updates. |
| **Division Oversight** | Dr. Maria Elena Santos, EPSA II | Policy compliance, official ALS Form 1 certification, SDO executive reporting. |

---

## 6. Staff Handover & Training Verification

The following handover materials and operational sessions have been conducted:
1. **Mobile Teacher Portal Training:** Walk-through of `/admin/enrollments`, intake review, remark submission, and mobile offline progress tracking.
2. **SDO Administrator Training:** Dashboard metric monitoring, CLC teacher designation, division-wide masterlist generation, and DomPDF report export.
3. **Learner Onboarding Guides:** Self-service registration instructions distributed to barangay halls and CLCs across San Juan City.
