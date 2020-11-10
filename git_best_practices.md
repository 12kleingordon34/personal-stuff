# Git

Git is becoming a critical component of technical workflows worldwide, providing a robust and simple way to track, version, share, and collaborate on code-related tasks.

This document aims to provide a simple "How To" guide to get started with Git. The first half of this section assumes no knowledge of Git, and outlines key steps on how to collaborate on a Git project, along with some links to additional sources of information.

In the second half of the document, some best practices are covered along with some justification for them. This latter section assumes familiarity with committing, pushing, pulling, cloning, ching `status` and `diff` and a basic git workflow. These section headers will be highlighted by an asterisk (*) and are not essential for a first reading.

## Docs

For reference, I highly recommend excellent (and readable) textbook [ProGit](https://git-scm.com/docs) which can be read for free online. When in doubt read the docs (or stack exchange). 

## Intro
Git differs from most Version Control Systems (VCSs)in the way it considers its data. It essentially captures snapshots of the filesystem at an instance (or version) in time, compared to alternative VCSs which capture the difference between multiple versions. This construction of Git (as if it were a mini filesystem) allows for a variety of powerful tooling which facilitate the development of code.

When stored inside a Git repository (or repo), the "status" of a file in falls within four main categories:
* `untracked` files are those within a Git repo that are not being tracked or versioned by Git
* `staged` files are files which have been marked by Git to be included in the next snapshot of the repo.
* `modified` files are tracked files that have been modified since their previous snapshot, but have not been staged.
* `commited` file versions are those which are tracked by the Git repository.

To check the status of files in your repository, run `git status`.

### Getting Started
Git comes preinstalled on Windows Subsystem for Linux (WSL) which can be downloaded from the _"Software Installation"_ facility on MyWorkplace. Before operating on a UCB Git project, make sure you have access to Azure. 

Once you have access to the [Azure DevOps](dev.azure.com/ucbalm), select your desired project. Once selected, select the `Repos` option on the side bar. Once a specific Repo has been selected, click the `Clone` button towards to top left of the page, and copy the `HTTPS` location to your terminal and run the following command:
```
git clone https://ucbalm@dev.azure.com/ucbalm/<UCB%Project>/_git/<Repository>
```

`git clone` takes the most recent snapshot of a repository from the remote (cloud based) Git repository and copies it to your local machine. 

## Workflow

In this section the basic workflow of a task will be covered along with the requisite commands.

Before making changes to a repo, checkout to the master branch and make sure it is up-to-date:
```
git checkout master
git pull -r
```
When starting a new body of work, always start a new branch. With our current approach to git, code on master is required to be complete and finished. Commits should **never** be pushed directly to the remote master branch on repos with multiple collaborators.
```
# Creates a new branch rooted on the current HEAD commit
git checkout -b <branch-name>
```
The branch isolates work from the master and allows teams to develop features efficiently. The standard convention is to use hyphens (as opposed to underscores for file names and sub-directories), and names should be succinct and clear.

After there are a few commits on the new branch, a remote copy of the work needs to be created. This allows github's Pull Request (PR) management tools to be used to merge the work into master on completion, and acts as a backup. The branch created previously is currently a local branch, it has no corresponding branch in the remote repo to track and push to. To push commits t˜o a new remote branch:

```
git push -u origin <branch-name>
```

The `-u` flag is shorthand for `set-upstream`, which is git terminology for the remote copy of the work that is updated from the local branch and acts as the instance any other copies are made from. (The remote branch `origin/branch-name` is the upstream branch of your local branch `branch-name`, and `branch-name` is tracking `origin/branch-name`) To view the state of local branch tracking, display verbose branch information with `git branch -vv`.

If the branch has only one developer, progress is simple. When more that one developer alters remote branches, complications arise that are dealt with later (Merging and Rebasing). Once the work is complete for a feature, a pull-request can be created using the github interface, conflicts resolved and finally the work merged, which will add your commits into master while creating an extra merge commit, and the branch deleted. Occasionally, merging into master will be blocked by _merge conflicts_, which will be discussed after detailing merging and rebasing.

### Making Changes
Consider the case where you have two files (`test_A.R` and `test_B.R`) you wish to add to a Git repo. There are multiple ways to add files to the staging area; below we propose four alternatives:
```
git add test_A.R
git add test_A.R test_B.R
git add .
git add -u
```
The first two commands add specific files to the staging area by directly calling their file names. The third command adds all files within the current working directory (`./`) to staging, whereas the fourth adds all modified files to the staging area. The latter is particularly useful if working with many modified scripts in a repository that includes additional files (such as datasets or `.pdf`s) that may not be committed to the repository.

### Commiting Changes

To commit these staged files to the repository use the command `git commit`. Commits are accompanied by messages which should capture high level information about what changes have been made in that specific commit. If the message is no longer than 50 - 60 characters, you can add the commend inline:
```
git commit -m "<my-customised-commit-message>"
```
If a commit contains a large number of changes, you can write the commit summary in an in-terminal text editor (such as vim or nano) using `git commit`. The customary structure for these messages involve a brief summary, followed by bullet points:
```
This commit cured the COVID epidemic

* Change 1
* Change 2
...
```

As tempting as it is to commit via `git commit -m 'update code'` every time, you will regret it in the long run. There are plenty of resources [online](https://chris.beams.io/posts/git-commit/) which link below in the links section on commit messages goes into lots of detail on best format, but the important bits are to be clear and exact, have a title and describe what was done and why. This can be a lifesaver when debugging, and is useful as the code is then in some ways self documenting.


### Pushing Changes
Now that you have commited your changes locally, you may want to push them to the remote repository. To push the very first changes of a branch you have just created, run
```
git push -u origin <branch-name>
```
where the `-u` command indicates that we are pushing "upstream" to the remote as mentioned above. All subsequent pushes can be made with the command `git push`.

### Pulling Changes
To update your local repository with all the updates your repo collaborators have pushed to remote,run `git pull -r` (or `git pull --rebase`). This command fetches all code changes to the repository and modifies your Git commit history accordingly by rebasing. Omitting the `--rebase` keyword merges the commits rather than rebasing them, but this can lead to messier commit histories. Rebasing should almost always be the preferred choice. (See later for distinction between merging and rebasing).

## \* Merging and Rebasing 

When two or more people are working on the same branch, each person will end up with their own version of the branch, distinct from the remote version, as they work on commits independently. At some point, one person will push their changes to the remote, at which point the commit history on the remote is different from the local history for all people aside from the initial pusher. For those other people to commit any changes to the remote, they will need to incorporate the new changes into their local copy by `git pull`ing and then push their own changes. This resolving of the differing histories introduces complication, and there are two strategies git can use for this, _merging_ and _rebasing_. Both versions have their own benefits and pitfalls. A more detailed discussion of the merits of each is covered in the Atlassian tutorial below, this is more bare bones.

Consider the following simplified mockup, Alice and Bob both had up to date branches at commit C, but Alice has pushed commits D, E to the remote before Bob could push F, G, so Bob now needs to incorporate D into his branch before pushing E:


```
    0 E         0 E         0 G
    |           |           |
    0 D         0 D         0 F
    |           |           |
    0 C         0 C         0 C
    |           |           |
    0 B         0 B         0 B
    |           |           |
    0 A         0 A         0 A
  Alice       Remote       Bob
```

The first way, merging takes D and applies it on top of F. This is what occurs if the commit is fetched using only `git pull`, and results in a commit history that looks like:
```
    0 Merge Commit
   / \
  /   \
 0 E   0 G
 |     |
 0 D   0 F
  \   /
   \ /
    0 C
    |
    0 B
    |
    0 A
  Merged
```

The merge commit will have some description along the lines of 'Merge branch-name into branch-name'. A cleaner history can be achieved using `git pull --rebase`. Rebase, in git terminology for situations like these, refers to resolving these history differences by rewinding local changes, applying the remote changes and then sequentially reapplying the local commits on top of the remote commits. Following a rebase operation, the commit history will look as follows.
```
    0 F
    |
    0 E
    |
    0 D
    |
    0 C
    |
    0 B
    |
    0 A
```
This is much more pleasant and clear, so always use `git pull --rebase`. The issues with rebase arise with resolving merge conflicts, which is where two commits (i.e. D and G) alter the same point of code that git cannot safely resolve. With the merge operation, the conflicts only need to be fixed once, while resolving a rebase merge conflict is harder, and is dealt with in detail in a later section.

## \* Correcting errors and reverting 

A quick aside here, but in the event you decide that something committed is wrong, git is designed to fix it by adding a fix commit, rather than removing a past commit. Git is designed to support and linear history of all changes and modifications to the code base, and the error forms a part of that history. Normally, an error will be fixed wither by or as part of a subsequent commit, which will suffice. If you absolutely decide a commit needs to go, `git revert` (in the docs, as a note, can also be done for pull requests) the commit. This will add an 'anti' commit to the branch that removes everything the reverted commit added. In the event an entire set of changes, consisting of several commits, needs to be reverted, git has this functionality, so revert them all rather than create a new branch.

## \* Merge conflicts

Along with discussing in detail resolving a merge conflict via a rebase, this section will cover when conflicts arise between your remote branch and the master branch of a repo. This will happen somewhat infrequently, when another task separate from your own alters the same piece of code, but will prevent PRs from being merged in github. Similar to the branch conflicts, these can be resolved by merging master onto your branch or rebasing master onto your branch. The situation below shows the final structures following several merges of master onto a remote branch (it's good practice to keep your branch up to date with changes on master, and means you resolve merge conflicts sooner and in smaller chunks).

```
        O                            O
        | \                          | \
        |  \                         |  \
        O + O                        |   O
        |   |                        |   |
        |   |                        |   |
        O   O                        |   O
        |   |                        |   |
        |   |                        |   |
        O   O  Merge Commit          |   O
        |  /|                        |   |
        | / |                        |   |
        O   O                        |   O
        |   |                        |   |
        |   |                        |   |
        O   O                        |   O *
        |   |                        |  /
        |   |                        | /
        O   O  Merge Commit          O +
        |  /|                        |
        | / |                        |
        O   O                        O
        |   |                        |
        |   |                        |
        O   O                        O
        |   |                        |
        |   |                        |
        O   O *                       O
        |  /                         |
        | /                          |
        O §                          O §
     Merge                         Rebase
```

With the merge, the remote branch gains several merge commits, and the commit history gets complicated. The rebase results in a much cleaner structure. To be clear, both situations above are designed to be comparable, with development of the feature branch on the right, and additional commits added to master which need to be incorporated.

To rebase master onto your branch, there is a procedure you can follow to do so safely.
```
# Make sure your feature branch is up to date
git checkout {branch-name}
git pull --rebase
# Make sure local master is up to date
git checkout master
git pull --rebase
# Rebase master onto your branch
git checkout {branch-name}
git rebase master
{resolve all merge conflicts}
# Overwrite remote history
git push -f origin {branch-name}
```

There are a few things to discuss in this procedure. The first two stages are safety stages, just to make sure everything is in place when you begin the rebase. The rebase stage, with the merge conflict resolution, can be painful, and is covered below. The final command is interesting as it is the only point in a regular git workflow where using `-f` or `force` is required/justifiable.

Git stores code as a series of historied changes. Each commit points to a previous commit, and details how to modify the code at the previous commit point to reach its own commit point. As such, the concept of history is vital to git, as a commit only makes sense pointing to a specific commit. When you rebase, rewinding the branch commits, applying master and re applying the branch commits, history is changed. In the diagram above, commit `*` on the branch changes from being descended from commit `§` to commit `+`. This change of history is why the force push is required.

After a rebase operation, running `git status` locally will say something along the lines of the `commit histories have diverged`. The remote history is no longer comparable to the local, and it cannot be made comparable by adding commits on the head, as the overall structure has changed. This is why the `force` is needed, the local rebased version is the new desired updated version, and we need to overwrite the remote, changing history as we do so.

## \* Resolving merge conflicts in rebases.

When a merge conflict arises, git will output instruction on how to resolve it, edit the files, add them and commit. With a rebase, the situation can get more complicated. As discussed, the rebase works by reapplying every one of your branch commits one-by-one on top of master. As such there is the possibility of a conflict at each commit, which is where the pain can come with rebases.

The basic step of resolving a rebase merge conflict is to edit the conflicting file, resolving the conflict, `git add {file}` to signify resolution and run `git rebase --continue`. The git output is useful here as it contains these instructions. The tricky part is how to resolve it safely. It can be tempting to resolve it at each stage to be the most up to date version, if you've been iterating on a procedure. However, do not do this. The rebase works by applying the commits sequentially, and as mentioned before, commits are records of change, not code snippits in their own right. If you change mid rebase to the most up to date version, the next rebasing commit, which details a change from two in development stages, can no longer be applied, and will create another merge conflict.

The short version for resolving each merge conflict is, go with the lower section of the conflict box:
```
<<<<
Upper
=====
lower
>>>>>
```
which represents the branch commit you are applying, but don't remove any functionality that was added in the upper section or overwrite it. Most merge conflicts typically require a bit more nuance however.

The best way to avoid overwhelming merge conflicts is ensuring that you regularly rebase on master to keep your work up to date with the rest of the team's.

### \* Notebooks

Notebooks contain a bunch of extra JSON XML formatting and all sorts, and almost always cause merge conflicts when two people work in parallel on a single notebook. Resolving the merge conflict is frequently desperately tedious and tricky if not effectively impossible, so the normal best solution is to select one complete version of the file from either side of the merge.

Considering a case as described above, where a rebase from master is being carried out on a branch and the rebase has paused for a merge conflict:

```
git checkout --ours {filename_from_master_to_be_kept}
git checkout --theirs {filename_from_branch_to_be_kept}
```

The use of 'ours' and 'theirs' can be a bit counter intuitive. The short version is above, if you are rebasing as described, `ours` is the master version and `theirs` is the branch version.

To go into more explanation (this section could just confuse, feel free to skip), when rebasing git is applying the commits from the branch on top of master in order, so at each step the branch is _on_ master (ours) and is taking a commit from the branch (theirs) to be applied. This is only the case when **rebasing**, when `merging`, the situation is reversed.

## Useful Git Tools

* `git stash`: stores all current uncommitted changes in a temporary storage location, useful if you need to pull the latest changes or otherwise change git state but don't want to commit your current changes. Recover the work via `git stash apply` or `git stash pop`. All stashes are preserved (viewable with `git stash list`) so multiple stashes from different branches are accessible, and are applied by `git stash apply stash@{n}`. The latter command `git stash pop` removes the stash from the list, which can help keep it uncluttered.
* `git checkout {commit_hash}`, temporarily move the repo state at a particular commit, useful for debugging, return by `git checkout {branch-name}`, only the first few characters of the hash are required.
* `git diff {commit1} {commit2}`, compare two points, useful for debugging.
* `git log`: show current commit history
* `git reflog`: show git command history, can be used to revert a selection of commands.

## Links

Commit Messages: https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
Merging vs Rebasing: https://www.atlassian.com/git/tutorials/merging-vs-rebasing

## Specific examples

This section details issues I've seen crop up a few times.

### Deletion of an 'unneeded' file

This is an odd case where a file is edited and then the changes in that file are deemed not needed. This could be that two people were working on a single file in separate PRs, of which one was merged, or a person starts editing an existing file and then decides against the changes.

In both of these cases, the changes to the file in question are no longer required, so it can be tempting to delete the file. In the latter case this is less likely: a more obvious result is to reset the file to master, however in the first (and this problem is accentuated if the PRs are being worked on at the same time), given Github will flag all the changes to the file, deletion seems an easy way to avoid them. In some cases deletion has been done unintentionally.

However, because of the linear nature of git, any deletion which is subsequently stored in a commit is an action. So considering file A, which is featured in PRs 1 and 2, if PR 1 is merged and file A is deleted in PR 2 (as the most up to date version is in master), when PR 2 is merged, the file will be deleted from master.

The 'solution' to this isn't really a solution: as there are two avenues of development, there is a conflict that needs to be resolved. If the file changes in PR 2 are no longer required, retain the file, rebase on master and select the master version, rather than deleting it.
