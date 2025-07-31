Environment Variable Sync Tool
==============================

This tool automatically syncs environment variables from `~/.bash_profile` to macOS LaunchAgent, making them available to GUI applications and non-shell processes.

Problem it solves
-----------------

When you set environment variables in `~/.bash_profile`, they're only available in Terminal sessions. GUI applications (like VS Code, IntelliJ, etc.) launched from Finder or Spotlight don't see these variables.

How it works
------------

1.  Captures current environment variables
2.  Sources `~/.bash_profile` to get new environment
3.  Detects changes and new variables
4.  Creates/updates a LaunchAgent plist that sets these variables
5.  Applies changes immediately (full effect after logout/login)

Installation
-----------

1.  **Clone or download the files**
    ```bash
    # Create directory for the tool
    mkdir -p ~/bin
    cd ~/bin
    
    # Download the sync script
    curl -O https://gist.githubusercontent.com/debedb/f8a4b8384cd00b64aa5dfa5f0a961b58/raw/sync-env-to-launchctl
    chmod +x sync-env-to-launchctl
    ```

2.  **Download and customize the LaunchAgent**
    ```bash
    # Download the plist template
    curl -O https://gist.githubusercontent.com/debedb/f8a4b8384cd00b64aa5dfa5f0a961b58/raw/com.user.sync-env
    
    # Edit the plist to replace 'alice' with your username
    sed -i '' "s|/Users/alice|$HOME|g" com.user.sync-env
    
    # Install the LaunchAgent
    cp com.user.sync-env ~/Library/LaunchAgents/
    ```

3.  **Load the LaunchAgent**
    ```bash
    launchctl load ~/Library/LaunchAgents/com.user.sync-env
    ```

4.  **Test it**
    ```bash
    # Add a test variable to your .bash_profile
    echo 'export TEST_VAR="hello world"' >> ~/.bash_profile
    
    # Wait a few seconds for the LaunchAgent to trigger
    sleep 15
    
    # Check if it's available to GUI apps
    launchctl getenv TEST_VAR
    ```

Usage
-----

The tool runs automatically whenever `~/.bash_profile` is modified. You can also run it manually:

```bash
~/bin/sync-env-to-launchctl
```

### What gets synced

The tool syncs:
- All `export` statements from `~/.bash_profile`
- Variables that were added or changed since last run
- Only exports environment variables (not functions or aliases)

### Logs

Check logs if something isn't working:
```bash
# Standard output
tail -f /tmp/sync-env.log

# Errors
tail -f /tmp/sync-env.error.log
```

Features
--------

- **Automatic**: Watches `~/.bash_profile` for changes
- **Incremental**: Only syncs variables that changed
- **Safe**: Preserves existing LaunchAgent environment variables
- **Logged**: Outputs detailed logs for troubleshooting
- **Throttled**: Won't run more than once every 10 seconds

How to verify it's working
-------------------------

1.  **Check LaunchAgent status**:
    ```bash
    launchctl list | grep sync-env
    ```

2.  **Check current GUI environment**:
    ```bash
    launchctl export | grep YOUR_VARIABLE
    ```

3.  **Test with a GUI app**: Launch an app from Finder/Spotlight and check if it sees your variables

Common issues
------------

### Variables not showing up immediately
- LaunchAgent changes apply immediately to new processes
- Existing GUI apps need restart to see new variables
- Some apps may need full logout/login

### Permission errors
```bash
# Make sure script is executable
chmod +x ~/bin/sync-env-to-launchctl

# Check LaunchAgent permissions
ls -la ~/Library/LaunchAgents/com.user.sync-env
```

### LaunchAgent not running
```bash
# Unload and reload
launchctl unload ~/Library/LaunchAgents/com.user.sync-env
launchctl load ~/Library/LaunchAgents/com.user.sync-env

# Check if it's loaded
launchctl list | grep sync-env
```

Uninstallation
-------------

```bash
# Unload LaunchAgent
launchctl unload ~/Library/LaunchAgents/com.user.sync-env

# Remove files
rm ~/Library/LaunchAgents/com.user.sync-env
rm ~/bin/sync-env-to-launchctl
rm /tmp/sync-env.log /tmp/sync-env.error.log

# Remove the generated environment plist (optional)
rm ~/Library/LaunchAgents/com.user.environment.plist
```

Technical Details
----------------

The script creates a LaunchAgent plist (`com.user.environment.plist`) that contains all your environment variables. This plist is loaded into the global LaunchAgent environment, making variables available to all GUI applications.

The sync process:
1. Captures current environment with `printenv`
2. Sources `~/.bash_profile` in a clean environment
3. Compares before/after to find changes
4. Updates the environment LaunchAgent plist
5. Loads the updated plist with `launchctl load`