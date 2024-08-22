# 03 - Version Control with Git

# Course Documents

[03 - Git Checklist.pdf](https://drive.google.com/file/d/1RCafn1RMyCsC3uyqAOKYItDOigwQARem/view?usp=drivesdk)

[03 - Git Handout.pdf](https://drive.google.com/file/d/1RId0PS9p03uc0RwQmPPV_lB68RTD_8vV/view?usp=drivesdk)

# Intro do Version Control and Git

- Devs working on the same code
- Code is hosted centrally on the internet
- Every developer has an entire copy of the code locally
- Code is fetched from remote repo and pushed to code repo
- Git knows how to merge changes automatically
- Merge Conflicts - when same line was changed e.g.
    - Git can't fix it alone… must be solved manually
- Best practice: push and pull often from the remote repo
    - Continuous Integration → integrate your code changes frequently
- Breaking changes doesn't affect you until you pulled the new code
- Version Control keeps a history of changes
- Every code change and file is tracked
- You can revert commits
- Each change labelled with commit message

# Basic Concepts of Git

- Remote Git Repository → where the code lives
    - Usually has an UI to interact with
- Local Git Repository → local copy of the code
- History → of code changes (git log)
- Staging → changes to commit
    - Working Dir → *git add* → Staging Area → *git commit* → Local Repo → *git push* → Remote Repo
- Git clients → to execute git commands

# Setup Git Repository Remote and Local

- Different Git Repository to register → github, gitlab, etc
- Platforms that host your repository
- Companies have own Git servers → Bitbucket
- private vs public repositories
- Repository created on GitLab
- Local Repository
    - git client needs to be connected with remote platform
    - you need to authenticate to GitHub/GitLab/…
    - your public ssh key must be added to the remote platform
    - $ git clone [git@gitlab.com](mailto:git@gitlab.com):ngmmartins/my-project.git
    

# Working with Git

- $ git status
    - status of local git repo
    - Untracked files are in working directory but not in the staging area.
    - Files that are in the staging area appear as "Changes to be committed”
    - Files that are under "Changes not staged for commit" were added to staging before but meanwhile they were modified
- $ git add <filename> OR git add . (will add all files)
    - this will put the added files in the staging area
- $ git commit
    - will commit only the changes under "Changes to be committed”
    - This will open a vim editor to enter the commit message
    - $ git commit -m "commit message" to add the commit message
- $ git log
    - history of changes (commits)
    - history from local repository
- $ git push
    - push the committed changes to the remote repository
    

# Initialise a Git project locally

- $ git init
    - do this on the directory we want to be a git repository.
- $ git status will show all files as Untracked
- $ git add . to add all files to staged
- $ git status will show all files as Changes to be committed
- $ git commit -m "Initial code”
- $ git status will show that we don't have any changes to commit, working tree clean
- $ git log
- This is still only a local repository. If we run $ git push it will fail and say we don't have a remote repo configured
- Go to GitLab and create an empty "test-node-app” repository
- $ git remote add origin git@gitlab.com"ngmmartins/test-node-app.git
    - tells local repository where to push the changes to
    - origin → your remote git repo
- if we do $ git push it fails because the branch has no upstream branch
- $ git push --set-upstream origin main

# Concept of Branches

- 1 branch per bugfix or feature
- Developers can commit without worrying to break main branch
- big feature branches long open, increase the chance of merge conflicts
- with branches you achieve a stable main branch
    - which is ready for deployment
- $ git branch → shows local branches
- When a new branch is created in the UI we can use: $ git pull
- $ git checkout <branch_name> → switches branches
- Create a branch from the command line:
    - $ git checkout -b <new_branch_name> → creates and switches to the new branch
        - It will create locally
    - Make some changes in the files
    - $ git add .
    - $ git commit -m "README edited”
    - $ git push --set-upstream origin feature/database-connection
        - can do just $ git push and it will fail and give the above command
- Many teams have a main and a develop branch
    - develop branch → intermediary master
    - main branch is always ready for deployment
    - during sprint teams work on features and bugfixes and merge them to develop.
    - at the end of sprint merge develop into master
- main branch vs main + develop branch
    - only main branch for continuous integration/delivery → **This is the modern DevOps way!**
        - pipeline is triggered whenever feature/bugfix code is merged into main
            - test → build → deploy
        - deploying every single feature/bugfix
    - with develop branch
        - features/bugfixes are collected in develop branch
        - develop branch often becomes "work in progress" branch
        

# Merge Requests / Pull Request

- best practice: Other developer reviews code changes before merging
- great chance to learn and grow from each other!
- on the remote git repo make the merge request/pull request
- As a reviewer you can approve or decline the pull request

# Deleting Branches

- Leave the branch or delete it
- best practice is to delete the branch after merging
    - if bugs arise or something needs fixed just create a new branch for that
- Note that deleting the branch on the remote doesn't delete it locally!
- To delete the branch locally:
    - $ git checkout main → switch to main branch
    - $ git pull → to get latest changes
    - $ git branch -d <name_of_branch_to_delete> → deletes the branch locally
    

# Rebase

**Avoiding merge commits**

**git pull vs git pull --rebase**

## git pull

- do a change on the remote repo in a branch
- do a change on the same local repo
- when trying to $ git push it will fail because we don't have the latest changes on the remote repo
- we need to do $ git pull to get latest changes
    - if it fails use $ git config pull.rebase false and the do $ git pull again
    - enter or leave the commit message
- $ git push → this will send the 2 commits (the one we did locally and the one we merged from the git pull)

## git pull --rebase

- do a change on the remote repo in a branch
- do a change on the same local repo
- when trying to $ git push it will fail because we don't have the latest changes on the remote repo
- $ git pull -r → git pull rebase, no merge commit, much cleaner project history (this change happens locally)
- $ git push → this will just send the commit we did locally

# Resolving Merge Conflicts

- Create a change in a local file and commit that change
- Create another change in the same file and line on the remote and commit it
- when trying to $ git push it will fail because we don't have the latest changes on the remote repo
- $ git pull -r
    - This will cause a conflict like "CONFLICT (content): Merge conflict in test.txt”
    - Since the changes are in the same place, git doesn't know how to proceed, so we must tell Git
    - to abort and get back to before the git rebase: $ git rebase —abort
    - Go to a editor to see the conflicts
        - In the editor will be possible to see the remote and local changes as the end result
        - we must decide which one to proceed
        - Example on VS Code before any action:
            
            ![Screenshot 2022-09-13 at 10.42.31.png](03%20-%20Version%20Control%20with%20Git%20694ac44b2ea7452397526abd1c385bb4/Screenshot_2022-09-13_at_10.42.31.png)
            
        - Example on VS Code after accepting the "Current" origin change (which is the one on the remote) and manually adding "… and locally”:
        
        ![Screenshot 2022-09-13 at 10.43.11.png](03%20-%20Version%20Control%20with%20Git%20694ac44b2ea7452397526abd1c385bb4/Screenshot_2022-09-13_at_10.43.11.png)
        
        - Click Accept Merge (not sure if it's necessary or just save)
    - Go back to t he command line and run $ git status to check, it will say the rebase is in progress
    - When the conflicts are solved we can run $ git rebase --continue
        - it can ask for a commit message
    - $ git push → to send the changes to the remote
    

# Gitignore

- add .gitignore file to your project (root directory)
- to exclude certain folders or files from git to be tracked
    - files/folders specific to your editor
    - build folders, where compiled code is located
- example of .gitignore file content:

```bash
.idea
.build
```

- if we already have a folde/file being tracked and we want to stop tracking it adding to .gitignore is not enough. We also need to remove it:
    - $ git rm -r --cached <folder/file_name> or $ git rm -r --cached .
    

# Git Stash

- save un-committed (work-in-progress) changes
- edit some file on the local repo without committing it
    - doing $ git status should show modified files
    - if we try to switch branches it will fail because there are local changes!
- $ git stash → will move the changes from working directory to a Stash
    - doing $ git status will reveal no changes (nothing to commit, working tree clean)
    - $ git stash list → lists the stashes
    - $ git stash show -p <stash_name> → shows a diff of the stash
        - example: $ git stash show -p stash@{0}
- after creating the stash we can for example switch branches
- when we want the stash changes back, we switch to the branch where the stash was created and:
    - $ git stash pop → this will retrieve the changes back
    

# Going back in history

- $ git log
    - Each commit has:
        - unique commit hash, like: e795e69327eb54f39b9c15b96e20652717ebb1a6
        - commit message
        - author + date
- Suppose we find a bug and notice it started since a certain time, we can "go back in time" and checkout the all code at that point (a specific commit hash)
- $ git checkout <commit_hash>
    - example: $ git checkout f1ab9dffbce9964367d7c0b312eb8862c26de64e
    - We now are in a "detached HEAD" state → it means we are not in the most update commit, but rather somewhere in the past
- After checking out the commit hash, we can actually create a new branch based on that commit hash
- $ git checkout <branch_name> will take you back to the up to date state of the branch
    - Example: $ git checkout bugfix/user-auth-error
    

# Undoing commits

## Reverting a commit (locally)

- remove one line on a file and edit the text on another line
- add and commit the file
- we realise the change was wrong and we want to revert the all commit
- if we do $ git log
    - the top commit is the one we just did, it's local to us and have the HEAD pointer
    - the next commit is the one in origin
- we can revert this last commit using:
    - $ git reset --hard HEAD~1
    - --hard → reverts the commit and also discards the changes
        - ~1 is the amount of commits we want to reset, in this case it's just 1, the last one.
- if we do $ git log the commit disappeared

## Correct a commit (locally)

- remove one line on a file and edit the text on another line
- add and commit the file
- we realise the change was wrong and we want to correct it and commit again
- we can revert this last commit using:
    - $ git reset HEAD~1
    - $ git reset --soft HEAD~1 is the same, but as it is the default it can be omitted
- if we do $ git status the change is still there, but if we do $ git log the commit is gone
- correct the file, add it, and commit it again

## Amend a commit (locally)

- remove another line of the file
- do a $ git add .
- $ git commit --amend → this will create a new commit and merge with the previous one
- $ git push

## Reverting a commit (on the remote also)

- we realised that a commit was wrong and it was already pushed to the remote
- $ git reset --hard HEAD~1
    - This removes the commit and discards changes from the local repo
    - but at this point it still is on the remote
- $ git push --force
    - we need to use --force because we are behind the remote, so a regular git push will fail
    - at this point the commit is gone from the remote
    - don't do this in main and develop branches!
    - only when working alone in a branch
    

## Alternative to revert in main/develop branch

- switch to main/develop branch
- remove a line on a file
- add, commit and push
- $ git revert <commit_hash>
    - Example: git revert 46cd2dbbce5f07e949f8c9fb3087cef29fb7abc9
    - the commit message already includes the Revert "commit message” word
    - This revert command actually creates a new commit with the changes reverted, but the "old/wrong" commit is not deleted as with git reset
- $ git push
- After this the remote will have a new commit called something like Revert "commit message” but the other old/wrong commit stays there

# Merging Branches

- We are working on a bugfix/b branch which has already some commits. Meanwhile main branch also got some commits since we branched. We want to merge the changes from main to our bugfix/b branch.
- Local main branch needs to be ip-to-date
    - $ git checkout main
    - $ git pull
- Switch back to bugfix/b branch
    - $ git checkout bugfix/b
- $ git merge <source>
    - Example: $ git merge main
    - This will take the changes from main and merge into our bugfix/b
    - it will create a new commit and we can do git push
    

# Git for DevOps

**Use cases for Git for DevOps**

- Infrastructure as Code
    - Many Kubernetes config files
    - Terraform and Ansible config files
    - Bash and Python scripts
    - Files should be
        - tracked → history of changes
        - securely stored in one place
        - shareable for DevOps team
- CI/CD Pipeline and Build Automation
    - checkout code, test and build application, etc
    - need integration for the build automation tool with application git repository
    - you need to setup integration with build automation tool and git repository