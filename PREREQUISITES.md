# Running `websocat` Unprivileged in LlamaLab Automate (Android 14)

On Android 10–14+, strict SELinux (`W^X`) and Scoped Storage policies prevent standard apps (`untrusted_app`) from directly executing binaries (`execve`) inside writable directories (`/data/data/` or `/sdcard/`), and block reading files placed in `/sdcard/Download/` by `adb` (`shell` UID).

To run `websocat` inside Automate’s standard **Shell command** block (without root or the Privileged Service), stage the binary inside Automate’s private app storage and load it through Android’s system dynamic linker.

---

## Prerequisites

* **Dynamically Linked `websocat` Binary:** Must match your device's CPU architecture (e.g., `arm-linux-androideabi` for 32-bit `armv8l`/`armeabi-v7a`, or `aarch64-linux-android` for 64-bit `aarch64`). *Note: Static `musl` builds will not work with `/system/bin/linker`.*
* **ADB Access** from a computer (used once during initial setup).

---

## Step 1: Push `websocat` to Shared Storage

From your computer, push the binary directly to `/sdcard/Download/` so it is accessible on the device:

```bash
adb push websocat /sdcard/Download/websocat
```

> **Note:** Push the file directly via `adb push` rather than copying it from `/data/local/tmp/` via `adb shell`, which tags the file with the `shell` UID and triggers Android 14 Scoped Storage blocks.

---

## Step 2: Import into Automate’s Private Directory

Because unprivileged apps cannot execute files from `/sdcard/`, copy `websocat` into Automate’s private `/files/` directory using Automate itself so Automate owns the file (`u0_aXXX`):

1. Open **LlamaLab Automate** and create a temporary flow (or add a one-time block).
2. Add a standard **Shell command** block (not *Privileged*).
3. Set the **Command line** to:
   ```sh
   sh -c 'mkdir -p /data/data/com.llamalab.automate/files && cp /sdcard/Download/websocat /data/data/com.llamalab.automate/files/websocat'
   ```
4. Run the flow once.

> **Note:** `chmod +x` is not required—`/system/bin/linker` only needs read (`-rw-------`) permission.

---

## Step 3: Verify Execution via System Linker

Instead of executing `websocat` directly, invoke it through `/system/bin/linker` (use `/system/bin/linker64` on 64-bit Android builds):

1. In a standard **Shell command** block, run:
   ```sh
   /system/bin/linker /data/data/com.llamalab.automate/files/websocat --version
   ```
2. Confirm that **Standard output** returns the `websocat` version string with **Exit code `0`**.

---

## Step 4: Clean Up Temporary Files

Once verified, delete the staging copy from shared storage via ADB:

```bash
adb shell rm -f /sdcard/Download/websocat
```

---

## Production Usage Template (With JSON Stdin)

Because Automate’s standard **Shell command** block executes commands directly without spawning a shell, always wrap pipes (`|`), redirects (`<`), or command chaining (`&&`) inside `sh -c '...'`:

```sh
sh -c '{ cat /data/data/com.llamalab.automate/files/req.json; echo; } | /system/bin/linker /data/data/com.llamalab.automate/files/websocat -n1 -k -H "X-Sonos-Api-Key: <YOUR_KEY>" wss://<SPEAKER_IP>:1443/websocket/api'
```

* **`{ cat ...; echo; }`:** Ensures a trailing newline (`\n`) is sent so the Sonos JSON-RPC parser recognizes the end of the frame.
* **`-n1`:** Forces 
