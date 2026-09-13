# SSH Connection (with GitHub as the Example)

## What SSH Is

- **SSH (Secure Shell):** a protocol for securely connecting one machine to another over an untrusted network, commonly used to log into remote servers, transfer files, or authenticate to services like GitHub.
- **SSH key pair:** two mathematically linked files that replace password authentication:
  - **Public key (safe to share):** you give to a server/service. You can usually register the key via the service's account settings page.
  - **Private key (never shared!):** stays on your machine and proves that you own the linked public key.
- **SSH agent:** a background program that keeps your private key (unlocked) in memory, so when you log into the account where the linked public key is registered, you don't need to type your account password on every connection.
- **Known hosts:** a local record of servers you've already verified, so SSH can warn you if a server's identity suddenly changes.

GitHub is one of many services that support SSH connection. I will use it as the example in the instructions below.

## General Setup Steps

### 1. Check for an existing key
- **Linux/Mac:** `ls -al ~/.ssh`
- **Windows:** `ls ~/.ssh`
- Look for `id_ed25519.pub` or `id_rsa.pub`. Found one? Skip to Step 4.

### 2. Generate a new key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```
- Same command on Linux, Mac, and Windows.
- `-t ed25519` sets the key type (a fast and secure algorithm).
- `-C "..."` attaches a label (usually your email) to help you identify the key later. It has no effect functionally.
- Press Enter to accept the default save location.
- Optionally set a passphrase (an extra password required to unlock the key).

### 3. Start the SSH agent and load your key
- **Linux/Mac:**
  ```bash
  eval "$(ssh-agent -s)"
  ssh-add ~/.ssh/id_ed25519
  ```

  - `ssh-agent -s` starts the agent and prints connection details; `eval "$(...)"` applies those details to your current terminal so it knows where to find the agent.
  - `ssh-add ~/.ssh/id_ed25519` loads your private key into that running agent.

- **Windows (PowerShell as Administrator):**
  ```powershell
  Get-Service ssh-agent | Set-Service -StartupType Manual
  Start-Service ssh-agent
  ssh-add ~/.ssh/id_ed25519
  ```

  - `Get-Service ssh-agent | Set-Service -StartupType Manual` finds the built-in ssh-agent Windows service and sets it to start on demand (it's disabled by default).
  - `Start-Service ssh-agent` starts the agent.
  - `ssh-add ~/.ssh/id_ed25519` loads your private key into the agent.
  - **"Access is denied" error?** PowerShell wasn't run as Administrator. Either relaunch it as admin, or use Git Bash instead (same commands as in Linux). This starts a temporary agent for that Git Bash session only, you'll need to redo it each time you open a new window.

### 4. Copy your public key

- Use the following commands to show the public key:
  - **Linux:** `cat ~/.ssh/id_ed25519.pub` 
  - **Mac:** `pbcopy < ~/.ssh/id_ed25519.pub`
  - **Windows (PowerShell):** `Get-Content ~/.ssh/id_ed25519.pub | clip`
- Select and copy the key (starts with `ssh-ed25519`). Copy the entire line, but make sure you don't get the extra space at the end.

> Only share the `.pub` file. Never share the private key (the file without `.pub`).

### 5. Register the public key with a service
This step differs depending on what you're connecting to:
- **A remote server:** append the key to `~/.ssh/authorized_keys` on that server.
- **GitHub (our example):** go to **GitHub → Settings → SSH and GPG keys → New SSH key**, paste the key, name it, and save.

### 6. Test the connection
- **To a generic server:** `ssh username@server-address`
- **To GitHub:**
  ```bash
  ssh -T git@github.com
  ```
  A successful reply confirms the key works and shows your GitHub username.

## Using SSH with GitHub

- You can clone a repo via SSH with `git clone`. Make sure to copy the correct SSH address from GitHub.
- TO switch an existing repo on your machine from HTTPS to SSH:
  
  ```bash
  git remote set-url origin git@github.com:username/repo.git
  ```
- Run `git remote -v` afterward to confirm the change.

