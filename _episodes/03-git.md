# GIT QUICK INTRO  
_Based on chapter 5 from Vince Buffalo's book "Bioinformatics Data Skills"  
Comics are from https://imgs.xkcd.com/comics_

<img src="https://imgs.xkcd.com/comics/git.png" width="270" align="right">

Why GIT? 
--------
- Allows You to Keep Snapshots of Your Project
- Helps You Keep Track of Important Changes to Code
- Helps Keep Software Organized and Available After People Leave

## Installing Git
If you’re on OS X or Linux, git should be already installed (use Homebrew (_e.g._, `brew install git`) or apt-get (_e.g._, `apt-get install git`), respectively, if you need to update). On Windows, install [Git for Windows](http://gitforwindows.org/). Finally, you can always visit the [Git website](https://git-scm.com/) for both source code and executable versions of Git as well as multiple learning resources.

## Basic Git

### Git Setup
Because Git is meant to help with collaborative editing of files, you need to tell Git who you are and what your email address is. To do this, use:  
`git config --global user.name "Sewall Wright"`
`git config --global user.email "swright@adaptivelandscape.org"`  
Make sure to use your own name and email, or course.

Another useful Git setting to enable now is terminal colors:  
`git config --global color.ui true`

### Creating Repositories: git init and git clone

To get started with Git, we first need to create a Git repository. There are two primary ways to it: by initializing one from an existing directory, or cloning a repository that exists elsewhere. We will start with the first :)

First, let's create a new directory in your home directory (or your preferred location):  
`mkdir -p ~/EEOB563/labs`  

Now, go to this directory and initialize it as a git repository:  
`cd ~/EEOB563/labs`  
`git init`.

### Tracking Files in Git: git add and git status I

Although you’ve initialized the EEOB563/labs as a Git repository, Git doesn’t automati‐ cally begin tracking every file in this directory. Rather, you need to tell Git which files to track using the subcommand `git add`. 

But before doing it, let's check the status of our directory:  
`git status`  
The command will tell you that we are on branch master, that there were no previous commits ("Initial commit") and that there is nothing to commit and no files in the direcotry.

Let's create a new file:
`echo # My brand new Git repository" > README.md`  
and check the status again:
`git status`

We can see now that we have an untracked file.  We use `git add` command to tell Git to track it:  
`git add README.md`  
Now Git is tracking the file and if we made a commit now, a snapshot of the file would be created.

### Staging Files in Git: git add and git status II

Here comes a confusing part: `git add` command is used both to track files and to stage changes to be included in the next commit. But to make it confusing, we have to add another file to our directory:  
`touch empty`  
Now, modify the README.md file (e.g., `echo '**Git** can be confusing!' >> README.md`) and make a commit. If you compare your committed file and your current file (`git diff`) you'll see that the previous version of the file has been committed! To include the latest changes, you'll need to explicitly stage them using `git add` again. But we have to learn how to make a commit first!

### Taking a Snapshot of Your Project: git commit

<img src="https://imgs.xkcd.com/comics/git_commit.png" width="400" align="right"> 

Modify the file as explained above and commit the changes with the commit command:
`git commit -m "First commit"`  
Notice, that we include a message in our commits.  If you try to skip it, a default text editor (likely vi) will open. **Also notice that this message should be informative!**  
`git diff` shows you the difference between the files in your working directory and what’s been staged. If none of your changes have been staged, `git diff` shows the difference between your last commit and the current versions of files. 

Earlier, we used `git add` to stage the changes. There’s an easy way to stage all tracked files’ changes and commit them in one command: `git commit -a -m "your commit message"`. The option -a tells `git commit` to automatically stage all modified tracked files in this commit.

