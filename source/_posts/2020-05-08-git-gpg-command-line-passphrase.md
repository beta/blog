title: Using Command-Line Passphrase Input for GPG with Git (for Windows)
date: 2020-05-08 15:17:27
tags:
- GPG
- Git
---
## The problem

I'm using Git for Windows, and have configured it to sign every single commit and tag using GPG (GnuPG), which uses Pinentry, a program that allows for secure entry of PINs or passphrases.

Every time when I commit my code, a Pinentry window will always popup and ask me for the passphrase to unlock my key. However, the window never gains focus, and I have to move my mouse and click on the input box to start typing, which is quite annoying to me.

![A Pinentry window without focus](/images/2020-05-08-pinentry.png)
<center><small>A Pinentry window without focus</small></center>

A [bug report](https://dev.gnupg.org/T4123) is found on GnuPG's Phabricator, but seems there's still no solution or workaround.

## Trying to use the "loopback" (command-line) mode of Pinentry

After digging deep into [GnuPG's documentations](https://www.gnupg.org/documentation/manuals/gnupg/Agent-OPTION.html), I found out that there's a "loopback" mode of Pinentry, which will redirect the passphrase queries back to the caller (GnuPG), so that the user will be prompted to input the passphrases directly in the command line.

Loopback mode is disabled by default. To enable it, edit the config of GPG agent (`~/.gnupg/gpg-agent.conf`) and add the following line.

```
allow-loopback-pinentry
```

Tell GPG to reload the config with `gpg-connect-agent reloadagent /bye`. After that, you will be able to add `--pinentry-mode loopback` to `gpg` commands.

![Pinentry loopback mode](/images/2020-05-08-pinentry-loopback.png)

Looks great. But...

## How to use loopback mode when signing commits with Git?

First I thought there might be some Git config that can be used to add extra arguments to GPG. There's a [`gpg.program`](https://git-scm.com/docs/git-config/2.26.0#Documentation/git-config.txt-gpgprogram) config for setting a custom program instead of `gpg` used by Git. I tried to set it to `gpg --pinentry-mode loopback`, but it won't work, throwing me errors like "*cannot spawn gpg --pinentry-mode loopback: No such file or directory*".

Then I wrote a Bash script to serve as a "proxy" for GPG. The script writes simple, just adding the loopback arg and letting all the other args pass through.

```bash
#!/usr/bin/bash
gpg --pinentry-mode loopback $@
```

Save it somewhere (`~/.gpg-pinetry-loopback` for me), grant the execution permission to it with `chmod +x ~/.gpg-pinentry-loopback`, and set the full, Windows-style path as the `gpg.program` config for Git:

```bash
> git config --global gpg.program "C:/Users/beta/.gpg-pinentry-loopback"
```

Now let's see if it works.

![Pinentry loopback mode](/images/2020-05-08-git.png)
<center><small>Bingo</small></center>

## Wrapping things up

1. Add `allow-loopback-pinentry` to `~/.gnupg/gpg-agent.conf`.

2. Reload GPG config.

```bash
gpg-connect-agent reloadagent /bye
```

3. Create a Bash script at `~/.gpg-pinentry-loopback` (or any path you like).

```bash
#!/usr/bin/bash
gpg --pinentry-mode loopback $@
```

4. Grant execution permission to the script. Replace the path with your own.

```bash
chmod +x ~/.gpg-pinentry-loopback
```

5. Set the script as the GPG program for Git. Replace the path with your own (full Windows-style path required).

```
git config --global gpg.program "C:/Users/beta/.gpg-pinentry-loopback"
```
