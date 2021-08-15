# Git for those who don't want to learn git.

For those who don't want to learn git, git is a version control system. It's
a way of working together on code in a decentralized way, where everyone's allowed
to work on their own copy of code, and then merge it whenever they wish in a 
relatively straight forward manner.

Despite how simple it sounds, git does a fairly complex job, because not only
does it make it easy to work in a decentralized fashion, it keeps track of the 
*history* of the codebase such that you can revert changes you've made (or merged)
both from yourself, and from other people whose code (*or history*) you've merged
with your own. 

As a tl;dr this guide dosen't focus on how to use and become adept with git, but only
what the bare basics of commands you should know and when you should use them. Learning
what these commands do to the history and developing a mental model for git is important,
but thise guide for the most part intends to skip over this (albeit crucial) detail.
The goal is only to get to the bare minimum amount of usability with git as fast
as possible.

# Git is about history.

Git stores lists of changes you've made over time to your code base called *commits*.
Each commit contains a commit message, and a set of changes you've made to your code.
You can view this *history*, or list of commits in any repository with the command 
`git log`. When you merge code into your repository, you take history from another 
"repository" (or rather branch as you'll learn in a bit) and you add it to your own. 
There's 3 major methods to do this, depending on what you're doing, but first you 
need to be able to make history.

# Making history

Git has 3 major areas where you can think of it as storing changes. It has the *unstaged*,
*staged*, and *commited* sets of changes. *Unstaged* changes are changes you've made, but
you haven't told git you're done making them yet. Any changes that are *staged* are changes
you've told git you're going to commit next. *Commited* changes are changes which are
already finalized, and been put on the git history. You can view the *unstaged* and *staged*
areas by doing `git status`. If you do `git add (folder or directory)`, you'll add the folder
or directory to the stage (you can verify this by doing `git status` again). If you do 
`git commit`, it'll open up an editor for you to write your commit message. Once you 
write your commit message, save, and exit, that commit will appear in `git log`, as 
it's now a part of the history. 

Making changes, staging changes, and then commiting changes regularly is a common part of using git. 
I *highly* reccomend making small commit regularly, and commiting when *any* unit of work is done.
No matter how small, if a feature is implemented, you should commit the feature. If you fix a bug,
commit the bug fix, and when you write a function, commit the function. It's also good pratice 
to describe how and why you make certain choices in your commit message, as it can help anyone
reviewing your code understand your choices. While this can in of itself make it hard to revert
larger changes using git, you can fix this relatively easily by using another git feature called
branches.

# Branches
In git, branches are "repositories" that share history up until a certain point. You can merge
history across branches, and you can delete branches as well. As a workflow, it's best to always
make a branch for any non-trivial change you're making, especially if it isn't your code base
but one that has other people who work on it as well. When you're working on a new feature: make
a new git branch. This can be done with `git branch (branch name)`. The branch name should ideally
succinctly as possible describe the feature you're doing. If you're fixing a bug on your issue
tracker, you can call it issue-343 or whatever - the name should be succinct and give an idea 
of what it's for. 

Once you've made a branch, you need to *switch* to that branch, which changes the git history
from your current branch's to the git history of another branch. This is done with 
`git checkout (branch name)`. It's worth noting that making and switching branches is common
enough where there's a shortcut to do both operations at the same time.
`git checkout -b (branch name)` makes a branch, and then switches to that branch. Once you've
done that, start working on your feature (don't forget to commit regularly!). Once you're done,
you can merge the branch locally, (or if you're using github, you can push your branch and then
submit a pull request). 

# Merging history.
There's 3 major strategies to merging history across branches: There's fast-forward, rebase, and 
then there's merge. Fast-forward is the simplest, and can be done with  `git merge --ff-only`. 
If one branch has more history then another, and the second branch dosen't have any 
unique history of it's own, the two branches just end up with identical history. It essentially 
just copies and pastes the history of the branches as long as one history is a direct subset 
of another. This is a useful way of keeping two branches in sync. One common usecase I have
for this is if I have a production branch which is deployed on my server (the server regularly
merges this branch from github), and I have a development branch, I can *fast forward merge* the
production branch with the development branch, which copies all the history from development to 
the production branch. 

The second kind is a normal merge, this can be done with `git merge --no-ff`. This takes the
history from both branches, preserves the history of both (it keeps track of where the history
diverges), and then it applies a *merge commit* on top. This is useful if you have a large
feature or change you want to merge into a branch, and you may want to *revert* the change 
in the future. Since you have a single commit that combines the two histories, you can
revert the singular "merge commit" that combines the two history, and you'll have reverted
all the changes made by that feature. When you own a repository on github, and you are taking
large changes, this is a nice way to merge a pull request, since in the future you can undo
the merge trivially.

The final, and my most commonly used strategy is the *rebase*. *Rebasing* is an operation
which rewrites history. When you rebase one branch on another, it takes all the differences
from one branch, and then tries to apply it on top of the history of another branch. If
a branch has commits A B C, and another has A D E, rebasing the second onto the first will
leave the second branch with the history A B C D E. This is in contrast too a merge commit,
which would leave you 
```
A   B   C
    D   E  F
```
where F is the *merge commit* that combines the two histories. The benefit a rebase leaves
you that a merge dosen't, is that a traditional merge can cause very messy history once 
more branches are merging together, and hence is harder to work with the history. Imagine
in the above case that the branch that started with A B C put in a commit that reverted B and C,
when you merge that branch again, you'd *again* be reverting those changes in another merge commit.
A lot of this history would be getting duplicated across two branches and can make it harder
to tell what's going on. a rebase in contrast will *always* only keep a single set of history,
as it'll always apply your changes on top of the existing ones. 

So what's the drawback of a rebase? For one, sometimes you *want* a merge commit, such as 
when you're adding a new feature you want to be able to remove easily. Another good 
reason is that *rebase* will modify history. Since you're adding history in middle of the
branch, this is an operation that will push everyone else who also has that same history
out of sync. If someone has ABC, and you have ABCDE, and then you rebase so now your history
is ABFCDE, then the person with ABC won't be able to trivially merge your branch. 

As a rule: *only rebase when no one else has your history.* In pratice, this means
I rebase from the main code when I am working with my own code. I then *merge* it
back upstream. 

As a workflow: Whenever there's a change in the code on github or wherever else - and
you want it locally, git rebase, and when you're accepting code on github from someone else, 
do git merge.

# Managing history

With git log, if you want to revert a change, you can find the *hash* of a commit
and then do `git revert` $HASH, and it'll make a commit that adds a commit on top
that undoes the commit with the given HASH. This works with git commits as well.

You can also manually modify history with git rebase -i, but I reccomend avoiding it
until you're comfortable with git.

# Working with remotes.

Remotes are just branches that aren't on your local machine. This includes github
or anywhere else. If you want to register a remote, you can do 
`git remote add (remote name) (remote url)`, and if you want to push a branch to
the remote, you can do `git push (remote name) (your branch)`. If you want to merge
a remote branch, you can either use `git fetch` (which updates your copy of the remote)
and do `git merge (remote name)/(remote branch) (your branch)` or you can take a shortcut 
and say `git pull --{merge strategy}`. For example, `git pull --ff-only (remote name) (your branch)`, 
`git pull --rebase (remote name) (your branch)` or `git pull --no-ff (remote name) (your branch)`. 

Again: in pratice, to sync with your remote, you probably want `git pull --rebase`. If you're working
through the github website, you probably want to do a merge commit. When in doubt, just follow the above.
To sync with the remote: git pull --rebase.
