+++
title = "Personalising Windows Folder Icons"
description = ""
tags = [
    "windows",
    "personalisation",
]
date = "2023-06-21"
categories = [
    "Windows",
]
menu = "main"
+++

## Overview

Source Code Management is a tool that is used by an increasing number of people; including myself as an IT Pro. Like anyone that uses source code management you likely have a folder containing multiple repositories from your personal code, your companies or anothers.

One thing that I wanted to be able to do was customise my folder icons so it _looks_ like a folder containing repositories rather than looking like any other old folder in Windows. It's relatively simple but can help you easily spot your Repository folder to get you working quicker.

## Get Icons

So I only needed three icons for what I wanted to do. I needed an icon for:

- Git SCM
- Azure DevOps
- GitHub

These three can be found in my GitHub - [MattDaines - Folder Icons](https:\\github.com\MattDaines\Folder-Icons).

If you want to create your own icons you should know that Windows expects the file format to be a `.ICO`, `.ICL`, `.EXE`, or a `.DLL` (possibly maybe others, but Windows gives the hint when you select `Browse` when changing the icon!). I chose to create .ICO files because I figured it was likely the easiest format for me to convert regular image file formats such as `.JPG`, `.PNG` etc, into an `.ICO`. It was made easy by another repository I found: [FoxP - PNG-to-ICO](https:\\github.com\FoxP\PNG-to-ICO); so thanks for that! 😊

## Update Icons

When you have the icons you need, then it's time to update the folder icons!

1) Go to the folder you wish to change
2) Right click the folder, select `Properties`
3) Select the `Customise` tab
4) Select the `Change Icon...` option at the bottom
5) Select `Browse`
6) Navigate to the `.ICO` you wish to update the folder icon to and select `Open`
7) Select `Ok`
8) Select `Ok`, again

It's that simple 😊

![image](imgs/file-explorer-after.png)
