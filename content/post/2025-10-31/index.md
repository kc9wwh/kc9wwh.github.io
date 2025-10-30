---
title: "Running Commands as the Active User on Linux with Fleet"
description: "A modular bash library that solves the root vs. user context challenge for graphical applications and notifications on Linux endpoints."
date: 2025-10-30T23:52:59Z
image: cover.jpg
categories: [Tech]
tags: [Fleet, Linux, Bash, Open Source, Device Management]
hidden: false
comments: true
draft: false
---

# Running Commands as the Active User on Linux with Fleet

If you've ever tried to display a notification or launch a browser from Fleet on a Linux system, you've probably run into a frustrating wall: **Fleet runs as root, but graphical applications need to run in the user's session.**

Running `notify-send` as root does nothing. The notification never appears because it's not connected to the user's display server. Same story with launching Firefox or installing user-specific applications. They need the right environment variables, the right user context, and the right permissions.

After seeing customers wrestle with this problem, I built `fleet_run_as_user.sh`, a modular library that handles the complexity of executing commands in the active user's graphical session.

## The Problem: Root vs. User Context

Fleet executes scripts with root privileges. That's great for system-level tasks like installing packages or modifying system files. But it creates a challenge when you need to:

* **Display notifications** to the logged-in user
* **Launch applications** in the user's GUI environment
* **Install software** in user space (like browser extensions)
* **Run scripts** that require access to user files or settings

The issue boils down to **environment variables and permissions**. A graphical application needs:

* `DISPLAY` or `WAYLAND_DISPLAY` (to know which screen to use)
* `XAUTHORITY` (for X11 authentication)
* `DBUS_SESSION_BUS_ADDRESS` (for D-Bus communication)
* `XDG_RUNTIME_DIR` (for Wayland runtime files)
* The correct **user ID** and **session context**

When you run a command as root, none of these are set correctly. The library solves this by discovering the active session and reconstructing the user's environment.

## How It Works: Three-Step Process

The library follows a simple pattern:

### 1. Find the Active Session

The `find_active_session()` function uses `loginctl` to identify the active graphical user session. It tries multiple strategies:

* First, look for a session with `seat0` (physical display)
* If that fails, find any graphical session
* Fall back to any user session (excluding system sessions)

Once found, it extracts three critical pieces:
* `SESSION_ID` (the session identifier)
* `SESSION_USER` (the username)
* `SESSION_UID` (the user's numeric ID)

### 2. Extract Display Environment

The `get_display_environment()` function reconstructs the user's graphical environment by reading process environment files from `/proc`. It:

* Starts with the session leader process
* Searches other user processes if needed
* Extracts display server variables (`DISPLAY`, `WAYLAND_DISPLAY`)
* Captures authentication tokens (`XAUTHORITY`, `DBUS_SESSION_BUS_ADDRESS`)
* Sets runtime directory paths (`XDG_RUNTIME_DIR`)
* Falls back to sensible defaults if variables aren't found

This approach works for both **X11** and **Wayland** display servers.

### 3. Execute in User Context

With session info and environment variables in hand, the library provides two execution functions:

**`run_as_session_user`** runs commands as the user WITHOUT graphical environment
* Use for: File operations, background tasks, non-GUI commands

**`run_as_graphical_user`** runs commands WITH full graphical environment
* Use for: Notifications, browser launches, GUI applications

## Implementation: Two Ways to Use It

### Option 1: As a Library (Recommended)

Source the script and call functions directly:

```bash
#!/bin/bash
source /path/to/fleet_run_as_user.sh

# Initialize
ensure_root || exit 1
find_active_session || exit 1
get_display_environment || exit 1

# Show a notification
show_notification "Security Update" "Please restart your device" "critical"

# Launch a browser to your internal portal
launch_browser "https://portal.yourcompany.com" "firefox"
```

### Option 2: Modify the Main Function

The script includes a `main()` function with examples. Uncomment and customize:

```bash
main() {
    ensure_root || exit 1
    find_active_session || exit 1
    get_display_environment || exit 1

    # Your custom action here
    run_as_graphical_user "notify-send 'Fleet' 'Compliance check complete'"
}
```

Then execute the script directly from Fleet.

## Real-World Use Cases

**Compliance Notifications**

Notify users when their device falls out of compliance:
```bash
show_notification "Action Required" "Your system is missing security updates. Please update within 24 hours." "critical"
```

**Self-Service Portal**

Launch a browser to your internal support portal:
```bash
launch_browser "https://support.company.com/ticket/12345" "google-chrome"
```

**User-Space Application Installation**

Install browser extensions or user-specific tools:
```bash
run_as_session_user "pip install --user your-package"
```

**Pre-Restart Warnings**

Give users a heads-up before forced reboots:
```bash
show_notification "System Maintenance" "Your device will restart in 5 minutes. Please save your work." "critical"
sleep 300
reboot
```

## Why It Works: The Technical Details

**loginctl vs. who**

The library uses `loginctl` instead of parsing `who` or `w` because it provides structured session data including session type (graphical vs. non-graphical) and seat information.

**Process Environment Discovery**

Reading `/proc/<pid>/environ` gives us the actual environment variables from running processes. The library searches multiple processes because not all user processes inherit all variables.

**Fallback Defaults**

If environment variables aren't found, the library uses reasonable defaults (`:0` for DISPLAY, `wayland-0` for Wayland). This handles edge cases where variables aren't set consistently.

**X11 and Wayland Support**

By checking for both `DISPLAY` and `WAYLAND_DISPLAY`, the library works regardless of which display server the distribution uses.

## Troubleshooting

Enable debug mode to see what's happening:
```bash
DEBUG=1 ./fleet_run_as_user.sh
```

Common issues:

* **"Could not find an active user session"** - No user is logged in graphically
* **"Display environment not set"** - Call `get_display_environment` before `run_as_graphical_user`
* **"This script must be run as root"** - The script needs root privileges to switch users

## Best Practices

**Always check return codes**
```bash
if ! find_active_session; then
    error_log "No active session found"
    exit 1
fi
```

**Use the right function**
* `run_as_session_user` for file operations and background tasks
* `run_as_graphical_user` for anything that needs a display

**Test with DEBUG=1**

Before deploying to production, test with debug logging enabled to verify the correct session and environment are detected.

**Handle no-user scenarios**

Not all Linux systems have an active graphical user. Add fallback logic for headless servers or systems where no one is logged in.

## Getting Started

1. **Save the library** to your Fleet scripts repository
2. **Test locally** with `DEBUG=1` enabled
3. **Create a Fleet script** that sources the library
4. **Deploy to a test group** before rolling out broadly

The library is modular, well-documented, and handles the edge cases that make this problem tricky. Whether you're pushing compliance notifications, launching self-service portals, or installing user-specific software, it gives you the building blocks to execute commands in the user's context reliably.

---

**Questions?** Drop them in the comments or reach out to the Fleet community on Slack. If you've built something cool with this library, I'd love to hear about it.

**Contributing?** The library is designed to be extended. Have ideas for additional convenience functions? Submit a PR or share your use case.

Catch ya on the next project!

Josh 🖖

> Photo by [Gabriel Heinzer](https://unsplash.com/@6heinz3r?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com)
