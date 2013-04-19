---
title: Maintaining Repositories and Managing Pull Requests
---

This page is a collection of tips and tricks for dealing with pull requests and managing repositories on github.

## Fetching Pull Requests

Github actually stores the refs for each pull request on the main repo.  Assuming you have the `jquery` repo set as your `origin` you can use this git alias to make life a little easier:

```shell
git config --global --add alias.pr '!f() { git fetch -f origin refs/pull/$1/head:pr/$1; } ; f'
```

Once you have installed this alias, you can fetch a pull request based on number like so:

```shell
git pr 55
```

This will create a local branch called `pr/55` which you can checkout.

There are other alternatives, look at the instructions GitHub provides on the Pull Request page (usually hidden in a popup triggered by the info-icon next to the "Merge pull request" button (don't ever use that button!).

Regardless of how simple the patch may be, **never use GitHub's pull request interface to merge**. The merge button will always create a merge commit, which creates a messy history.

## Verifying author info and CLA

Check that the commit has full name and a valid email address (via `git log`).
Check the author has signed the CLA. If they haven't, ask them so sign it.
This step should eventually be automated.

## How to test a pull request 

To test the branch that will be pulled, you should start by checking out the branch locally.

```shell
git checkout pr/55
```

Run the unit tests, `grunt` etc, and make sure things work correctly.

### Testing the merge

To test if the fix works when merged into the official repo, you can simply merge the `pr/55` branch into your local `master` branch.

Note: You must be on your local master branch (or whatever branch you want to merge into) before executing the following command.

```shell
git merge pr/55
```

If everything looks good after the merge, you can push the changes up to GitHub and it will automatically close the pull request.

```shell
git push origin master
```

If there is a problem with the merge, you can reset back to the previous state and then leave a comment on the pull request.

```shell
git reset --hard origin/master
```

## Fetch and cherry-pick

You can also fetch commits and cherry-pick them. For that you need two steps:

```shell
git fetch <remote> <branch>
get cherry-pick <commit>
```

For example:

```shell
git fetch https://github.com/petersendidit/jquery-ui.git tabs_4386
git cherry-pick 710d762
```

Unlike merging, cherry-picking creates a new commit. This means that after you edit the commit message, your history will contain a commit with a different sha than the one in the pull request. When you push this commit up to GitHub the pull request will not be automatically closed. After pushing to GitHub, close the pull request with a comment that says "Thanks, landed in <sha>" where <sha> is the new commit. 

## How to fix a single commit

Sometimes there will be a pull request with a single commit that looks good, but the commit message doesn't conform to our Commit Message Style Guide, or just some whitespace that looks bad. In this case, `merge` or `cherry-pick` as described above, then amend the commit using `--amend`. That way you can edit the code changes or the commit message or both.

So assuming you fixed the code and now want to commit, use this:

```shell
git commit -a --amend
```

That'll give you a chance to edit the message, and will commit all changes you made.  After making the change you want, you can push.

Much like `cherry-pick`, this rebase will change the SHAs so you will need to manually close the pull request with a reference to the final commit.


## How to fix multiple commits

Often pull requests contain one initial commit and then multiple fixup commits, based on code reviews. Or you have multiple valid commits, but individual changes or commit messages are bad. In this case, an interactive rebase is your friend.

To start, get the commits you want to land in a local branch. For that, fetching the pull request as decribed above is the key.

```shell
git pr 55
git checkout -b temp-branch pr/55
git rebase master -i
```

Interactive rebase, triggered by that last line, will open your editor to let you choose what to do with each commit. Read the instructions included there or the entry for `git help rebase` for further info.

You can now merge the `temp-branch` into master as done above and push then delete the `temp-branch`.

Much like `cherry-pick`, this rebase will change the SHAs so you will need to manually close the pull request with a reference to the final commit.

## Backporting Fixes

If you merge (or cherry-pick) a fix into master that should be included in the next 1.x.y release, you'll need to copy the fix over to the 1-x-stable branch.

```shell
# git cherry-pick -x <sha>
git cherry-pick -x 710d762
``` 

The `-x` flag tells git to include a reference to the original commit in the new commit message.

This copies the changes from the commit in the master branch into the 1-x-stable branch, creating a new commit. Because this is a new commit, we use the -x option so that we can easily associate changes in the 1-x-stable branch with the original fix in the master branch.

Now that you've cherry-picked the commit into the 1-x-stable branch, all you need to do is push the commit to GitHub.