### Seeing File Differences and Commit History: git diff and git log
We already saw `git diff` above (there are obviously, many more options (try `git help diff`).

We can use `git log` to visualize the chain of commits. The strange looking mix of numbers and characters after commit is a SHA-1 checksum. SHA-1 hashes act as a unique ID for each commit in your repository. You can always refer to a commit by its SHA-1 hash. 

### Moving and Removing Files: git mv and git rm
When Git tracks your files, it wants to be in charge. Using the command `mv` to move a tracked file will confuse Git. The same applies when you remove a file with `rm`. To move or remove tracked files in Git, we need to use Git’s version of mv and rm: `git mv` and `git rm`.

### Telling Git What to Ignore: .gitignore
You may have noticed that git status keeps listing which files are not tracked. As the number of files starts to increase this long list will become a burden. To ignore some files (or subdirectories), create and edit the file .gitignore in your repository directory. For example, we can add our empty file there: `echo "empty" > .gitignore` or a set of fastq file you may have in your data directory: `echo "data/seqs/*.fastq" >> .gitignore`. Here are some expamples of files you want to ignore:  
- large files  
- intermediate files  
- temporary files  
GitHub maintains a useful repository of .gitignore suggestions: [https://github.com/github/gitignore/tree/master/Global](https://github.com/github/gitignore/tree/master/Global).
You can create a global .gitignore file in ~/.gitignore_global and then configure Git to use this with the following:
`git config --global core.excludesfile ~/.gitignore_global`

### Undoing a Stage and Getting Files from the Past: git reset and git checkout
Recall that one nice feature of Git is that you don’t have to include messy changes in a commit—just don’t stage these files. If you accidentally stage a messy file for a commit with `git add`, you can unstage it with `git reset`.  
Similarly, if you accidentally overwrote your current version of README.md by using > instead of >> , you can restore this file by checking out the version in our last commit:  
`git checkout -- README.md`.  
But beware: restoring a file this way erases all changes made to that file since the last commit! 

## Collaborating with Git: Git Remotes, git push, and git pull

The basis of sharing commits in Git is the idea of a remote repository, which is a version of your repository hosted elsewhere. Collaborating with Git first requires we configure our local repository to work with our remote repositories. Then, we can retrieve commits from a remote repository (a pull) and send commits to a remote repository (a push).

### Creating a Shared Central Repository with GitHub

The first step of for us is to create a shared central repository, which is what you and your collaborator(s) share commits through. Here we will use [GitHub](http://github.com/), a web-based Git repository hosting service. [Bitbucket](http://bitbucket.org/) and [GitLab](http://git.linux.iastate.edu) server hosted by ISU are other Git repository hosting service you and your collaborators could use (both provide free private repositories). 

Navigate to [http://github.com](http://github.com) and sign up for an account (if you don't have one). After your account is set up, you’ll be brought to the GitHub home page, were you'll find a link to create a new repository. Follow it and provide a repository name. You’ll have the choice to initialize with a README.md file (GitHub plays well with Markdown), a .gitignore file, and a license (to license your software project). For now, just create a repository named **eeob563**. After you’ve clicked the “Create repository” button, GitHub will forward you to an empty repository page – the public frontend of your project.

### Authenticating with Git Remotes

GitHub uses SSH keys to authenticate you. SSH keys prevent you from having to enter a password each time you push or pull from your remote repository. We create SSH keys on our computer (or any other computer you are using) with the `ssh-keygen command`. It creates a private key at ~/.ssh/id_rsa and a public key at ~/.ssh/id_rsa.pub. You have an option to use a password, but you don't need to.
**However, it’s very important that you note the difference between your public and private keys: you can distribute your public key to other servers, but your private key must be never shared.** 
Navigate to your account settings on GitHub, and in account settings, find the SSH keys tab. Here, you can enter your public SSH key (remember, don’t share your private key!) by using ``cat ~/.ssh/id_rsa.pub` to view it, copy it to your clipboard, and paste it into GitHub’s form.

### Connecting with Git Remotes: git remote

Let’s configure our local repository to use the GitHub repository we’ve just created as a remote repository. We can do this with:  
`git remote add origin git@github.com:your_name/eeob563`

In this command, we specify not only the address of our Git repository at github , but also a name for it: origin. By convention, origin is the name of your primary remote repository.

Now if you enter `git remote -v` (the -v stands for verbose), you see that our local Git repository knows about the remote repository:
`git remote -v`  
`origin git@github.com:username/zmays-snps.git (fetch)`   
`origin git@github.com:username/zmays-snps.git (push)`  

You can have multiple remote repositories. If you ever need to delete an unused remote repository, you can with `git remote rm <repository-name>`.

### Pushing Commits to a Remote Repository with git push

Let’s push our initial commits from zmays-snps into our remote repository on Git‐ Hub. The subcommand we use here is git push <remote-name> <branch>. We’ll talk more about using branches later, but recall from “Tracking Files in Git: git add and git status Part I” on page 72 that our default branch name is master. Thus, to push our zmays-snps repository’s commits, we do this:
`$ git push origin master`

### Pulling Commits from a Remote Repository with git pull

To pull the commits from a remote repository, you use the `git pull origin master` command. Of course pulling them makes sense if somebody else is contributing to the repository.  Examples can be your professor updating the course GitHub, you yourself working on two different computers, or several collaborators working on a joint project. In fact, it's a good practice to always `pull` before you `push`!

### Merge conflicts

Occasionally, you’ll pull in commits and Git will warn you there’s a merge conflict. Resolving merge conflicts can be a bit tricky, but the strategy to solve them is always the same:

1. Use `git status` to find the conflicting file(s).
2. Open and edit those files manually to a version that fixes the conflict.
3. Use `git add` to tell Git that you’ve resolved the conflict in a particular file.
4. Once all conflicts are resolved, use `git status` to check that all changes are staged. Then, commit the resolved versions of the conflicting file(s). 

It’s also wise to immediately push this merge commit, so your collaborators see that you’ve resolved a conflict and can continue their work on this new version accordingly.

For complex merge conflicts, you may want to use a merge tool. Merge tools help visualize merge conflicts by placing both versions side by side, and pointing out what’s different (rather than using Git’s inline notation that uses inequality and equal signs). Some commonly used merge tools include Meld and Kdiff.

### More GitHub Workflows: Forking and Pull Requests
While the shared central repository workflow is the easiest way to get started collaborating with Git, 
GitHub suggests a slightly different workflow based on forking repositories. When you visit a repository 
owned by another user on GitHub, you’ll see a “fork” link. Forking is an entirely GitHub concept—it is 
not part of Git itself. By forking another person’s GitHub repository, you’re copying their repository 
to your own GitHub account. You can then clone your forked version and continue development in your own 
repository. Changes you push from your local version to your remote repository do not interfere with the 
main project. If you decide that you’ve made changes you want to share with the main repository, you can 
request that your commits are pulled using a pull request (another feature of GitHub).
This is the workflow GitHub is designed around, and it works very well with projects with many collaborators. 
Development primarily takes place in contributors’ own repositories. A developer’s contributions are only 
incorporated into the main project when pulled in. This is in contrast to a shared central repository workflow, 
where collaborators can push their commits to the main project at their will. As a result, lead developers 
can carefully control what commits are added to the project, which prevents the hassle of new changes breaking 
functionality or creating bugs.

## Using Git to Make Life Easier: Working with Past Commits
So far in this chapter we’ve created commits in our local repository and shared these commits with our collaborators. But our commit history allows us to do much more than collaboration—we can compare different versions of files, retrieve past versions, and tag certain commits with helpful messages.
After this point, the material in this chapter becomes a bit more advanced. Readers can skip ahead to Chapter 6 without a loss of continuity. If you do skip ahead, book‐ mark this section, as it contains many tricks used to get out of trouble (e.g., restoring files, stashing your working changes, finding bugs by comparing versions, and editing and undoing commits). In the final section, we’ll also cover branching, which is a more advanced Git workflow—but one that can make your life easier.

### Getting Files from the Past: git checkout
Anything in Git that’s been committed is easy to recover. Suppose you accidentally overwrite your current version of README.md by using > instead of >>. You see this change with git status:
$ echo "Added an accidental line" > README.md $ cat README.md
Added an accidental line
$ git status
# On branch master
# Changes not staged for commit:
# (use "git add <file>..." to update what will be committed)
# (use "git checkout -- <file>..." to discard changes in working directory)
#
# modified: README.md
#
no changes added to commit (use "git add" and/or "git commit -a")
This mishap accidentally wiped out the previous contents of README.md! However, we can restore this file by checking out the version in our last commit (the commit HEAD points to) with the command git checkout -- <file>. Note that you don’t need to remember this command, as it’s included in git status messages. Let’s restore README.md:
$ git checkout -- README.md
$ cat README.md
Zea Mays SNP Calling Project
Project started 2013-01-03
Samples expected from sequencing core 2013-01-10
    We downloaded the B72 reference genome (refgen3) on 2013-01-04 from
    http://maizegdb.org into `/share/data/refgen3/`.
But beware: restoring a file this way erases all changes made to that file since the last commit! If you’re curious, the cryptic -- indicates to Git that you’re checking out a file, not a branch (git checkout is also used to check out branches; commands with multiple uses are common in Git).
By default, git checkout restores the file version from HEAD. However, git checkout can restore any arbitrary version from commit history. For example, suppose we want to restore the version of README.md one commit before HEAD. The past three com‐ mits from our history looks like this (using some options to make git log more con‐ cise):
$ git log --pretty=oneline --abbrev-commit -n 3 20041ab resolved merge conflict in README.md 08ccd3b added reference download date
dafce75 added ref genome download date and link
Thus, we want to restore README.md to the version from commit 08ccd3b. These SHA-1 IDs (even the abbreviated one shown here) function as absolute references to your commits (similar to absolute paths in Unix like /some/dir/path/ le.txt). We canalways refer to a specific commit by its SHA-1 ID. So, to restore README.md to the version from commit 08ccd3b, we use:
$ git checkout 08ccd3b -- README.md $ cat README.md
Zea Mays SNP Calling Project Project started 2013-01-03
    Samples expected from sequencing core 2013-01-10
    We downloaded refgen3 on 2013-01-04.
If we restore to get the most recent commit’s version, we could use:
$ git checkout 20041ab -- README.md
$ git status
# On branch master
nothing to commit, working directory clean
Note that after checking out the latest version of the README.md file from commit 20041ab, nothing has effectively changed in our working directory; you can verify this using git status.

### Stashing Your Changes: git stash
One very useful Git subcommand is git stash, which saves any working changes you’ve made since the last commit and restores your repository to the version at HEAD. You can then reapply these saved changes later. git stash is handy when we want to save our messy, partial progress before operations that are best performed with a clean working directory—for example, git pull or branching (more on branching later).
Let’s practice using git stash by first adding a line to README.md:
$ echo "\\nAdapter file: adapters.fa" >> README.md $ git status
# On branch master
# Changes not staged for commit:
# (use "git add <file>..." to update what will be committed)
# (use "git checkout -- <file>..." to discard changes in working directory)
#
# modified: README.md
#
no changes added to commit (use "git add" and/or "git commit -a")
Then, let’s stash this change using git stash:
$ git stash
Saved working directory and index state WIP on master: 20041ab resolved merge conflict in README.md
HEAD is now at 20041ab resolved merge conflict in README.md

Stashing our working changes sets our directory to the same state it was in at the last commit; now our project directory is clean.
To reapply the changes we stashed, use git stash pop:
$ git stash pop
# On branch master
# Changes not staged for commit:
# (use "git add <file>..." to update what will be committed)
# (use "git checkout -- <file>..." to discard changes in working # directory)
#
# modified: README.md
#
no changes added to commit (use "git add" and/or "git commit -a") Dropped refs/stash@{0} (785dad46104116610d5840b317f05465a5f07c8b)
Note that the changes stored with git stash are not committed; git stash is a sepa‐ rate way to store changes outside of your commit history. If you start using git stash a lot in your work, check out other useful stash subcommands like git stash apply and git stash list.
More git di : Comparing Commits and Files
Earlier, we used git diff to look at the difference between our working directory and our staging area. But git diff has many other uses and features; in this section, we’ll look at how we can use git diff to compare our current working tree to other commits.
One use for git diff is to compare the difference between two arbitrary commits. For example, if we wanted to compare what we have now (at HEAD) to commit dafce75:
$ git diff dafce75
diff --git a/README.md b/README.md
index 016ed0c..9491359 100644
--- a/README.md
+++ b/README.md
@@ -3,5 +3,7 @@ Project started 2013-01-03
     Samples expected from sequencing core 2013-01-10
    -Maize reference genome version: refgen3, downloaded 2013-01-04 from
    +We downloaded the B72 reference genome (refgen3) on 2013-01-04 from
     http://maizegdb.org into `/share/data/refgen3/`.
    +
    +Adapter file: adapters.fa
    
> ## Specifying Revisions Relative to HEAD
Like writing out absolute paths in Unix, referring to commits by their full SHA-1 ID is tedious. While we can reduce typing by using the abbreviated commits (the first seven characters of the full SHA-1 ID), there’s an easier way: relative ancestry refer‐ ences. Similar to using relative paths in Unix like ./ and ../, Git allows you to specify commits relative to HEAD (or any other commit, with SHA-1 IDs).
The caret notation (^) represents the parent commit of a commit. For example, to refer to the parent of the most recent commit on the current branch (HEAD), we’d use HEAD^ (commit 08ccd3b in our examples).
Similarly, if we’d wanted to refer to our parent’s parent commit (dafce75 in our exam‐ ple), we use HEAD^^. Our example repository doesn’t have enough commits to refer to the parent of this commit, but if it did, we could use HEAD^^^. At a certain point, using this notation is no easier than copying and pasting a SHA-1, so a succinct alternative syntax exists: git HEAD~<n>, where <n> is the number of commits back in the ances‐ try of HEAD (including the last one). Using this notation, HEAD^^ is the same as HEAD~2.
Specifying revisions becomes more complicated with merge commits, as these have two parents. Git has an elaborate language to specify these commits. For a full specifi‐ cation, enter git rev-parse --help and see the “Specifying Revisions” section of this manual page.


Using git diff, we can also view all changes made to a file between two commits. To do this, specify both commits and the file’s path as arguments (e.g., git diff <com mit> <commit> <path>). For example, to compare our version of README.md across commits 269aa09 and 46f0781, we could use either:
$ git diff 46f0781 269aa09 README.md #or
$ git diff HEAD~3 HEAD~2 README.md
This second command utilizes the relative ancestry references explained in “Specify‐ ing Revisions Relative to HEAD” on page 101.
How does this help? Git’s ability to compare the changes between two commits allows you to find where and how a bug entered your code. For example, if a modified script produces different results from an earlier version, you can use git diff to see exactly which lines differ across versions. Git also has a tool called git bisect to help devel‐ opers find where exactly bugs entered their commit history. git bisect is out of the scope of this chapter, but there are some good examples in git bisect --help.

### Undoing and Editing Commits: git commit --amend
At some point, you’re bound to accidentally commit changes you didn’t mean to or make an embarrassing typo in a commit message. For example, suppose we were to make a mistake in a commit message:
$ git commit -a -m "added adpters file to readme" [master f4993e3] added adpters file to readme
     1 file changed, 2 insertions(+)
We could easily amend our commit with:
$ git commit --amend
git commit --amend opens up your last commit message in your default text editor, allowing you to edit it. Amending commits isn’t limited to just changing the commit message though. You can make changes to your file, stage them, and then amend these staged changes with git commit --amend. In general, unless you’ve made a mistake, it’s best to just use separate commits.
It’s also possible to undo commits using either git revert or the more advanced git reset (which if used improperly can lead to data loss). These are more advanced top‐ ics that we don’t have space to cover in this chapter, but I’ve listed some resources on this issue in this chapter’s README file on GitHub.

## Working with Branches
Our final topic is probably Git’s greatest feature: branching. If you’re feeling over‐ whelmed so far by Git (I certainly did when I first learned it), you can move forward to Chapter 6 and work through this section later.
Branching is much easier with Git than in other version control systems—in fact, I switched to Git after growing frustrated with another version control system’s branching model. Git’s branches are virtual, meaning that branching doesn’t require actually copying files in your repository. You can create, merge, and share branches effortlessly with Git. Here are some examples of how branches can help you in your bioinformatics work:
• Branches allow you to experiment in your project without the risk of adversely affecting the main branch, master. For example, if in the middle of a variant call‐ ing project you want to experiment with a new pipeline, you can create a new branch and implement the changes there. If the new variant caller doesn’t work out, you can easily switch back to the master branch—it will be unaffected by your experiment.
• If you’re developing software, branches allow you to develop new features or bug fixes without affecting the working production version, the master branch. Once the feature or bug fix is complete, you can merge it into the master branch, incor‐ porating the change into your production version.
• Similarly, branches simplify working collaboratively on repositories. Each collab‐ orator can work on their own separate branches, which prevents disrupting the master branch for other collaborators. When a collaborator’s changes are ready to be shared, they can be merged into the master branch.
Creating and Working with Branches: git branch and git checkout
As a simple example, let’s create a new branch called readme-changes. Suppose we want to make some edits to README.md, but we’re not sure these changes are ready to be incorporated into the main branch, master.
To create a Git branch, we use git branch <branchname>. When called without any arguments, git branch lists all branches. Let’s create the readme-changes branch and check that it exists:
$ git branch readme-changes $ git branch
* master
      readme-changes
The asterisk next to master is there to indicate that this is the branch we’re currently on. To switch to the readme-changes branch, use git checkout readme-changes:
$ git checkout readme-changes Switched to branch 'readme-changes' $ git branch
      master
    * readme-changes
Notice now that the asterisk is next to readme-changes, indicating this is our current branch. Now, suppose we edit our README.md section extensively, like so:
    # Zea Mays SNP Calling Project
    Project started 2013-01-03.
    ## Samples
    Samples downloaded 2013-01-11 into `data/seqs`:
            data/seqs/zmaysA_R1.fastq
            data/seqs/zmaysA_R2.fastq
            data/seqs/zmaysB_R1.fastq
            data/seqs/zmaysB_R2.fastq
            data/seqs/zmaysC_R1.fastq
            data/seqs/zmaysC_R2.fastq
## Reference

    We downloaded the B72 reference genome (refgen3) on 2013-01-04 from
    http://maizegdb.org into `/share/data/refgen3/`.
Now if we commit these changes, our commit is added to the readme-changes branch. We can verify this by switching back to the master branch and seeing that this com‐ mit doesn’t exist:
$ git commit -a -m "reformatted readme, added sample info" [readme-changes 6e680b6] reformatted readme, added sample info
     1 file changed, 12 insertions(+), 3 deletions(-)
$ git log --abbrev-commit --pretty=oneline -n 3 6e680b6 reformatted readme, added sample info 20041ab resolved merge conflict in README.md 08ccd3b added reference download date
$ git checkout master
Switched to branch 'master'
$ git log --abbrev-commit --pretty=oneline -n 3 20041ab resolved merge conflict in README.md 08ccd3b added reference download date
dafce75 added ref genome download date and link
Our commit, made on the branch readme-changes. The commit we just made (6e680b6).
Switching back to our master branch.
Our last commit on master is 20041ab. Our changes to README.md are only on the readme-changes branch, and when we switch back to master, Git swaps our files out to those versions on that branch.
Back on the master branch, suppose we add the adapters.fa file, and commit this change:
$ git branch * master
      readme-changes
$ echo ">adapter-1\\nGATGATCATTCAGCGACTACGATCG" >> adapters.fa $ git add adapters.fa
$ git commit -a -m "added adapters file"
[master dd57e33] added adapters file
     1 file changed, 2 insertions(+)
     create mode 100644 adapters.fa
Now, both branches have new commits. This situation looks like Figure 5-6.

Another way to visualize this is with git log. We’ll use the --branches option to specify we want to see all branches, and -n 2 to only see these last commits:
$ git log --abbrev-commit --pretty=oneline --graph --branches -n2 * dd57e33 added adapters file
| * 6e680b6 reformatted readme, added sample info
|/
Merging Branches: git merge
With our two branches diverged, we now want to merge them together. The strategy to merge two branches is simple. First, use git checkout to switch to the branch we want to merge the other branch into. Then, use git merge <otherbranch> to merge the other branch into the current branch. In our example, we want to merge the readme-changes branch into master, so we switch to master first. Then we use:
$ git merge readme-changes
Merge made by the 'recursive' strategy.
     README.md | 15 ++++++++++++---
     1 file changed, 12 insertions(+), 3 deletions(-)
There wasn’t a merge conflict, so git merge opens our text editor and has us write a merge commit message. Once again, let’s use git log to see this:
$ git log --abbrev-commit --pretty=oneline --graph --branches -n 3 * e9a81b9 Merge branch 'readme-changes'
|\
| * 6e680b6 reformatted readme, added sample info

    * | dd57e33 added adapters file
    |/
Bear in mind that merge conflicts can occur when merging branches. In fact, the merge conflict we encountered in “Merge Conflicts” on page 92 when pulling in remote changes was a conflict between two branches; git pull does a merge between a remote branch and a local branch (more on this in “Branches and Remotes” on page 106). Had we encountered a merge conflict when running git merge, we’d follow the same strategy as in “Merge Conflicts” on page 92 to resolve it.
When we’ve used git log to look at our history, we’ve only been looking a few com‐ mits back—let’s look at the entire Git history now:
$ git log --abbrev-commit --pretty=oneline --graph --branches * e9a81b9 Merge branch 'readme-changes'
|\
| * 6e680b6 reformatted readme, added sample info
    * | dd57e33 added adapters file
    |/
    *   20041ab resolved merge conflict in README.md
    |\
    | * dafce75 added ref genome download date and link
    * | 08ccd3b added reference download date
    |/
    * 269aa09 added reference genome info
    * 46f0781 added information about samples
    * c509f63 added .gitignore
    * e4feb22 added markdown extensions to README files
    * 94e2365 added information about project to README
    * 3d7ffa6 initial import
Note that we have two bubbles in our history: one from the merge conflict we resolved after git pull, and the other from our recent merge of the readme-changes branch.


Branches and Remotes
The branch we created in the previous section was entirely local—so far, our collabo‐ rators are unable to see this branch or its commits. This is a nice feature of Git: you can create and work with branches to fit your workflow needs without having to share these branches with collaborators. In some cases, we do want to share our local branches with collaborators. In this section, we’ll see how Git’s branches and remote repositories are related, and how we can share work on local branches with collabora‐ tors.
Remote branches are a special type of local branch. In fact, you’ve already interacted with these remote branches when you’ve pushed to and pulled from remote reposito‐ ries. Using git branch with the option --all, we can see these hidden remote branches:
$ git branch --all * master
      readme-changes
      remotes/origin/master
remotes/origin/master is a remote branch—we can’t do work on it, but it can be synchronized with the latest commits from the remote repository using git fetch. Interestingly, a git pull is nothing more than a git fetch followed by a git merge. Though a bit technical, understanding this idea will greatly help you in working with remote repositories and remote branches. Let’s step through an example.
Suppose that Barbara is starting a new document that will detail all the bioinformatics methods of your project. She creates a new-methods branch, makes some commits, and then pushes these commits on this branch to our central repository:
$ git push origin new-methods
Counting objects: 4, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 307 bytes | 0 bytes/s, done. Total 3 (delta 1), reused 0 (delta 0)
To git@github.com:vsbuffalo/zmays-snps.git
     * [new branch]      new-methods -> new-methods
Back in our repository, we can fetch Barbara’s latest branches using git fetch <remo tename>. This creates a new remote branch, which we can see with git branch -- all:
$ git fetch origin
remote: Counting objects: 4, done.
remote: Compressing objects: 100% (1/1), done. remote: Total 3 (delta 1), reused 3 (delta 1) Unpacking objects: 100% (3/3), done.
From github.com:vsbuffalo/zmays-snps
     = [up to date]      master     -> origin/master
     * [new branch]      new-methods -> origin/new-methods
$ git branch --all * master
      new-methods
      remotes/origin/master
      remotes/origin/new-methods
git fetch doesn’t change any of your local branches; rather, it just synchronizes your remote branches with the newest commits from the remote repositories. If after a git fetch we wanted to incorporate the new commits on our remote branch into our local branch, we would use a git merge. For example, we could merge Barbara’s new- methods branch into our master branch with git merge origin/new-methods, which emulates a git pull.
However, Barbara’s branch is just getting started—suppose we want to develop on the new-methods branch before merging it into our master branch. We cannot develop on remote branches (e.g., our remotes/origin/new-methods), so we need to make a new branch that starts from this branch:
$ git checkout -b new-methods origin/new-methods
Branch new-methods set up to track remote branch new-methods from origin. Switched to a new branch 'new-methods'
Here, we’ve used git checkout to simultaneously create and switch a new branch using the -b option. Note that Git notified us that it’s tracking this branch. This means that this local branch knows which remote branch to push to and pull from if we were to just use git push or git pull without arguments. If we were to commit a change on this branch, we could then push it to the remote with git push:
$ echo "\\n(1) trim adapters\\n(2) quality-based trimming" >> methods.md $ git commit -am "added quality trimming steps"
[new-methods 5f78374] added quality trimming steps
     1 file changed, 3 insertions(+)
$ git push
Counting objects: 5, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 339 bytes | 0 bytes/s, done. Total 3 (delta 1), reused 0 (delta 0)
To git@github.com:vsbuffalo/zmays-snps.git
       6364ebb..9468e38  new-methods -> new-methods
Development can continue on new-methods until you and your collaborator decide to merge these changes into master. At this point, this branch’s work has been incorpo‐ rated into the main part of the project. If you like, you can delete the remote branch with git push origin :new-methods and your local branch with git branch -d new-methods.

Continuing Your Git Education
Git is a massively powerful version control system. This chapter has introduced the basics of version control and collaborating through pushing and pulling, which is enough to apply to your daily bioinformatics work. We’ve also covered some basic tools and techniques that can get you out of trouble or make working with Git easier, such as git checkout to restore files, git stash to stash your working changes, and git branch to work with branches. After you’ve mastered all of these concepts, you may want to move on to more advanced Git topics such as rebasing (git rebase), searching revisions (git grep), and submodules. However, none of these topics are required in daily Git use; you can search out and learn these topics as you need them. A great resource for these advanced topics is Scott Chacon and Ben Straub’s Pro Git book.
