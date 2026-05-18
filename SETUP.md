# MSLI Integrated Management System — Deployment Guide

This guide is for IT / administrators setting up MSLI at a new site. The deployment has two halves:

1. **Server side** — one machine on the LAN runs MariaDB and holds the shared database. Done once per site.
2. **Client side** — every workstation that will use MSLI installs the client app and points it at the server. Done once per workstation.

> **Audience:** IT installers / administrators.
> **For day-to-day app usage**, see the in-app **Help → User Guide**.

---

## 1. Server Setup (one-time)

Pick **one** machine to host the database. Every client workstation on the network connects to it. Requirements:

- Windows 10 / 11 (or Windows Server).
- Always-on while users are working.
- Internet access **during install only** — used to download MariaDB.

The easiest path uses the one-click **MSLI Server Setup** installer. There's also a manual path if you'd rather drive MariaDB yourself.

### 1A. Recommended — one-click installer

1. On the chosen server machine, download **`MSLI-Server-Setup.exe`** from the latest release: <https://github.com/ryanclaret/MSLI-Releases/releases/latest>.
2. Double-click `MSLI-Server-Setup.exe`. Walk through the wizard:
   - **Welcome** → Next.
   - **MariaDB root password** — choose a strong one and **write it down**.
   - **Install location** → Install.
3. Wait ~3–5 minutes for MariaDB to download, install, and bootstrap the database.
4. On the final screen, leave **"Show this server's IP address"** ticked → Finish.
5. A black window appears listing one or more IPs. **Write down** the one starting with `192.168.` or `10.` — every client workstation will type this into Connection Settings.

The installer has done all of the following for you:
- Installed MariaDB and started the Windows service.
- Created the `msli_schema` database.
- Created the application user `Admin` (password `MSLI_user_0001`) that the client app uses.
- Created the starter administrator account `admin` (password `admin123`) for first login.
- Opened TCP 3306 on the Windows Firewall.

Skip to **Step 1.3 (Reserve the server's IP)** below.

### 1B. Manual server install (advanced — skip if you used 1A)

Use this path only if you need full control or you're running on Linux / a cloud VPS where the installer doesn't apply.

1. **Install MariaDB 10.6+** or MySQL 8.0+. Configure it to listen on TCP `0.0.0.0:3306` and open the firewall.
2. Connect as `root` and run:
   ```sql
   CREATE DATABASE msli_schema
       CHARACTER SET utf8mb4
       COLLATE utf8mb4_unicode_ci;

   CREATE USER 'Admin'@'%' IDENTIFIED BY 'MSLI_user_0001';
   GRANT ALL PRIVILEGES ON msli_schema.* TO 'Admin'@'%';
   FLUSH PRIVILEGES;

   USE msli_schema;
   INSERT IGNORE INTO users (username, password, role)
   VALUES ('admin', 'admin123', 'Administrator');
   ```
3. **You do not need to run any migration `.sql` scripts.** The client app's built-in `SchemaUpdater` creates every table and applies every schema change automatically on first launch (and on every later launch — it's idempotent).

> ⚠️ The default credentials `Admin / MSLI_user_0001` are baked into the client's `App.config`. If you change them on the server, the client app needs a rebuild to match. Coordinate with whoever maintains the build.

### 1.3 Reserve the server's IP in your router (DHCP reservation)

**Don't skip this step.** By default the router can hand the server a different IP after a reboot or power outage, which silently breaks every client.

1. Log into the office router's admin page (usually `http://192.168.1.1` or `http://192.168.0.1` — check the sticker on the router).
2. Find the section named **DHCP Reservation**, **Address Reservation**, **Static DHCP**, or **LAN → DHCP Server** (varies by brand).
3. Add a new reservation:
   - **IP address**: the one you wrote down in step 1A.5 (or read with `ipconfig` if you went the manual route).
   - **MAC address**: run `ipconfig /all` on the server, find "Physical Address" under the network adapter you're using.
   - **Description / Name**: `MSLI Server`.
4. Save / Apply. Some routers need a reboot.
5. Verify by rebooting the server and running `ipconfig` again — the IPv4 address should match what you reserved.

---

## 2. Client Setup (per workstation)

### 2.1 Install

1. On the workstation, download **`MSLI-Setup.exe`** from <https://github.com/ryanclaret/MSLI-Releases/releases/latest>.
2. Double-click `MSLI-Setup.exe`. It installs to `%LocalAppData%\MSLI\` — **no admin rights required**.
3. The app launches automatically when install finishes.

### 2.2 First-run connection setup

The first launch can't connect to the database (because the server IP isn't configured yet), so the **Connection Settings** dialog opens.

1. **Server**: the LAN IP you wrote down in step 1A.5 (e.g. `192.168.1.50`).
2. **Port**: leave as `3306` unless your server is using a different one.
3. Click **Test Connection** — should report success.
4. Click **Save**.

The setting is stored locally; later launches connect automatically.

### 2.3 First login

Use the starter administrator account:

- **Username**: `admin`
- **Password**: `admin123`

The first time `admin` logs in, the app upgrades the plaintext password to a BCrypt hash automatically. **Change the password immediately** via **User → Change Password**.

Then use **User → Manage User** to create real accounts for everyone else and **User → Manage Role** to assign roles. After that, you can either disable or rename the `admin` account.

### 2.4 Updates

There's nothing to do on the workstation. Every time MSLI launches, it checks the public `MSLI-Releases` repo for a newer version, downloads the delta in the background, and applies it on the next launch. You'll see the bumped version number on the splash screen.

### 2.5 Schema changes ship with the client

You will **not** be asked to run `ALTER TABLE` statements or migration scripts when a new MSLI version arrives. The client's `SchemaUpdater` adds new columns / tables / indexes on first launch after the update. If a new release has notes that say "manual data adjustment required" (e.g. assign a new role), that part is handled in the app's admin screens — not by editing SQL.

---

## 3. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| **"Cannot connect to server"** on first run | Wrong IP in Connection Settings; server firewall blocking TCP 3306; or MariaDB bound to `127.0.0.1` only. On the server, edit `my.ini` (or `my.cnf`) and confirm `bind-address=0.0.0.0`, then restart the MariaDB service. |
| **"Access denied for user 'Admin'@…"** | The `Admin` user was created with `'@localhost'` instead of `'@%'`. Re-run the `CREATE USER` line from section 1B. |
| **Login screen shows "account locked"** | Five+ failed attempts in 15 minutes. Wait 15 minutes, or as DB admin clear `failed_attempts` / `lockout_until` on the row in `users`. |
| **Forgot the admin password** | As DB admin, set `users.password` for `admin` back to plaintext (e.g. `'admin123'`) — the app re-hashes it on next login. |
| **Got logged out unexpectedly** | Someone else signed in with the same account. MSLI enforces one active session per account — the newer login wins. Use distinct accounts per person. |
| **App doesn't auto-update** | Check `error.log` in `%LocalAppData%\MSLI\app-<version>\` for Squirrel exceptions. The public `MSLI-Releases` repo must be reachable from the workstation over HTTPS — corporate proxies that block GitHub will prevent updates. |
| **The server's IP changed after a reboot** | DHCP reservation isn't set. Go back to step 1.3 and pin the IP to the server's MAC. Until then, every client's Connection Settings must be re-pointed at the new IP. |
| **A new release says "manual data adjustment required"** | Read the GitHub release notes on `MSLI-Releases`. Typical adjustments: assigning a new role to existing users, or filling in a new template slot. Use the in-app admin screens (User → Manage Role / User → Manage User / Forms → Template). |
