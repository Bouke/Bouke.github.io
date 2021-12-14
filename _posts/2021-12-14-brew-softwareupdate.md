---
layout: post
title: Careful what you wish for
---

Be careful what you wish for. I tried to install some tooling on my machine using Homebrew. It required an update to the Command Lne Tools. Homebrew helpfully included a command which I could run to update the Command Line Tools:

```
> brew install mongodb-database-tools
mongodb-database-tools 100.3.0 is already installed but outdated (so it will be upgraded).
==> Downloading https://fastdl.mongodb.org/tools/db/mongodb-database-tools-macos-x86_64-100.5.1.zip
######################################################################## 100.0%
==> Upgrading mongodb/brew/mongodb-database-tools
  100.3.0 -> 100.5.1

Error: Your Command Line Tools are too outdated.
Update them from Software Update in System Preferences or run:
  softwareupdate --all --install --force

If that doesn't show you any updates, run:
  sudo rm -rf /Library/Developer/CommandLineTools
  sudo xcode-select --install

Alternatively, manually download them from:
  https://developer.apple.com/download/all/.
You should download the Command Line Tools for Xcode 13.1.
```

Little did I know that this would trigger an OS upgrade (Monterey 12.1) causing down time for half an hour. Also note that ^C didn't stop the update and the UI in System Preferences also had no cancel button.
```
> softwareupdate --all --install --force
Software Update Tool

Finding available software
Downloading macOS Monterey 12.1
Downloading: 98.56%^C
Session Contents Restored on 14 Dec 2021 at 11:51
```
