These are tips and triks from a FaraDars course on Git.

# config
```
git config --global user.name "name"
git config --global user.email "email"
```


# init, add,  commit

## files are initially untracked > add them to stage > commit them

    git init
    git status
    git log

## add file/files to stage

    git add <file>
    git add -A  ## add all files
    git add "*.html"

## commit staged files

    git commit <file>

## commit all staged changes

    git commit -m "message" -a

## add comment for log

    git commit -m "message" <file>

## difference for last commit
`diff` is a bash command (`diff -u`) and `HEAD` indicates the last commit. The following command shows differences between the last commit & current change:

    git diff HEAD

## see changes for staged files

    git diff --staged

## remove changes from stage

    git reset <file>

# back to the last commit (i.e. `HEAD`) and removing recent changes
    git checkout -- <file>


# Branches

## give the list of branches

    git branch
    git branch <branch name>

## delete the branch

    git branch -d <branchname>

## swithch to a branch

    git checkout <branchname>

## merge a branch

    git merge <branch name>

## remove the file from git & file system

    git rm <filename>


# Remote Repositories

## To clone a remote repository
when you clone a repositoty it is on origin > master. We can change the default origin name. The remote repository (i.e. **origin**) is automatically linked to the local repository (i.e. **master**). Therefore you can run `sudo git push origin master` to push your changes.

    git clone <repo URL>

## pushing your changes to the origin (reguires login)

    git push origin master

# pulling changes from origin (remote) to master (local): no login required

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

# list all tags containing "v"

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
