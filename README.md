# XAMPP MySQL: Fixing Startup Errors

A guide to recovering a XAMPP MySQL that won't start without losing databases.

> **Warning:** Always back up `C:\xampp\mysql\data` before doing anything.

---

## Step 0: Back up first

Copy the entire `C:\xampp\mysql\data` folder to a safe place (e.g., Desktop). It holds all your databases.

## Step 1: Check the real error

In XAMPP, click **Logs** next to MySQL, or open:

```
C:\xampp\mysql\data\mysql_error.log
```

Read the last lines. They tell you which fix below applies.

## Fix A: Port 3306 already in use (most common)

Another MySQL service is running.

1. Press `Win + R`, type `services.msc`.
2. Stop any "MySQL" service.
3. Start MySQL again from XAMPP.

Alternatively, click **Netstat** in XAMPP to see what is using port 3306.

## Fix B: Corrupted tables / InnoDB crash

1. Open `C:\xampp\mysql\bin\my.ini`.
2. Under `[mysqld]`, add:
   ```ini
   innodb_force_recovery = 1
   ```
3. Save and start MySQL.
4. If it starts, **immediately export your databases** via phpMyAdmin or `mysqldump`.
5. If it still fails, raise the value to 2, 3, up to 4 (do not go higher unless desperate).
6. After exporting, remove that line.

## Fix C: Replace system files from backup (keeps your data)

The official XAMPP repair method:

1. Rename `C:\xampp\mysql\data` to `data_old`.
2. Copy `C:\xampp\mysql\backup` to a new folder named `data`.
3. From `data_old`, copy **your database folders** into the new `data`. Skip these system folders: `mysql`, `performance_schema`, `phpmyadmin`, `sys`.
4. Copy `ibdata1` from `data_old` into the new `data` (overwrite).
5. Start MySQL.

> Do **not** copy `ib_logfile0` and `ib_logfile1`. Let MySQL recreate them.

## Fix D: Stale pid / lock files

Delete these from the `data` folder if present, then restart:

- `mysql.pid`
- `aria_log_control`

---

## Quick Reference

| Symptom in log | Likely cause | Go to |
|----------------|--------------|-------|
| Port 3306 / "address already in use" | Another MySQL running | Fix A |
| InnoDB errors, "crash recovery" | Corrupt tables or logs | Fix B / C |
| Missing/corrupt `mysql` system tables | Damaged system folder | Fix C |
| "Can't create/write to file" pid | Stale pid file | Fix D |
| Unknown variable / invalid value | Bad `my.ini` | Edit `my.ini` |

If stuck, paste the last ~10 lines of `mysql_error.log` to identify the exact fix.
