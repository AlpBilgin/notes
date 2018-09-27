# notes

## New install breadcrumbs

Read sibling sections for tips on the listed tools

### Terminal Emulator

NTFS: cmder vanilla
Posix: zsh + oh-my-zsh powerlevel9k theme

### Node

point npm global folder to a user dir. Then install pnpm. Then use pnpm to install n. Use npm only when switching to an older node with n breaks pnpm.

### Python

Read up again on the python package that exposes different versions per project.

## Git

### From MAN pages

1

git config --list --show-origin makes a list of all git settings active in a folder and where they come from. 

### From https://opensource.com/article/18/4/git-tips

1

I need to periodically merge the changes from the upstream repo into my fork—which I do by using an alias I call upstream-merge. It's defined like this:

`upstream-merge = !"git fetch origin -v && git fetch upstream -v && git merge upstream/master && git push"`

**The ! at the beginning of the alias definition tells Git to run the command via the shell.** This example involves running a number of git commands, but aliases defined in this way can run any shell command.

2

`git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative`

The --graph option adds the graph to the left side of the log, --abbrev-commit shortens the commit SHAs, --date=relative expresses the dates in relative terms, and the --pretty bit handles all the other custom formatting. I have this aliased to git lg, and it is one of my top 10 most frequently run commands.

3

One of the hazards with force pushes happens when somebody else has made changes on top of the same branch in the remote copy of the repository. When you force-push your rewritten history, those commits will be lost. This is where `git push --force-with-lease` comes in—it will not allow you to force-push if the remote branch has been updated, which will ensure you don't throw away someone else's work.

4

This flag will cause the git add command to look at all the changes in your working copy and, for each one, ask if you'd like to stage it to be committed, skip over it, or defer the decision (as well as a few other more powerful options you can see by selecting ? after running the command). `git add -p` is a fantastic tool for producing well-structured commits.

5

Some projects have a rule that each commit in the repository must be in a working state—that is, at each commit, it should be possible to compile the code or the test suite should run without failure. This is not too difficult when you're working on a branch over time, but if you end up needing to rebase for whatever reason, it can be a little tedious to step through each rebased commit to make sure you haven't accidentally introduced a break.

Fortunately, git rebase has you covered with the -x or --exec option. `git rebase -x <cmd>` will run that command after each commit is applied in the rebase. So, for example, if you have a project where npm run tests runs your test suite, git rebase -x npm run tests would run the test suite after each commit was applied during the rebase. This allows you to see if the test suite fails at any of the rebased commits so you can confirm that the test suite is still passing at each commit.

6

you can simply run git diff HEAD@{yesterday}, and see all the changes that have happened since then. This also works with longer time periods (e.g., git diff HEAD@{'2 months ago'}) as well as exact dates (e.g., git diff HEAD@{'2010-01-01 12:00:00'}).

You can also use these date-based revision arguments with any Git subcommand that takes a revision argument. Find full details about which format to use in the man page for gitrevisions.

7

Have you ever rebased away a commit, then discovered there was something in that commit you wanted to keep? You may have thought that information was lost forever and would need to be recreated. But if you committed it in your local working copy, it was added to the reference log (reflog), and you should still be able to access it.

Running git reflog will show you a list of all the activity for the current branch in your local working copy and also give you the SHA1 of each commit. Once you've found the commit you rebased away, you can run git checkout <SHA1> to check out that commit, copy any information you need, and run git checkout HEAD to return to the most recent commit in the branch.

### From http://365git.tumblr.com/

1

add a new .gitignore file in an empty directory to commit it while ignoring the contents

## macOS

### From SO

1

show hidden: Just adding the key to the global domain seems to work:
`defaults write -g AppleShowAllFiles -bool true`
You have to quit and reopen applications to apply changes as usual.

## Node.JS

### From Docu

1

avoid npm permission errors that come with running everything in sudo.

https://docs.npmjs.com/getting-started/fixing-npm-permissions

2

Avoid npm itself

https://www.npmjs.com/package/pnpm
