# macOS Full Disk Access for Touch Notifications

The `keycutter-touch-notify` service requires Full Disk Access on macOS to monitor system logs for YubiKey touch events.

## Symptoms

Without Full Disk Access, you'll see these symptoms:

- No boop sound when YubiKey needs touching
- Service crashes repeatedly (exit code 77)
- Log file shows "YubiKey touch needed!" immediately on startup then restarts
- Running `log stream` manually shows: `log: Must be admin to run 'stream' command`

## Solution

Grant Full Disk Access to the bash binary that runs the script.

### Steps

1. Open **System Settings** (or System Preferences on older macOS)
2. Click **Privacy & Security** in the sidebar
3. Scroll down and click **Full Disk Access**
4. Click the **+** button (you may need to authenticate)
5. Press **Command+Shift+G** to open "Go to Folder"
6. Enter the path to your bash binary:
   ```
   /opt/homebrew/Cellar/bash/5.3.9/bin
   ```
   (Check your version with `ls /opt/homebrew/Cellar/bash/`)
7. Select the `bash` file and click **Open**
8. Ensure the checkbox next to bash is enabled
9. Restart the touch-notify service:
   ```bash
   launchctl unload ~/Library/LaunchAgents/com.keycutter.touch-notify.plist
   launchctl load ~/Library/LaunchAgents/com.keycutter.touch-notify.plist
   ```

### Verification

Test that log stream now works:

```bash
timeout 2 log stream --level debug --style ndjson --predicate 'processImagePath == "/kernel"' 2>&1 | head -3
```

If successful, you'll see JSON log entries instead of "Must be admin" error.

Check the service logs:

```bash
tail -f ~/.local/state/keycutter/touch-notify.log
```

You should see it listening without immediately triggering "YubiKey touch needed!".

## Notes

- The bash binary path may differ if you have a different version installed
- You need to add the actual binary, not a symlink (that's why we use the Cellar path)
- After macOS updates or bash upgrades, you may need to re-add the new binary
