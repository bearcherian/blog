---
title: "Git Users and Their Last Commits"
date: 2019-03-19T12:19:25-04:00
description: Getting a list of users who have committed to a repo, by their last commit date
tags: 
- git
- logs
- bash
- linux
draft: false
---

I needed to remember who was working with me on a certain project over the last year. I had some trouble remembering, because this last year has been crazy personally, and the project overlapped a couple years. All the years and months and days were running together in my brain.

Fortunately, we have git logs, but it's not straightforward. I came up with this set of commands to help with the task:

```
git log --pretty=format:"%an" | \
    sort | \
    uniq | \
    awk '{system("git log -1 --author="$1 " --pretty=format:\"%ad%x09%an\""); print "\r"}' | \
    sort -k5n -k2M -k3n
```

Here's what we're doing:

1. get all of the user names in the git logs
+ sort them lexically
+ remove repeats (sort -u could have replaced these)
+ do another git log for each user, but print out their last commit, and format it to be just the date and their name separated by a tab. 
+ Sort by the columns (year, month, date)
