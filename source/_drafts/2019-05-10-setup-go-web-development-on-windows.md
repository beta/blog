title: Setup Go Web Development Environment with WSL on Windows
date: 2019-05-10 23:25:28
tags:
- Go
- VS Code
- Windows
- WSL
---
I've got a Surface Book 2, but it hasn't been used too much for programming. Due to the lacking of things like POSIX APIs and command-line utils, web development in Windows does not come as easy as the Unix world, and some workflows of your Apple-user colleagues are impossible to achieve on Windows. So I have to use my five-year-old MacBook Pro at work, but it's getting really old and slow these days.

Recently Microsoft [publishes](https://code.visualstudio.com/blogs/2019/05/02/remote-development) a [Remote Development Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack) for VS Code, which allows remote development via SSH, containers and WSL. So I'm considering if these new tools can offer me a better development experience on Windows. And I made some experiments and found them quite usable.

Here are some notes on how to setup a web development environment for Go with WSL (Windows Subsystem for Linux).

## 0. Prerequisites

- Your PC should be running a newer version of Windows 10 which supports WSL. Check WSL's [documentation](https://docs.microsoft.com/en-us/windows/wsl/about) for detail.
- Currently the remote development extensions are only supported by the [insiders build](https://code.visualstudio.com/insiders/) of VS Code. Stable builds should support them later.

## 1. Setup WSL

First of all, the WSL feature should be turned on. Open the Start Menu, search for and open "**Turn Windows features on or off**". Enable "**Windows Subsystem for Linux**" and click **OK**. The system may reboot several times.

![Windows Features dialog](https://i.imgur.com/yBcuoXG.png)

After the reboot is finished, open Microsoft Store and search for "**Linux**". You should see a big banner with a "**Get the apps**" button on it. Click it. Then you will see all the Linux distros that are supported to run with WSL. Select the one that you are familiar with or you love, then install it.

I'm using Ubuntu 18.04 LTS as I've used it for the most.

## 2. Terminal
