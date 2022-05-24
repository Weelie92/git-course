# Git Course
This is a simple course to give a quick guide to git.

## Git tools
For this course I highly recommend only using the command line, just to get a feel of it. Otherwise, I recommend
the follwing tools:
- [GitHub Desktop](https://desktop.github.com/) - For simple commiting and amending. There is some squashing and
reordering available, but I generally move to the command line for these kinds of things.
- [TortoiseGit](https://tortoisegit.org/) (Windows only) - This has everything. I don't use it personally, but it 
is used by fellow trusted git enthusiasts.
- Git integrations in your IDE. These vary some, but are OK to use.

## Git config
Just a quick note before we start. There are a lot of ways to use git more efficiently, and a lot of preferences
to tweek. I created [this gist](https://gist.github.com/anbergem/3d834671a4bb153dd6833b7eca7d3a3e), which is a
collection of settings I have accumulated over the years. Feel free to take what you want. The examples later
are printed using the `git lg` command. It is my sole most used command, along with `git status`. Of course, I
did not create this alias. It is taken from 
[this stack overflow post](https://stackoverflow.com/questions/1057564/pretty-git-branch-graphs).

## Core concepts

### Commit
A commit is simply a set of changes to a single or multiple files, with a message describing the changes.
Every commit has a hash, which is based on the content it self, the message and the timestamp of the commit.
If any of these are changed, the hash will change.

### Remote
A remote is a git server where the repository is stored. It is a common place where multiple people can 
access the same git repository. A remote is not required, although sharing a git repository without one 
is a lot more work...

Call your default remote for "origin". Don't be funny and call it something else. "origin". Always "origin".

A git repository can have multiple remotes. An example of this is the hosting service 
[Heroku](https://www.heroku.com/). After setting things up, you can deploy your code to heroku server using a 
git remote. For example if this remote is called "heroku", we can say `git push heroku master`.
This will push the master branch to heroku's master branch, which on heroku's server will trigger the code
deploy your application.

When having multiple remotes, they may not always be in sync. Every push needs to specify which remote 
you are pushing to.

### Branch
It is assumed you know what a branch is. If not, go look it up. In short, it is just a pointer to a commit.

Common commands:
```sh
# Create a branch where HEAD is
git checkout -b my-branch

# Create a branch, putting it's base somewhere else
git checkout -b my-branch other-branch  # my-branch will now be at the same location as other-branch

# Force create a new branch where HEAD is.
git checkout -B my-branch  # Warning: This may delete all commits that are only pointed to by my-branch

# Force create a new branch, putting it's base somewhere else
git checkout -B my-branch other-branch # Warning: This may delete all commits that are only pointed to by my-branch
```
I personally use this last one a lot, doing `git checkout -B master origin/master`, after a fetch. This 
moves my local master up to where origin's master is, *and* moved my HEAD to point at master as well. I
use it so much that I created a separate command for it in my 
[git config gist](https://gist.github.com/anbergem/3d834671a4bb153dd6833b7eca7d3a3e): `git cu master`. 
This specifically updates the branch to the origin of that branch. `cu` is for `checkout and update`.

### HEAD
`HEAD` is used a lot in git. `HEAD` is simply just where you are, and `origin/HEAD` is where origin's 
(the remote named origin) head is. This will almost always be at `origin/main` (GitHub) or `origin/master` 
(everywhere else).

### Tag
A tag is just another type of pointer to a commit. This is often used for versioning. Tags are more difficult to
delete, so if you have pushed a tag. Make sure it's safe to delete it. 

### Push
Pushing is just the act of pushing a branch or a tag to a remote. When pushing a new branch to a remote, you
have to specify the name of the new branch on the remote as well, typically:
```
git push origin --set-upstream-to your-branch
```
This is tedious. Feel free to steal my command in the [git config gist](https://gist.github.com/anbergem/3d834671a4bb153dd6833b7eca7d3a3e): `git push-u`

#### Force pushing
> WARNING: Force pushing is used to rewrite the truth in the remote. Use with causion for your own branch, 
*extreme causion* if force pushing to a branch that is not your own, and **EXTREME CAUSION** if force
pushing to the main branch!

If you have done changes to any commits that have previously been pushed to a branch, a normal push will not work,
since the history does not match. You will then have to force push.
```
git push --force-with-lease
```
I show you this command first, so that you will remember it better. This is a safer version of force pushing, that
will only force push, if no one else has pushed to the branch in the meantime. Always use `--force-with-lease`. There
should really be no option to just use `--force`...


### Re-writing history
#### Amend
Amending a commit is the simplest way of rewriting history. It is simply just doing small changes to your latest
commit. These changes can be code changes, or simply just editing the commit message.
```sh
git add some/file.py # Optionally add some changes
git commit --amend
```
The will prompt for a new commit message, where you can either leave it be as it was, or change it.

It is a very handy tool, which should definitely be put in your arsenal. 

Note: Since changes are made, the commit hash will not be the same. Therefore, a force push is required if the 
remote needs to be updated. If you haven't pushed the latest commit yet, you're good to go.


#### Cherry Picking
Before we get to rebasing, let's take a quick look at cherry picking. Cherry picking is not really rewriting 
history, but it might be easier to understand rebasing, if we understand it. As you might guess from the name,
a cherry pick is just picking a commit and putting it onto your HEAD. It is quite simple.
```sh
git cherry-pick some-branch  # Pick the latest commit from branch "some-branch"
git cherry-pick some-branch~1  # Pick the second to latest commit from branch "some-branch"
git cherry-pick e799fa4  # Pick the specific commit e799fa4
```
Doing this will result in the changes from the specific commit being applied to where you are. This can be any 
commit from any branch! Of course, the changes that the commit represent may not fit with the current state
that you are in, and you may therefore get a conflict:
```
λ git cherry-pick 4ccb363
Auto-merging path/to/file
CONFLICT (content): Merge conflict in path/to/file
error: could not apply 4ccb363... Some commit message
hint: after resolving the conflicts, mark the corrected paths
hint: with 'git add <paths>' or 'git rm <paths>'
hint: and commit the result with 'git commit'
```

Conflicts can be a pain, we'll cover that later. To proceed now, we have to fix the conflict, proceeded by:
```sh
git add path/to/file
git status
# On branch tmp
# Cherry-pick currently in progress.
#   (run "git cherry-pick --continue" to continue)
#   (use "git cherry-pick --skip" to skip this patch)
#   (use "git cherry-pick --abort" to cancel the cherry-pick operation)
# 
# Changes to be committed:
#         modified:   path/to/file
git cherry-pick --continue
```
As we see, if we are way over our heads when doing a cherry-pick and getting conflicts, we can always abort. This
can also be done before doing `git add`.

Ok, so what about rebasing?

Rebasing is simply just cherry-picking multiple (or a single) commits, one by one. We can, of course, do this manually:
```sh
git cherry-pick e799fa4..4229616  # Pick a range of commits, exluding the last commit.
```
This, however, is quite limiting. Rebasing is a powerful feature of git where you can automatically or interactively 
rewrite your git history. But for most purposes, it is just simple cherry picking.

#### Rebasing
> WARNING: Rebsing rewrited history. When starting out, use a tmp branch to save your work. Accidentically deleting
your branch or wrongly rewriting history is not easily reversable. `git checkout -b tmp`, `git checkout your-branch`.

##### Plain rebasing
Ok. Let's jump right into it. Say you are in this situasjon:
```
*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
| * cf5375e - Yet another commit (HEAD -> your-branch)
| * 73fb2db - Another commit 
| * 9c539f8 - A commit. 
|/
*   d707771 - Merged PR 125: Some PR 
```
You started on master, off commit `d707771`, the commit
for PR (Pull Request) 125. You went off into your branch, `your-branch`, and made some commits. In the mean time,
there has been another PR, PR 126. The PR may be related to the work you have done, or it might not be relevant at 
all, not touching any of the same files. Either way, before making a PR for `your-branch` into `master`, it is a 
good idea to make sure that your branch is "up to speed" with master. You want to make sure that when you make a PR,
there shouldn't be any conflicts to resolve when merging into master. The simplest way of ensuring that is to rebase
your branch onto master.

Now that we have learnt about cherry picking, we know that this means that we want to move our base up to master, 
and then take the commits from `your-branch` one by one. Lastly, we move `your-branch` to point to where `HEAD` is.

This is visualizing the steps, one by one:
```
Starting point
*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
| * cf5375e - Yet another commit (HEAD -> your-branch)
| * 73fb2db - Another commit 
| * 9c539f8 - A commit. 
|/
*   d707771 - Merged PR 125: Some PR 

Cherry-pick first commit
*   9c539f8 - A commit. (HEAD)
*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
| * cf5375e - Yet another commit (your-branch)
| * 73fb2db - Another commit 
| * 9c539f8 - A commit. 
|/
*   d707771 - Merged PR 125: Some PR 

Cherry-pick second commit
*   73fb2db - Another commit (HEAD)
*   9c539f8 - A commit. 
*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
| * cf5375e - Yet another commit (your-branch)
| * 73fb2db - Another commit 
| * 9c539f8 - A commit. 
|/
*   d707771 - Merged PR 125: Some PR 

Cherry-pick third commit
*   cf5375e - Yet another commit (HEAD)
*   73fb2db - Another commit
*   9c539f8 - A commit. 
*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
| * cf5375e - Yet another commit (your-branch)
| * 73fb2db - Another commit 
| * 9c539f8 - A commit. 
|/
*   d707771 - Merged PR 125: Some PR 

Move your-branch to where HEAD is
* cf5375e - Yet another commit (HEAD -> your-branch)
* 73fb2db - Another commit 
* 9c539f8 - A commit. 
* e799fa4 - Merged PR 126: New PR. (master, origin/master)
* d707771 - Merged PR 125: Some PR 
```
Of course, when rebasing, this is not referred to as cherry-picking at all. That is just my way of explaining it.

For each of these steps, the cherry-pick is applied, and so for each step, there may be a potential conflict. That 
means that if there are conflicts, we have to solve the conflicts for each commit, add the files, and then run 
`git rebase --continue`, just like we did `git cherry-pick --continue`.

When experiencing this the first time, it may be a little confusing. Say that you experience a conflict in the first 
part of the rebase (cherry picking `9c539f8 - A commit.`). You are now resolving the conflicts that have happened 
only between `e799fa4 - Merged PR 126: New PR.` and `9c539f8 - A commit.`. The other, future commits have not 
happened yet. You may be used to how all the files look at the end of your work, that is, at `your-branch`. But 
the rebase hasn't gotten that far yet. If this wasn't clear, hopefully the exercise on this will help.

We are now done. We can create our PR, and when merged, it may look something like this:
```
* c5838b3 - Merged PR 127: PR from your-branch (HEAD -> master, origin/master)
* e799fa4 - Merged PR 126: New PR. 
* d707771 - Merged PR 125: Some PR 
```

> Quick note: Since we never pushed our branch to the remote, we can simply push the branch from it's rebased 
position. If we had already pushed it once before, we would be in this position:
>```
>*   cf5375e - Yet another commit (HEAD -> your-branch)
>*   73fb2db - Another commit 
>*   9c539f8 - A commit. 
>*   e799fa4 - Merged PR 126: New PR. (master, origin/master)
>| * cf5375e - Yet another commit (origin/your-branch)
>| * 73fb2db - Another commit 
>| * 9c539f8 - A commit. 
>|/
>*   d707771 - Merged PR 125: Some PR 
>```
>We would then have to force push, to update `origin/your-branch`. This is because we have re-written history.

> Quick note: This is the case when using a merge strategy of "squash and merge". Another strategy may look 
something like this:
> ```
> *   c5838b3 - Merged PR 127: PR from your-branch (HEAD -> master, origin/master)
> |\
> | * cf5375e - Yet another commit
> | * 73fb2db - Another commit
> | * 9c539f8 - A commit.
> |/
> *   e799fa4 - Merged PR 126: New PR. 
> ```
> This is a merge strategy called "semi-linear merge", and is only available in Azure DevOps (and maybe GitLab). It
is by far my preferred merge strategy.

##### Interactive Rebasing
We have now talked about rebasing to update your branch to the new version of the code base. This is one of the two 
most common uses of rebasing. The other is interactive rebasing.

Let's just jump right into an example. Let's take our example from earlier, where we have this starting point:
```
* cf5375e - Yet another commit (HEAD -> your-branch)
* 73fb2db - Another commit 
* 9c539f8 - A commit. 
* e799fa4 - Merged PR 126: New PR. (master, origin/master)
```
We may want to do some changes here. Say that we have a small fix that should've been a part of 
`9c539f8 - A commit.`, but you forgot, or that you found a bug in that code that should be fixed. Of course,
we can simply write a new commit with the message "Fixed some bug". However, the branch isn't official yet,
and isn't really commited and merged into the code. It is still just a branch that you are developing on. Wouldn't
it be better to actually fix the commit, while you've working on it, than to have a new commit that will be there
for all time, just to fix a commit that is on your own branch? The answer is of course "YES!"

Enter interactive rebasing.

We run the command `git rebase -i master`. This says that we want to do a new multiple cherry-pick rebase from 
master, applying all my commits again. This time, however, we want to do it interactively, hence the flag `-i`. 

We will get the following prompt:
```
pick 9c539f8 - A commit. 
pick 73fb2db - Another commit 
pick cf5375e - Yet another commit

# Rebase 9c539f8..cf5375e onto e799fa4 (5 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Read through this, and see if you understand this. Take note of the following:
- pick
- reword
- squash
- fixup
- drop

These are the ones that I personally use. Also note the order of the commits. The oldest is at the top.

> Quick note: squash and fixup says "meld into the previous commit". Note that due to the order, this will be the 
commit above in the rebase window.

#### Fixup
The last history re-writing tool is a simple fixup. This is just a shortcut to doing an interactive rebase where
you merge two commits with a fixup. 

Again, take the following example:
```
* cf5375e - Yet another commit (HEAD -> your-branch)
* 73fb2db - Another commit 
* 9c539f8 - A commit. 
* e799fa4 - Merged PR 126: New PR. (master, origin/master)
```

Say we just want a simple code change to `9c539f8 - A commit.`. We can add the file changes with `git add`,
followed by `git commit --fixup 9c539f8`. This will create a commit that looks something like this:
```
* b7f277c - fixup! A commit. (HEAD -> your-branch)
* d872fe7 - Yet another commit
* 73fb2db - Another commit 
* 9c539f8 - A commit. 
* e799fa4 - Merged PR 126: New PR. (master, origin/master)
```

We can now run the command `git rebase -i --autosquash`, we now get this prompt:
```
pick 9c539f8 - A commit. 
fixup b7f277c - fixup! A commit.
pick 73fb2db - Another commit 
pick cf5375e - Yet another commit

# Rebase 9c539f8..cf5375e onto e799fa4 (5 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
The fixup commit is automagically moved to where it is supposed to be. As this is the case, we don't have to
do any changes. We can simply close/exit the editor, and the rebase will take place, squashing the two commits,
using the commit message of `9c539f8 - A commit`.

This is a *very* nice tool, which you are encouraged to use.

> Quick tip: run `git config --global rebase.autosquash true` to use autosquashing always when rebasing. I have
never come across a time where this was an unwanted behaviour. In my opinion, it should be the default.

## Tasks
Now that you have learnt the gist of it, let's try it out! There are multiple branches in this repo. They are
in the format `task/description` and `soltion/description`. Good luck!