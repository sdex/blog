---
layout: post
title: Configure Firejail for Android Studio to run AI agents safely
categories:
  - Android
  - AI
  - Ubuntu
---

After reading yet another article about an AI agent wiping a home directory, I finally decided to protect myself. A quick search led me to [Firejail for VS Code](https://softwareengineeringstandard.com/2025/12/15/ai-agents-firejail-sandbox/). Since I use Ubuntu, this is a perfect match.

![Image]({{ site.baseurl }}/images/firejail_android.png)

The process is quite simple. 

1. Install Firejail

    ```bash
    sudo apt-get update && sudo apt-get install firejail
    ```

2. Create the config directory

    ```bash
    mkdir -p ~/.config/firejail
    ```

3. Create the profile file

    ```bash
    nano ~/.config/firejail/android-studio.profile
    ```

4. Configure the profile
Paste the following into the file. This whitelists only the necessary directories and blocks everything else by default.

    ```bash
    # 1. Allow access to your tools and projects
    whitelist ~/android

    # 2. Allow access to Configs/Build tools
    whitelist ~/.android
    whitelist ~/.gradle
    whitelist ~/.java
    whitelist ~/.jdks
    whitelist ~/.config/Google
    whitelist ~/.cache/Google
    whitelist ~/.local/share/Google
    whitelist ~/.local/share/JetBrains
    whitelist ~/.firebender
    whitelist ~/.gitconfig

    # 3. Essential system bridges
    include /etc/firejail/whitelist-common.inc
    include /etc/firejail/whitelist-runuser-common.inc
    ```

5. Create an alias to run it in safe mode
Add this to your `~/.bashrc` to launch it quickly from the terminal:

    ```bash
    alias studio-safe='nohup firejail --profile=~/.config/firejail/android-studio.profile ~/android/as/android-studio-canary/bin/studio.sh > /dev/null 2>&1 &'
    ```

6. Create a desktop entry 
To launch from your app menu, create `~/.local/share/applications/android-studio-safe.desktop`:

    ```bash
    [Desktop Entry]
    Version=1.0
    Type=Application
    Name=Android Studio (Safe)
    Comment=Isolated Android Studio for AI Safety
    # Use the full path to firejail and the studio script
    Exec=firejail --profile=/home/sdex/.config/firejail/android-studio.profile /home/sdex/android/as/android-studio-canary/bin/studio.sh
    Icon=/home/sdex/android/as/android-studio-canary/bin/studio.png
    Terminal=false
    Categories=Development;IDE;
    StartupNotify=true
    ```

That's it. To verify, ask your AI agent to create a file in your home directory or try to `touch ~/test` from the built-in Android Studio terminal. You’ll find it’s trapped in its own little sandbox, unable to see or touch your sensitive data.
