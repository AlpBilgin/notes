# Git

## From MAN pages

1

git config --list --show-origin makes a list of all git settings active in a folder and where they come from.

## From https://opensource.com/article/18/4/git-tips

1

I need to periodically merge the changes from the upstream repo into my fork—which I do by using an alias I call upstream-merge. It's defined like this:

`upstream-merge = !"git fetch origin -v && git fetch upstream -v && git merge upstream/master && git push"`

**The ! at the beginning of the alias definition tells Git to run the command via the shell.** This example involves running a number of git commands, but aliases defined in this way can run any shell command.

2

`git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative`

The --graph option adds the graph to the left side of the log, --abbrev-commit shortens the commit SHAs, --date=relative expresses the dates in relative terms, and the --pretty bit handles all the other custom formatting. I have this aliased to git lg, and it is one of my top 10 most frequently run commands.

3

One of the hazards with force pushes happens when somebody else has made changes on top of the same branch in the remote copy of the repository. When you force-push your rewritten history, those commits will be lost. This is where `git push --force-with-lease` comes in—it will not allow you to force-push if the remote branch has been updated, which will ensure you don't overwrite  someone else's work.

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

## From http://365git.tumblr.com/

1

add a new .gitignore file in an empty directory to commit it while ignoring the contents

## From https://devops.stackexchange.com/questions/1325/jenkins-shows-the-job-as-failed-if-there-is-nothing-to-commit-to-gitlab

git diff-index --quiet HEAD || git commit -m "Jenkins automatic update commit"

second command is only reached if first command spits out a difference

## from SO

1

ignore commited files

add to .gitignore
git -rm --cached /path/to/file

## From GISTS

1
https://gist.github.com/myusuf3/7f645819ded92bda6677

To remove a submodule you need to:

- Delete the relevant section from the .gitmodules file.
- Stage the .gitmodules changes git add .gitmodules
- Delete the relevant section from .git/config.
- Run git rm --cached path_to_submodule (no trailing slash).
- Run rm -rf .git/modules/path_to_submodule (no trailing slash).
- Commit git commit -m "Removed submodule "
- Delete the now untracked submodule files rm -rf path_to_submodule
