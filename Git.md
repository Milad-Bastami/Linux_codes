These are tips and triks from a FaraDars course on Git.

# config
```
git config --global user.name "name"
git config --global user.email "email"
git config --global color.ui true
git config --global core.editor emacs
```


# init, add, commit, move or remove

## files are initially untracked > add them to stage > commit them

    git init
    git status
    git log   # see the commit history
    git log --prety=oneline --abbrev-commit  # nice formatings
    git log -n2   # last two  commits
    git log --graph #a text graph representation of commits
    git log --pretty=oneline --abbrev-commit --branches --graph #will display text graph of commit to all branches.

`git log` opens the commits history in the default `pager` which is usually program `more` or `less`. quite: `q`; fowrard: `space bar`; backward: `b`.
## add file/files to stage
Git will only track the fils that we specifie, not all files in the directory (large data files, ..). A tracked file is a  file git knows about it, but am **stagged file** is a file not only being tracked but also it's changes being stagged to be included in the next commit. `git add` both tracks new files and stage the changes.

    git add <file>
    git add -A  ## add changes from all tracked and untracked files
    git add "*.html"
    git add README data/README
    git add .
    git rm <filename> ## remove fie from git and filesystem

## commit staged files

    git commit <file>

## commit all staged changes

    git commit -m "message" -a

## add comment for log

    git commit -m "message" <file>

## `git diff` difference between commits and files
`diff` is a bash command (`diff -u`) and `HEAD` indicates the last commit.
- `git diff HEAD`: shows differences of working directory between the last commit (HEAD) & current change.
- `git diff`: diff compares the version of the file in the directory with the last staged changes. Therefore if all changes are staged, diff shows no difference.
- `git diff dafc75`: differences between last commit (HEAD) and specific commit (dafc75)
- `git diff 46f078 269aa09 README.md`: diffs between two commit to a file.
- `git diff HEAD~3 HEAD~2 README.md`: is equivalent to above command.
- In the diff output, `---a/file` indicate the current version (i.e the last commit) of a file and `+++b/file` indicates the version with changes. diff tries to break changes in `hunk`s (i.e. large changes blocks).

## see changes for staged files
This will show sttaged changes relative to the last commit (i.e. shows what is going to be in the next commit)

    git diff --staged
## Refering to commits (specifying revisions):
1. using the first 7 characters of Commits ID (SHA).
2. Specifying revisions relative to HEAD:
  1. `^` refers to the parent commit of a commit. `HEAD^`: parent of HEAD or one commit before HEAD. `HEAD^^: two commits before HEAD`. `HEAD^^^`
  2. `HEAD~2` is equivalent to `HEAD^^`.

## Undoing a stage `reset`

    git reset <file>
    echo "TODO: some changes" >> README.md
    git status                  # the change is stagged
    git reset HEAD README.md
    git status                  # the change is unstagged

  by default `git reset` will reset the **staging area** which is called **index**. So `git reset` resets the index. `HEAD` is a pointer to last commit in the current branch (which is by default the master).

## back to the last commit (i.e. `HEAD`)
it removes recent changes

    git checkout -- <file>

## moving or removing tracked files
we should use `git mv` or `git rm`. These command also stage the changes so that changes are ready to be commited

    git mv README README.md
    git commit -m "adding  extension to README"

## Telling Git to ignore: `.gitignore` file
when there are large number of files that should not be tracked (e.g. sequence files) and you dont want to see a long list of untracked file promt each time, you can creat a file `.gitignore` or edit it and add a line to it the following line `data/seqs/*.fastq`. Then add and commit this file.
1. add a list of file to be ignored to `.gitignore`. e.g. `data/seqs/*.fastq`
2. `git add .gitignore`
3. `git commit -m "added .gitignore"`

**Files that should be ignored:**
- *large files:* sequence files or other data files
- *intermediate files:* e..g. SAM or BAM files. it is better to store the command oor script that produce the data not the data itself.
- *text editor temporary files*: these contain ~ or #. These should always be ignored. E.g. using wildcards in .gitignore: `*~` and `\#*\#`.
- *temporary code files:*e.g. python temporary files like `overlap.pyc`

**Global .gitignore:**
universally ignore files across all repositories. GitHub has a usefull repository of candicate files to be excluded. we should add these files to `~/.gitignore_global` and then configure git to use it:

    git config --global core.excludesfile ~/.gitignore_global

