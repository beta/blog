title: Setup Go Development Environment with VS Code and WSL on Windows
date: 2019-05-10 23:25:28
tags:
- Go
- VS Code
- Windows
- WSL
---
I've got a Surface Book 2 but it hasn't been used too much for programming. Due to the lacking of things like POSIX APIs and command-line utils, web development in Windows does not come as easy as the Unix world, and some workflows of my Apple-user colleagues are impossible to achieve on Windows. So I have to use my five-year-old MacBook Pro at work, but it's getting really old and slow these days.

Recently Microsoft [published](https://code.visualstudio.com/blogs/2019/05/02/remote-development) a [Remote Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) for VS Code, which allows remote development via SSH, containers and WSL. So I'm considering if these new tools can offer me a better development experience on Windows. And I made some experiments and found them quite usable.

Here are some notes on how to setup a web development environment for Go with WSL (Windows Subsystem for Linux).

## 0. Prerequisites

- Your PC should be running a newer version of Windows 10 which supports WSL. Check WSL's [documentation](https://docs.microsoft.com/en-us/windows/wsl/about) for detail.
- Currently the remote development extensions are only supported by the [insiders build](https://code.visualstudio.com/insiders/) of VS Code. Stable builds should support them later.

## 1. Setup WSL

#### Enable WSL Feature

First of all, the WSL feature should be turned on. Open the Start Menu, search for and open "**Turn Windows features on or off**". Enable "**Windows Subsystem for Linux**" and click **OK**. The system may reboot several times.

![Windows Features dialog](https://i.imgur.com/yBcuoXG.png)

#### Install WSL Distro

After the reboot is finished, open Microsoft Store and search for "**Linux**". You should see a big banner with a "**Get the apps**" button on it. Click it. Then you will see all the Linux distros that are supported to run with WSL. Select the one that you are most familiar with or you love, then install it.

I'm using Ubuntu 18.04 LTS as I've used it for the most.

#### Shell (Optional)

The Linux distro you use comes with a built-in shell, usually bash. You may switch to another shell if you don't feel good with bash. As for me, I'm using zsh with [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh).

#### Terminal (Optional)

WSL uses Windows Console Host (conhost) by default. Search for your distro name (e.g. "Linux") in the Start Menu to run it. It's usable, though it's a little too simple (or shabby) and there's not too much to customize.

I've tried some alternatives like [Cmder](https://cmder.net/) (which uses [ConEmu](https://conemu.github.io/) under the surface), and [Hyper](https://hyper.is/) which is built with web technologies. Finally I selected [Terminus](https://eugeny.github.io/terminus/), for it has a clear UI with an easy-to-use settings (and it even supports acrylic background!).

![Terminus](https://imgur.com/JLm8to1.png)

However, as Terminus's made with Electron and it renders with GPU, it's a bit more energy-hungry than the traditional native terminal emulators. There's a [list of terminals that work well with WSL](https://github.com/sirredbeard/Awesome-WSL#terminals). You can select any one you like.

If your terminal cannot locate the home dir of the Linux distro (often it's in something like "`/mnt/c/Users/<name>`/"), a simple workaround is to use "`C:\WINDOWS\system32\wsl.exe ~`" as your custom shell. The trailing "`~`" gets you to the home dir. ([*Source*](https://github.com/microsoft/WSL/issues/1346))

## 2. Install Go in WSL

#### Install from Ubuntu Repository

Installing Go in the WSL is the same as in a normal Linux distro. In Ubuntu, the following commands will install the latest version from the official repository (which is provided by Ubuntu, not the Go team).

```bash
sudo apt update
sudo apt install golang-go
```

#### Install a Binary Release

Sometimes the distro-provided Go is outdated, or you may need a specific version. Download from [Go's website](https://golang.org/dl/) and follow the [installation guide](https://golang.org/doc/install). For example I'm installing the latest version (1.12.5) at this time:

```bash
wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.12.5.linux-amd64.tar.gz
```

Then edit environment variables in the shell's config file, e.g. `~/.bash_profile` for bash and `~/.zshrc` for zsh.

```bash
export GOPATH="$HOME/Go" # or any directory to put your Go code
export PATH="$PATH:/usr/local/go/bin:$GOPATH/bin"
```

## 3. Configure VS Code

#### Install the Remote Development Extension

Setup up VS Code is easy. First, install the remote dev extension. If you only uses WSL, the [Remote - WSL](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-wsl) extension is all you need. Otherwise you may consider the full [Remote Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack).

#### Enter Remote Dev Environment

After the remote extension is installed, you will see a colored button in the left-bottom corner of VS Code, named "Open a remote window". Click it and select "**Remote-WSL: New Window**" to open a window with remote environment enabled. Some setup work will be done now and you should see the progress in the right-bottom corner.

#### Setup the Go Extension

Then you need to setup the [Go extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.Go) for the remote dev env. Here's something different if you are new to remote development in VS Code. The VS Code environment is divided into two parts with the remote dev extension installed, which are the local part and the remote part, both including its own extensions and tools. Some extensions, specifically workspace extensions (as opposed to UI extensions), should be installed remotely to use in remote development.

![Remote environment in VS Code](https://code.visualstudio.com/assets/blogs/2019/05/02/remote-environment.png)

If you are a new user of the Go extension, it will be installed remotely automatically in the remote env. Or if you have installed it locally before, you will see a "**Install on WSL**" button on the Go extension. Click it to make it available in the remote env, and you should be prompted to reload the window.

![Install extension on WSL](https://imgur.com/s6rAAmf.png)

#### Install Go Tools

Go tools must be installed to use the functionalities of the extension. Use the "**Go: Install/Update Tools**" command in VS Code, or open the WSL shell and install manually following [the guide](https://github.com/Microsoft/vscode-go/wiki/Go-tools-that-the-Go-extension-depends-on).

#### Configure the remote Go extension

The Go extension should be re-configured remotely to work properly. Open the VS Code settings, you will see a new "**Remote (WSL)**" tab on the top. Config the Go extension here.

Or you can edit the JSON config file directly. Normally it's located in the WSL at "`/home/<name>/.vscode-remote/data/Machine/settings.json`".

**Important:** `GOPATH` and `GOROOT` must be set for the remote Go extension, as it does not know where to read the environment variables you've set in your shell configs.

Below is my extension config for reference.

```json
{
    "terminal.integrated.shell.linux": "/usr/bin/zsh",
    "go.gopath": "/home/beta/Coding/Go",
    "go.goroot": "/usr/local/go",
    "go.formatTool": "goformat",
    "go.autocompleteUnimportedPackages": true,
    "[go]": {
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": true
        }
    }
}
```

Now you've setup a fully working Go development environment with VS Code and WSL. Happy coding :)

## 3. FAQ

**(Why are my Go extension configs not working?)** If you set something in the remote configs, for example changing the format tool, but it's not working, check if there is a workspace config file and a different value is set there. Workspace configs will shadow the others.
