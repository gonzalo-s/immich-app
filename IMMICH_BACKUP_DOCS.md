# Immich Backup & Automation Strategy

This document provides a comprehensive guide to the backup procedures, automation logic, and manual maintenance flows for your Immich Media Server.

---

## 📂 Infrastructure Overview

These paths and identifiers are used across the backup scripts to ensure data consistency.

| Component               | Path / Value                           |
| :---------------------- | :------------------------------------- |
| **App Directory**       | `/home/gonzalo/immich-app`             |
| **Source Library**      | `/media/gonzalo/Data Main1/library`    |
| **Backup Mount Point**  | `/media/gonzalo/Backup1`               |
| **External Drive UUID** | `ff12a215-2c2f-4649-a390-0d8a23e6ef0e` |
| **Main Log File**       | `/home/gonzalo/immich_backup.log`      |

---

## 🛠 Backup Flows

### 1. Manual Backup (Pre-Update)

Use this procedure before performing a system update (e.g., jumping to **v2.5.2**) to ensure you have a fresh recovery point.

```bash
# Execute the core backup script directly
sudo /home/gonzalo/immich-app/backup_immich.sh
```

**What this script performs:**

- **Mount Verification:** Checks if the 1TB external drive is mounted via its unique UUID.
- **Safety Stop:** Aborts the process if the destination is not a valid mount point to prevent filling up the system's root disk.
- **Service Management:** Stops the `immich-server` container to ensure database consistency during the dump.
- **Database Snapshot:** Creates a compressed PostgreSQL dump named `immich-db-YYYY-MM-DD.sql.gz`.
- **File Sync:** Uses `rsync` to mirror your photo and video library incrementally.
- **Service Restoration:** Restarts the `immich-server` once the sync is finished.

---

### 2. Automated Scheduled Backup (Cron)

This flow is managed by a wrapper script designed to run unattended during low-activity hours.

**Recommended Cron Schedule (Sunday 2:05 AM):**

```bash
# Edit this via: sudo crontab -e
5 2 * * 0 /home/gonzalo/run_full_backup.sh > /home/gonzalo/full_backup.log 2>&1
```

**The Automation Sequence:**

1. **Sync Window:** The script initiates a **20-minute (1200s) sleep** delay to allow mobile devices to finish uploading the day's media.
2. **Worker Execution:** It triggers `backup_immich.sh` to perform the heavy lifting (DB dump and file sync).
3. **Smart User Detection:** The script checks if the user `gonzalo` is currently logged into a graphical desktop session.
4. **Conditional Power Management:**
   - **User Detected:** The system stays powered on for continued use.
   - **No User Detected:** The system executes `shutdown now` to save energy after the backup completes.

---

## 🗄 Backup File Structure

All backup data is stored on the external drive under `/immich-backup/`:

- **`immich-db-YYYY-MM-DD.sql.gz`**: The primary database backup.
- **`files/`**: A mirrored copy of the Immich library folder.
- **`docker-compose.yml`**: A backup of your container orchestration config.
- **`.env`**: A backup of your environment variables, including the specific Immich version number.

---

## ⚠️ Safety & Maintenance

- **Log Monitoring:** You can review the status of the last 50 automated backup events by running:
  `tail -n 50 /home/gonzalo/immich_backup.log`.
- **Root Privileges:** Because the scripts perform `mount` and `shutdown` actions, the automation **must** be configured in the **Root Crontab** to avoid "Permission Denied" errors.
- **Disk Space:** The `rsync` command is non-destructive and only copies new or modified data, significantly reducing the time required for subsequent backups.
