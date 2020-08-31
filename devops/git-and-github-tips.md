# Git & GitHub Tips

### Add Upstream Remote to Our Forked Git Repos

Assuming we have forked the repo.

Now we clone our forked repo:

```text
$ git clone git@github.com:YOUR-USERNAME/THE-REPO.git
```

See what remotes we have:

```text
$ git remote -vv
origin	git@github.com:YOUR-USERNAME/THE-REPO.git (fetch)
origin	git@github.com:YOUR-USERNAME/THE-REPO.git (push)
```

To add the upstream remote:

```text
$ git remote add upstream git://github.com/UPSTREAM-USERNAME/THE-REPO.git
$ git remote -vv
origin	git@github.com:YOUR-USERNAME/THE-REPO.git (fetch)
origin	git@github.com:YOUR-USERNAME/THE-REPO.git (push)
upstream	git@github.com:UPSTREAM-USERNAME/THE-REPO.git (fetch)
upstream	git@github.com:UPSTREAM-USERNAME/THE-REPO.git (push)
```

### Syncing a fork

1. Fetch the branches and their respective commits from the upstream repository.

Commits to `master` will be stored in a local branch, `upstream/master`.

```text
$ git fetch upstream
```

1. Check out your fork's local `master` branch.

```text
$ git checkout master
```

1. Merge the changes from `upstream/master` into your local `master` branch.

This brings your fork's master branch into sync with the upstream repository, without losing your local changes.

```text
$ git merge upstream/master

$ git status
On branch master
Your branch is ahead of 'origin/master' by 4 commits.
  (use "git push" to publish your local commits)
```

If you just want to sync up without local changes, you may push it after `git merge`:

```text
$ git push

$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Check What's Going to be Pushed?

For a list of files to be pushed, run:

```text
$ git diff --stat --cached [remote/branch]
```

### How to Delete Local and/or Remote Branch?

```text
$ git push --delete <remote_name> <branch_name>
$ git branch -d <branch_name>
```

### To Contribute Multiple PRs?

Better use a dedicated branch for one PR, like below.

For PR-1:

```text
$ git checkout master
$ git checkout -b pr-1
...work, work, work
$ git add xxx
$ git commit -am "add pr-1"
$ git push origin pr-1
```

Now there is another PR-2 you want to do, while waiting for the PR-1 to be accepted and merged:

```text
$ git checkout master
$ git checkout -b pr-2
...work, work, work
$ git add xxx
$ git commit -am "add pr-2"
$ git push origin pr-2
```

### How to "Rollback" the versions of one specific file?

```text
$ git checkout master~2 ThatFile.ext
$ git commit -m "revert back" ThatFile.ext
$ git push
```

### `git push --set-upstream origin master` but somehow the old user is being used?

```text
$ git push --set-upstream origin master
remote: Permission to brightzheng100/xxx.git denied to <ANOTHER/OLD USER>.
```

If you checked through `git config --global -l` or `cat ~/.gitconfig` but same issue remains, it's most likely caused by **OSX Keychain**. Resolve it by following below steps:

1. In Finder, search for the `Keychain Access app`
2. In Keychain Access, search for `github.com`
3. GitHub Password Entry in KeychainFind the "internet password" entry for `github.com`
4. Edit or delete the entry accordingly.

### Merge PR With Conflicts

```text
# start from master branch
git checkout master

# checkout a new branch
git checkout -b pr-merging

# add the PR in as a remote
git remote add pr-agregory999 https://github.com/agregory999/platform-automation-pipelines.git
# fetch the content
git fetch pr-agregory999
# check it out as a local branch
git checkout --track pr-agregory999/master -b pr-agregory999

# rebase pr-merging so both have same ancestor
git rebase pr-merging

# work work work
<WORK ON CONFLICTS...>
<ADD/EDIT/DELETE FILES...>

# add all changes as one shot & commit
git add .
git commit -am "Merge pull request #3 from agregory999/master"

# checkout to pr-merging and merge it
git checkout pr-merging
git merge --no-ff --log -m "Merge PR #3 from agregory999/master" pr-agregory999

# checkout master
git checkout master

# merge pr-merging with --squash
git merge --squash pr-merging

# now all changes will be displayed
git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
	new file:   ops-files/resource-gcs.yml
	modified:   vars-dev/vars-common.yml
	modified:   vars-pez/vars-common.yml

# add all changes in and commit
git add .
git commit -am "Merge pull request #3 from agregory999/master"

# push it
git push


#######clean up##########

git branch -D pr-agregory999
git branch -D pr-merging
git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

### Ref

1. [https://help.github.com/articles/syncing-a-fork/](https://help.github.com/articles/syncing-a-fork/)
2. [https://stackoverflow.com/questions/3636914/how-can-i-see-what-i-am-about-to-push-with-git](https://stackoverflow.com/questions/3636914/how-can-i-see-what-i-am-about-to-push-with-git)



