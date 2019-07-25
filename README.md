# HOVER Engineering

# Development Setup

1. Open `Terminal.app` on your Mac

2. Copy/paste this into `Terminal.app`:

    ```sh
    curl https://raw.githubusercontent.com/hoverinc/engineering/master/script/bootstrap | sh
    ```

    You'll be asked for your:

   - **macOS password** : This is so that the script can use `sudo` to install some things.
   - **Bash / Z shell preference** : If you don't know, just press `return` to use the default.
   - **GitHub account email address** : This is used to create SSH keys for working with GitHub.

***

# TODO:
- [ ] Get AWS access from devops / do AWS MFA setup
- [ ] Detect if the user of script/bootstrap is in the HOVER GitHub org, and if so:
  - [ ] Clone all relevant @hoverinc repos
  - [ ] Run those repos' bootstrap and setup scripts