# Branch
**Various purposes of branches:**
- experiment with your projects. e.g. test a new variant caller.
- devekope new features and bug fixes
- Working collaboratively. a branch for each contributor to make specific changes.
- Branching in git is visual (it does not copy the file into you account)

## Working with branches
- `git branch`: get the list of branches
- `git branch <branch name>`: creates branch
- `git branch -d <branchname>`: delete the branch
- `git checkout <branchname>`: swithch to a branch
- `git merge <Otherbranch>`: merge otherbranch to current branch. First check out to current branch first run the command.
- `git checkout <branchName>`: check out to a branch
- `git log --pretty=oneline --abbrev-commit --branches --graph`: will display text graph of commit to all branches.

## Remote branches
We can use `git branch --all` to see also remote branches. Actually when we add a remte repo to a local repo, we created also creatted a remote branch for the local `master branch`. The remote branch for the local `master` branch is `remotes/origin/master`. To synchronize the local with the remote branch, we use `git pull`. `git pull` is composed of first `git fetch` and then `git merge`.
When a colleague creates a local branch, named `new-methods` and commit some changes:
1. He can push commits to remote  repository: `git push origin new-methods`
2. We can fetch the *latest branches* from  the remote repo: `git fetch origin`
3. This will add a remore repo to the  output of `git branch --all`
4. To merge the remote branch to our local master: `git merge origin/new-methods` within master.
5. To add some commits to the remote branch and work on it, we first creates a new branch that *starts* from remote branch: `git checkout -b new-methods origin/new-methods`. The `-b` flag leads to creating and swithching to a branch simultaneously.
6. Then we can commit changes and usse `git push` or `git pull` within this new branch *without any arguments*.
7. When you and your decided to mergethe branch with master, you can delete the local branch using  `git branch -d new-methods` and remove remote branch using `git push origin :new-methods`
# Remote Repositories
## To clone a remote repository
when you clone a repositoty it is on origin > master. We can change the default origin name. The remote repository (i.e. **origin**) is automatically linked to the local repository (i.e. **master**). Therefore you can run `sudo git push origin master` to push your changes. If local directory is not  specified, it will clone into the current directory.

    git clone <repo URL> <local-directory>

## pushing your changes to the origin (reguires login)

    git push origin master
    git push <remote-name> <branch>

# pulling changes from origin (remote) to master (local): no login required
It is a good practice to *commit changes before pulling*. git gives an error if a pull will change a file that has uncommited changes.
    git pull origin master

## to add a remote repository to current git

    git remote add <name> <repo url>

## current remote status

    git remote
    git remote -v  ## verbose


# tags
Display a specific tag or commit

    git show <commit ID>
    git show <tag>

Display all commits

    git show

## add a tag (for version control) (-a annotate)

    git tag -a v2.0 -m "message"

## add a tag (i.e. version) to a specific commit

    git tag -a v.1.8 -m"this is version 1.8, relesable" <tag ID>

## list all tags containing "v"

    git tag -l 'v*'



# Pushing tags

## tag usually is not pushed by default. to push tags

    git push origin <tag name>
    git push origin v1.8  # push v1.8 to origin
    git push origin --tags # push all tags

## checkout to a tag (it's not like a branch)

    git checkout <tag name>


# signing taqs and commits

## `pgp` (prettyGoodPrivacy) is a key system which is implemented with gpg program
```
gpg --list-keys
gpg --gen-key  ## generate two key (pub & sec)
gpg --list-secret-keys --keyid-format LONG  ## list my secret keys

git config --global user.signingkey <sec key>

git tag -s v.2.1  -m "message" <commit ID> # -s: signing the tag
git show <tag ID>  ## signature is  addd to the tag
git tag -v v.2.1  ## verfy the signature
git commit -m "message" -S  ## signing the commit
```


# fixing buggs -- blame
finding the history of commits to a file or a line and blaming the guilty

    git blame <file name> -L8  # L specifies the line
    git blame <file name> -L8,10  # line 8 to 10

# finding buggs -- bisect
finding the bug by comparing commits. when a bug is present in the current state but was not present in a previous commit, you can find it by bisect. `bisect`: binary search of commits

    git bisect start
    git bisect bad                ## specifies the commit (here HEAD) with buggs
    git bisect good <commit ID>   ## specifies the commit (here HEAD) without buggs

