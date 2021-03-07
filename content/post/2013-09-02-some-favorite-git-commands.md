---
title: Some favorite git commands
author: yan
date: 2013-09-02
url: /some-favorite-git-commands/
categories:
  - code

---
Working on HTTPS Everywhere, an open source project with dozens of contributors, has sharpened my git vocabulary immensely. I figured I&#8217;d list a few lesser-known commands that I like:

  * _git log_ __-pretty=oneline -n<number> &#8211;abbrev-commit_ -G<regex>_: This shows the <number> latest commits in oneline format with shortened commit hashes that added or removed lines matching <regex>. The git pickaxe options (-S, -G) are super useful for searching git commit contents instead of just the commit messages.
  * _git checkout <branch> <path/to/file>; git reset; git add -p; git commit_: This is almost always less preferable to _git cherry-pick_, but it is useful when you want only certain hunks of commits to certain files in <branch>. _git checkout_ copies the file from the other branch into the current branch and stages the changes for commit. _git reset_ unstages them so we can select the parts we want to add to our branch on a hunk-by-hunk basis using _git add -p_. Unfortunately this doesn&#8217;t preserve authorship of the changes from <branch>.
  * git update hooks: We decided that it would be great if the HTTPS Everywhere git server could automatically reject commits that broke the build. It turns out that this is possible using a server-side git update hook, which runs just before updating the refs on the git server. The strategy is to create a temporary copy of the remote with the newly pushed changes, do a test build, and reject the changes if the build fails. See [here][1] for an example of this in HTTPS Everywhere adapted from a Stack Overflow answer.

More to come!

 [1]: https://github.com/diracdeltas/https-everywhere/blob/master/hooks/update
