# First Day Setup Guide

## Setup on standard-issue Windows 10 desktop

### Item 1: Optional: Make new user for desktop login

More info: <https://support.microsoft.com/en-us/windows/add-or-remove-accounts-on-your-pc-104dc19f-6430-4b49-6a2b-e4dbd1dcdf32#WindowsVersion=Windows_10>

### Item 2: Install VS Code

Download VS Code from website and install.

Make sure you selected Add to PATH during the installation.

### Item 3: Install Discord

Download Discord from website and install.

### Item 4: Install Postman

Download Postman from website and install.

### Item 5: Install Node

Download Node from website.

### Item 6: Install Git

More info: <https://git-scm.com/download/win>

Use winget CLI tool.

Open terminal/CMD and run the following:

```bash
winget install --id Git.Git -e --source winget
```

Need to add following to PATH variables to get git to work on terminal:

```bash
C:\Program Files\Git\bin\
C:\Program Files\Git\cmd\
```

More info: <https://linuxhint.com/add-git-to-path-windows/>

Continue set up git as per documentation: <https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup>

Suggest to set VS Code as Git editor:

```bash
git config --global core.editor "code --wait"
```

More info: <https://code.visualstudio.com/docs/sourcecontrol/overview#_vs-code-as-git-editor>

### Item 7: Generate SSH key pair for gitlab and config

At home holder (usually `C:\Users\[PC USERNAME]`, commonly shortened to just `~`)

```bash
cd ~
```

Run following command:

```bash
ssh-keygen -t rsa -b 4096 -C your_email@email.com
```

**REMEMBER TO MODIFY ABOVE COMMAND WITH YOUR OWN EMAIL ADDRESS.**

SSH key pair with name `id_rsa` and `id_rsa.pub` should appear in `~/.ssh`

At `~/.ssh` create file with name `config` (without file extension) with following content:

```config
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile C:\Users\[PC USERNAME]\.ssh\id_rsa
    IdentitiesOnly yes
    PubkeyAcceptedAlgorithms +ssh-rsa
    HostkeyAlgorithms +ssh-rsa
```

Modify the above content of config file according to the SSH key directory in your workstation computer setup.

Try: Git clone repo.

Issue: Cloning repo directly from link in gitlab repo page **WILL NOT** work due to missing port number.
Git clone command needs to include port number to work. Quirk due to self-hosted nature of self-hosted Gitlab instance.

### Item 8: Make OCBC Malaysia account

Download mobile app and register. Referral code: **XXXX**

May take more than 12 hours to complete processing at OCBC side.

Actually using the activated OCBC account will take another 12 hours due to device safety policy on 2FA activation (OneToken).

### Item 9: Set up thunderbird

Initial email account set up by IT dept may only complete after lunch on first day of duty.

Set up Thunderbird (pre-installed) as per on-screen prompts

Try: send email to ownself
Issue: Sending email from Thunderbird may fail with error "unable to login to smtp server..."
Probable cause: password mismatch between incoming and outgoing server settings
Mitigation: click on "enter new password" at the error prompt, key in password, and check "use password manager to store credentials...". Retry sending email.

### Item 10: Optional: Set up multiple SSH keys (for github and gitlab)

If you choose to have separate/different SSH keys for multiple services (e.g. one for github, another one for gitlab), follow below instructions. Otherwise, you may just copy the previously generated SSH public key and use it.

Use Item 7 to generate another SSH key, but this time give it a special name (e.g. adding `_github` at the end).

Most likely generated key pair will reside in home folder root. Move the pairs into the `~/.ssh` folder.

The following part about ssh-agent is based on: <https://bobbyhadz.com/blog/unable-to-start-ssh-agent-service-error-1058-powershell>

Open Powershell in Administrator Mode
Run:

```bash
Get-Service ssh-agent
```

Check that ssh-agent is stopped.

Run:

```bash
Get-Service ssh-agent | Select StartType
```

Should get that StartType is "Disabled".

Run:

```bash
Get-Service -Name ssh-agent | Set-Service -StartupType Manual
```

Note: -Start**up**Type, different from previous command.

Run:

```bash
Get-Service ssh-agent | Select StartType
```

Should get that StartType is "Manual" now.

To start ssh-agent, run:

```bash
Start-Service ssh-agent
```

Check that ssh-agent is running:

```bash
Get-Service ssh-agent
```

Back to **user** Powershell. Run:

```bash
ssh-add -l
```

If no errors other than saying file empty, run:

```bash
ssh-add [path to identity file]
```

In config file, modify it too look as below:

```bash
Host gitlab.com
    HostName gitlab.com
    User git
    IdentityFile C:\Users\[PC USERNAME]\.ssh\id_rsa
    IdentitiesOnly yes
    PubkeyAcceptedKeyTypes +ssh-rsa
    HostkeyAlgorithms +ssh-rsa

Host github.com
    HostName github.com
    IdentityFile C:\Users\[PC USERNAME]\.ssh\id_ed25519_github
    PreferredAuthentications publickey
```

Modify above IdentityFile paths to your actual identity files' paths (directory, filename, etc).

Test connection to github:

```bash
ssh -T git@github.com
```

You should see:

```bash
Hi [registered username on github]! You've successfully authenticated, but GitHub does not provide shell access.
```

More info: <https://gist.github.com/MatheusPoliCamilo/b66c6d49a3c8d39193175db0bce77b73>

### Item 11: Install yarn

Run the following in any directory:

```bash
npm install --global yarn
```

Check if `yarn` can be run from Powershell:

```bash
yarn --version
```

If a block of red error appears, open Powershell in admin mode, then run:

```bash
Set-ExecutionPolicy -ExecutionPolicy Unrestricted
```

More info: <https://bobbyhadz.com/blog/yarn-cannot-be-loaded-running-scripts-disabled>

If still doesn't work, add following to PATH:

```bash
%USERPROFILE%\AppData\Roaming\npm\node_modules\yarn\bin
```
