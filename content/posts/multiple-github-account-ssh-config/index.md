---
title: "Multiple Github Account Ssh Config"
date: 2026-03-24T00:13:36+08:00
draft: false
tags: ["git", "ssh", "github"]
categories: ["dev"]
series: []
description: "Memo of how to setup multiple github account ssh config"
cover:
  image: ""
  alt: ""
ShowToc: true
TocOpen: false
---

# How to setup multiple github account ssh config

If you use both a personal and a work GitHub account, you’ve likely run into issues like commits using the wrong email or pushes failing due to incorrect SSH keys.

## The Problem

GitHub associates each SSH key with a single account. That means:

- One key = one account
- Multiple accounts = multiple SSH keys

You also need to ensure commits use the correct user.name and user.email based on the project.

## Overview

Here's what we are going to setup:

1. Generate an SSH Key per Github account.
1. Add each public key to the corresponding Github account.
1. Configure `~/.ssh/config` file with host aliases.
1. Configure git to auto-switch identity based on repository directory

Below are two example accounts:

|              | Personal                     | Work                     |
| ------------ | ---------------------------- | ------------------------ |
| GitHub user  | `Simon`                      | `Simon-work`             |
| Email        | `simon@personal.dev`         | `simone@work.com`        |
| SSH key      | `~/.ssh/id_ed25519_personal` | `~/.ssh/id_ed25519_work` |
| Projects dir | `~/personal/`                | `~/work/`                |

### Step 1: Generate SSH Keys

```bash
# Personal account
ssh-keygen -t ed25519 -C "simon@personal.dev" -f ~/.ssh/id_ed25519_personal

# Work account
ssh-keygen -t ed25519 -C "simon@work.com" -f ~/.ssh/id_ed25519_work
```

After this the output structure would like:

```bash
~/.ssh/
├── id_ed25519_personal       # private key (personal)
├── id_ed25519_personal.pub   # public key (personal)
├── id_ed25519_work            # private key (work)
└── id_ed25519_work.pub        # public key (work)
```

### Step 2: Add Keys to the SSH Agent

```bash
# Start the agent
eval "$(ssh-agent -s)"

# Add both keys
ssh-add ~/.ssh/id_ed25519_personal
ssh-add ~/.ssh/id_ed25519_work

# Verify they're loaded (optional)
ssh-add -l
```

### Step 3: Add public key to Github

```bash
# Copy personal public key
cat ~/.ssh/id_ed25519_personal.pub
# → Copy the output

# Copy work public key
cat ~/.ssh/id_ed25519_work.pub
# → Copy the output
```

### Step 4: Configure SSH config

The core of the whole setup. Edit `~/.ssh/config`:

```bash
# Personal
Host gh-personal # This is the alias for personal account's github domain
  HostName github.com
  User git # Github use git as user to authenticate into github using ssh (git@domain.com)
  IdentiyFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes

# Work
Host gh-work # This is the alias for work account's github domain
  HostName github.com
  User git
  IdentiyFile ~/.ssh/id_ed25519_work
  IdentitiesOnly yes
```

- `Host <aliases>` - create a sematic alia for specific account.
- `IdentitiesOnly yes` - tell ssh to only use the specified key and do not try other key.

### Step 5: Verify the setup

```bash
# personal
ssh -T gh-personal

# work
ssh -T gh-work
```

### Step 6: Setup remote url

For existing repos, update the remote URL:

```bash
# personal
cd ~/personal/repo_personal
git remote set-url origin git@gh-personal:simon/repo_personal.git

# work
cd ~/work/repo_work
git remote set-url origin git@gh-work:simon-work/repo_work.git

# after setup url, verify with git command
git remote -v
```

## References

- [GitHub Docs: Managing multiple accounts](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-your-personal-account/managing-multiple-accounts) — GitHub's official guide on handling multiple accounts with SSH and HTTPS.
- [GitHub Docs: Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) — Official walkthrough for generating keys and adding them to your account.
- [GitHub Docs: Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) — Step-by-step for uploading your public key.
- [Git Documentation: git-config Conditional Includes](https://git-scm.com/docs/git-config#_conditional_includes) — Official docs on `includeIf` with `gitdir`, `gitdir/i`, `onbranch`, and `hasconfig`.
- [OpenSSH Manual: ssh_config](https://man.openbsd.org/ssh_config) — Full reference for all SSH config options (`IdentityFile`, `IdentitiesOnly`, `Host`, etc.).
- [Arch Wiki: SSH keys](https://wiki.archlinux.org/title/SSH_keys) — Arch-specific guide covering key generation, ssh-agent, and systemd integration.
