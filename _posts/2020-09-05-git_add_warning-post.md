---
layout: post
title: git add warning
date: 2020-09-04 18:00:00 +0800
category: debug
thumbnail: /style/image/thumbnail.png
icon: note
---

* content
{:toc}

# git add时的警告

在使用git add 命令后提示警告
> warning: CRLF will be replaced by LF in dir/note.md.
The file will have its original line endings in your working directory.

其原因是:
* Unix系统行尾使用 **换行line feed(LF)** 表示
* Windows系统一行用 **回车carriage return(CR)** 和 **换行(LF)** 表示,即为 **CRLF**

git add 时会自动对换行符进行替换

如果是个人开发,可以执行以下命令忽略该警告
```tcl
git config core.autocrlf true
```
在签出代码时，git会将LF结尾转换为CRLF


如果是团队协作(跨平台)项目, 建议与团队进行沟通并保持统一的换行,然后修改编辑器设置为指定换行类型,尽量避免git对文件的自动替换.

**建议不要使用git的自动转换!!!**

******************************

原文写的更清楚:[跳转原文](http://vcloud-lab.com/entries/devops/resolved-git-warning-lf-will-be-replaced-by-crlf-in-file)
部分节选:

> In Unix systems the end of a line is represented with a line feed (LF). 
> In windows a line is represented with a carriage return (CR) and a line feed (LF) thus (CRLF).
> When you get code from git that was uploaded from a unix system they will only have an LF.


If you are a single developer working on a windows machine, and you don't care that git automatically replaces LFs to CRLFs, you can turn this warning off by typing the following in the git command line.

```tcl
git config core.autocrlf true
```

If you want to make an intelligent decision how git should handle this, read the documentation.
> `Formatting and whitespace issues are some of the more frustrating and subtle problems that many developers encounter when collaborating, especially cross-platform. It’s very easy for patches or other collaborated work to introduce subtle whitespace changes because editors silently introduce them, and if your files ever touch a Windows system, their line endings might be replaced. Git has a few configuration options to help with these issues.
```
core.autocrlf
```
If you’re programming on Windows and working with people who are not (or vice-versa), you’ll probably run into line-ending issues at some point. This is because Windows uses both a carriage-return character and a linefeed character for newlines in its files, whereas Mac and Linux systems use only the linefeed character. This is a subtle but incredibly annoying fact of cross-platform work; many editors on Windows silently replace existing LF-style line endings with CRLF, or insert both line-ending characters when the user hits the enter key.

Git can handle this by auto-converting CRLF line endings into LF when you add a file to the index, and vice versa when it checks out code onto your filesystem. You can turn on this functionality with the core.autocrlf setting. If you’re on a Windows machine, set it to true – this converts LF endings into CRLF when you check out code:
```
git config --global core.autocrlf true
```


I would still recommend:
```
git config core.autocrlf false
git add --renormalize .
git commit -m "Do not touch eol"
```
If you can, avoid Git making any change to your eol, and work with editors which respect the eol of the file being edited.