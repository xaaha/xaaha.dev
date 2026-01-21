---
title: Connect Multiple GitHub Accounts to your Machine
date: 2026-01-21
description: How to connect multiple GitHub Accounts in your machine
draft: false
category: GitHub CI/CD
---

# Connect to Multiple Github Accounts With SSH from One Computer

To connect and authorize multiple GitHub accounts from the same computer, you can set up SSH configurations for each account. By doing so, you can associate different SSH keys with different GitHub accounts, making it easy to push or pull code from multiple repositories without conflicts. Here’s how to manage multiple GitHub accounts using SSH on the same machine:

Here is the tldr for connecting into github for one account

```bash
ssh-keygen -t ed25519 -C "ryan.ferreira@guildeducation.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

## Steps

I am using mac but the process is mostly similar between Linux and Mac. If you are using Windows, Please visit [github's steps](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) to generate ssh keys if you use another system.

### Generate a New SSH Key for Each GitHub Accounts

For each GitHub account, generate a new SSH key. You’ll have different keys associated with each account.

For example, to generate an SSH key for your work GitHub account:

```bash
ssh-keygen -t ed25519 -C "your_work_email@example.com"
```

When prompted, save the key with a unique filename (for example, id_ed25519_work):

```bash
# Enter file in which to save the key
(/Users/yourusername/.ssh/id_ed25519): /Users/yourusername/.ssh/id_ed25519_work
```

> Repeat the process for your personal GitHub account, using a different key and filename:

```bash
ssh-keygen -t ed25519 -C "your_personal_email@example.com"
```

Save this key with a unique filename as well `(e.g., id_ed25519_personal):`

```bash
# Enter file in which to save the key
(/Users/yourusername/.ssh/id_ed25519): /Users/yourusername/.ssh/id_ed25519_personal

```

### Add the SSH Keys to the SSH Agent

After generating the SSH keys, you need to add them to your SSH agent, which manages your keys. This way you don't have to enter the key every time it's required.

First, ensure your SSH agent is running:

```bash
eval "$(ssh-agent -s)"
```

Then, add each key to the agent:

```bash
# For the work account:
ssh-add ~/.ssh/id_ed25519_work
# For the personal account:
ssh-add ~/.ssh/id_ed25519_personal
```

If you use passphrase for any key you generated above, and you use mac, run

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_personal
## OR
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_work

```

It will ask to enter your respective passphrase, and keychain will save it for you. Now, you don't have to enter it again. If you don't enter this, you might have to enter the passphrase on every pull or push.

### Add SSH Keys to GitHub

Go to GitHub for each account and add the corresponding public SSH key to the account.

For the work account:

• Copy the SSH public key:

```bash
cat ~/.ssh/id_ed25519_work.pub | pbcopy
cat ~/.ssh/id_ed25519_personal.pub | pbcopy
```

• Go to your work and personal GitHub account settings:
• Settings > SSH and GPG Keys > New SSH Key.
• Paste the public key, giving it a descriptive name (e.g., “Work Key” and "Personal Key").

### Configure SSH to Handle Multiple Accounts

Now, you need to configure SSH to handle multiple GitHub accounts by modifying your SSH configuration file (~/.ssh/config). This will let SSH know which key to use for each GitHub account.

Edit (or create) the SSH configuration file: `nvim ~/.ssh/config`

Add the following entries to specify different hosts for each account:

```config
# Personal GitHub account
Host github-personal # this could just be personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# Work GitHub account
Host github-work # or just work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

Here:

• Host is an alias for the remote server, so you can use github-personal and github-work to differentiate between your personal and work accounts.
• IdentityFile specifies which private SSH key to use for that account.

### Clone Repositories Using Custom Hostnames

Now, you can clone repositories by using the custom hostnames: github-personal or github-work as defined in your ~/.ssh/config.

For the personal account use: `git clone git@github-personal:username/repo.git`

For the work account: `git clone git@github-work:username/repo.git`

### Pushing and Pulling from Multiple Accounts

After you’ve cloned the repositories using the correct custom hostname, Git will automatically use the correct SSH key for each account when you push or pull. You don’t need to do anything extra once it’s set up.

### Optional But Recommended: Configuring Git User Information Per Repository

You may also want to set different Git user information (name and email) for each account. This tells GitHub, which account pushed to the repo. You can do this on a per-repository basis by navigating to the repository and setting the user details:

```bash
cd path/to/repo

# Set user information for personal repo
git config user.name "Personal Name"
git config user.email "personal_email@example.com" ## or just xaaha
```

For the work repo:

```bash
cd path/to/work/repo
# Set user information for work repo
git config user.name "Work Name"
git config user.email "work_email@example.com"
```

This ensures that commits made in each repository will use the appropriate email address for the respective GitHub account.

### For your old repos on your computer...

If you have repos on your computer cloned before the change, you might get an error that you don't have repository access.

```bash
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

To fix it, change the remote repository url of the repo that's having issues.

```bash
git remote set-url origin git@xaaha:username/repository.git # or git@work_email
```

Verify the change with

```bash
git remote -v
```

and finally git pull should work

```bash
git pull
```

## Summary of the Setup:

1. Generate separate SSH keys for each account.
2. Add the SSH keys to GitHub for the respective accounts.
3. Set up an SSH config file to manage multiple identities with different SSH keys.
4. Clone repositories using custom hostnames to differentiate accounts.
5. Optionally, set Git user information for each repository.

This setup will allow you to seamlessly manage and authorize multiple GitHub accounts from the same machine without conflicts.
