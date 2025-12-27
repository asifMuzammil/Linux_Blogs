![Alt text](logrotate.png)
# Logrotate: The Right Way to Manage Logs on Linux Servers

When you manage a Linux server, log files quietly grow in the background.
Web servers, system services, custom scripts, containers—everything writes logs. If they are ignored, one day your server will suddenly fail with a **“No space left on device”** error.

This is one of the most common and preventable causes of production outages.

Many beginners try to solve this by deleting logs manually using commands like:

```bash
rm -rf *.log
```

This approach is risky and often ineffective. If an application is still writing to a log file, deleting it does **not** immediately free disk space. The space is only released after the application restarts, which can cause confusion and downtime.

There is a safer, standard, and professional solution: **Logrotate**.

---

## The Common Misconception: “Linux Handles Logs Automatically”

This belief is only partially true.

Linux rotates **system logs** (such as `syslog`, `auth.log`, and package manager logs) by default. However, Linux has no awareness of your **custom applications**, web apps, background workers, or container logs.

If you do not explicitly configure Logrotate for your own services, those logs will continue to grow indefinitely until the disk is full.

---

## What Is Logrotate?

Logrotate is a system utility that automatically manages log files based on rules you define.
It can:

* Rotate log files
* Compress old logs
* Delete outdated logs
* Run commands before or after rotation

It works silently in the background and prevents uncontrolled log growth.

---

## How Logrotate Is Structured

### 1. Global Configuration

**File:** `/etc/logrotate.conf`

This file contains global defaults such as rotation frequency and retention policy. It mainly applies to system logs.

You can view it using:

```bash
cat /etc/logrotate.conf
```

---

### 2. Application-Specific Configurations

**Directory:** `/etc/logrotate.d/`

This is where **your custom configurations belong**.

Each file inside this directory represents **one application or service**. Logrotate automatically reads every file placed here.

For example:

* `/etc/logrotate.d/nginx`
* `/etc/logrotate.d/mysql`
* `/etc/logrotate.d/my-web-app`

You should **never** edit system files directly. Always create a new file for your application in this directory.

---

## A Real-World Logrotate Configuration

Below is a practical example for a web application using Nginx.

**File:** `/etc/logrotate.d/my-web-app`

```conf
/var/log/my-web-app/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts

    # Store old logs in a dedicated archive directory
    olddir /var/log/my-web-app/archives

    prerotate
        # Optional commands before rotation
    endscript

    postrotate
        # Reload Nginx so it reopens log files
        invoke-rc.d nginx rotate >/dev/null 2>&1
    endscript
}
```

This configuration tells Logrotate exactly **which logs to manage** and **how to handle them**.

---

## Explanation of Key Options

* **daily**
  Logs are checked and rotated every day.

* **rotate 14**
  Keeps logs for 14 days and deletes older ones.

* **compress**
  Compresses rotated logs to save disk space.

* **delaycompress**
  Prevents compressing the most recent rotated file to avoid conflicts.

* **olddir /var/log/my-web-app/archives**
  Moves old logs into a separate archive directory to keep the main log folder clean.

**Important:**
The archive directory must already exist and must be on the same filesystem.

---

## Quick Tip: Test Your Configuration Safely (Dry Run)

Before trusting a new Logrotate configuration in production, you can test it without rotating any files:

```bash
logrotate -d /etc/logrotate.conf
```

The `-d` (debug) flag performs a **dry run**. It shows exactly what Logrotate *would* do—without modifying or deleting any logs. This is the safest way to verify your configuration before deployment.

---

### Optional (slightly more advanced, still simple)

If you want to force rotation during testing:

```bash
logrotate -f /etc/logrotate.conf
```

Use `-f` carefully. It forces rotation even if conditions (like `daily`) are not met.

---

## Advanced Use Case: Upload Rotated Logs to S3

In some environments, logs must be stored long-term for auditing or compliance. Instead of keeping everything on the server, you can upload rotated logs to cloud storage.

This is done using the **`lastaction`** block inside the **same Logrotate config file**.

Add the following inside `/etc/logrotate.d/my-web-app`:

```conf
lastaction
    aws s3 sync /var/log/my-web-app/archives/ s3://my-company-logs/web-app/
endscript
```

### How This Works

* `lastaction` runs **after rotation and compression are fully complete**
* Ensures only finalized `.gz` files are uploaded
* Prevents uploading logs that are still being written

---


## Who Runs Logrotate? The Role of `/etc/cron.daily`

Logrotate is **not a daemon** and does not run continuously.

Instead, it is triggered by a cron job located at:

```
/etc/cron.daily/logrotate
```

This script runs **once per day** and does the following:

1. Calls the Logrotate binary
2. Reads `/etc/logrotate.conf`
3. Reads every file inside `/etc/logrotate.d/`
4. Applies rotation rules if conditions are met

You normally **do not need to modify** anything inside `/etc/cron.daily`.
As long as the file exists and is executable, Logrotate will run automatically.

On systemd-based systems, you can verify this using:

```bash
systemctl status logrotate.timer
```

---

## Final Thoughts

Effective automation is not about writing complex scripts. It is about using proven tools in the correct way.

Whenever you deploy a new application, adding a small Logrotate configuration should be part of your standard setup. A simple file in `/etc/logrotate.d/` can protect your server from unexpected outages and emergency disk cleanups.

---
*This guide is maintained by the community for the community. Feel free to contribute, share, and adapt as needed.*

## Feel free to reach out!

### Here are some ways to connect with me:

###  Social Media:

- [LinkedIn](https://www.linkedin.com/in/asif-muzammil-hussain-b6742441/)
- [GitHub](https://github.com/asifMuzammil/github-actions-docker-ghcr)
- [Personal Email](m.asif.muzammil@gmail.com)
- [Medium](https://medium.com/@m.asif.muzammil)
- [Kaggle](https://www.kaggle.com/asifmuzammil)

#DevOps #Linux #SysAdmin #Logrotate #Automation #TechTips

---