git searches among commits (revisions) from good to bad stepwise and gives the name of a commit. you should check wether it is good or bad and specify the result of your considerations with (git bisect good or bad). After considering all revisions, bisect gives the  name and ID of the problematic commit.

# Collaborating  with git:
## `git remote; git push; git pull`
**remote repositories** are versions of our repository hosted remotely. To collaborate we need to configure our local repo to work with remote repo. You can collobrate through different *workflows*. A common easy workflow for collaboration is **shared central repositoory**:
 1. create a local repo and commit some changes
 2. push  repo and commits to shared central repo
 3. your  colleague clone the repo and make changes to his local version
 4. He then push his changes  to central repo
 5. You can get updated with his commits by pulling changes
 6. *merge conflicts* arise when youu and your colleague make changes to the same section of a file

## SSH and github
1. Create an ssh key, start the ssh-agent and add private to ssh-agent. Then add ssh key to GitHun account. If you use sudo with git, you should also use sudo to generate the keys. But it would be better to not use sudo for git or ssh. Therefore make sure to change the ownerhip of the repo directory in local machine using `sudo chown -hR milad /home/milad/Desktop/Linux-tips` and then you can use git and ssh without sudo.

    sudo chown -hR milad /home/milad/Desktop/Linux-tips
    ssh-keygen -t rsa -b 4096 -C "mi.bastami@live.com"
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
    cat ~/.ssh/id-rsa

2. Change your remote's URL from HTTPS to SSH with the `git remote set-url` command. Remotes URLs should be based on git user not https to be able to use ssh for authentication.

    git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
    git remote set-url origin git@github.com:Bastami/Linux-tips.git

    To reverse the change use:

        git remote set-url origin https://github.com/USERNAME/REPOSITORY.git

3. Then test your connection:

        ssh -T git@github.com

4. You are now able to use `git push origin master` to push your changes without authentication.

## Connecting with `git remote`
To configure local  repository to connect with a remote repository:

    git remote add origin git@github.com:username/repositoryName.git
    git remote -v       # Get the list of remotes
    git remote rm <remote name> # Remove a remotes

## Merge conflicts
The general strategy to resolve merge conflicts:
1. use `git status` to find the conflicting file(s).
2. open and edit the file.
3. use `git add` to tell git that you resolved the conflict
4. use `git status` to check that all changes are staged.
5. `git push`

When  you open the  file containing conflict, `<<<<<HEAD` indicates the start of our version. `=====` indicates the end of our HEAD and start of the collaborators changes. `>>>>> sdf..` indicates the end of collaborator changes and *the different conflicting chunk*. There may be more than one chunk.
Resolving merge conflicts accross multiple files may damage the code and prevent proper running. Any change need to be sanity checked after pulling and before resolving all changes. `merge  tools` like `meld` and `kdiff` can be used for side by side vizualization of the differences for more complicated merge conflicts.

## More GitHub workflows: `forking` and `pull request`
forking: creating a copy of someone else repository in you repository and cloning it to your local machine, making changes and making a pull request. Commits are monitors by leadeer and if appropriate incorporated into main repository.

# working with past Commits
## `git checkout`: getting file from the past
- `git checkout -- <file>`: cheking out a file  to the HEAD (i.e. last commit) and erasing all uncommited changes. `--` indicates we are cheking out a file not a branch.
- `git log --pretty=oneline --abbrev-commit -n 3` and checkout to a specific commit by `git checkout <commitID> -- <file>`

## `git stash`: Stashing you  changes
Stashing is a way of storing changes outside of commits history. Stashing is similar to checkingout with a difference that the messy changes we made to a repository can be stored and then restores the repository to HEAD. This is usefull whne you want to `pull` some changes or `branching`, but you recently added changes are not yet ready to be commited first. When we stash our working directory, the directory sets at the state of the last commit and is thefore clean. If frequently used, subcommands of slash may be useful: e.g. `git stash list` or `git stash apply`
1. `git stash`: stashes the working directory
2. `git stash pop`: reapply the changes.

## undoing and editing  Commits: `git commit --amend`
`git commit --amend` may be used if you made a mistake in a commits message or mistakanly commited some changes. This command will open the message of the last commit in the editor to edit it. The file itself may be changes, staged and then amended. Moreover, `git revert` or `git reset` can be used to undo a commit.
