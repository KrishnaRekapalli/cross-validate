---
toc: false
layout: post
description: A common probelm that I run into when working on active repositories.
categories: [git, development, software engineering]
title: Git's magic to resolve common issues with active repositories
---

Whenever I create a branch for developing a new feature, I do a `git fetch` and `git rebase origin/main` to make my local main brach upto date with the remote branch. Then I create a new branch using `git checkout -b <NEW_BRANCH_NAME>`

But when working on repositories with active development where multiple people simultaneosly merge their changes to main branch, it gets a bit messy as the main branch in the remote is the ahead of the local main branch and thus the current local branch I am working on is out of sync. 

To demonstrate the issue better, let us pick an example repository. I have created a new repo in my GitHub `upgraded-octo-waffle`

![]({{ site.baseurl }}/images/repo_on_gihub.png "Initial state")


I cloned the reposiotry into my local machine using 

```shell
$ git clone git@github.com:KrishnaRekapalli/upgraded-octo-waffle.git
```

If the repository is cloned into the local machine sometime ago and if the local main and the remote main branches are out of sync, we can use the commands to check if the local brnach needs to be updated and take necessary action i.e. `rebase`

```shell
$ git fetch # fetches the latest changes
$ git status # shows how far beihid is the local brnach from the remote
$ git rebase origin/main #updates the local branch with all the new changes
```

Now once I have all the latest changes incorporated to my local `main` branch, I set out to work on my new feature by creating a new branch.

```shell
$ git checkout -b KR-my-new-feature
```
![]({{ site.baseurl }}/images/git_new_branch.png "Creating a new branch")

Before changes:
![]({{ site.baseurl }}/images/before_changes.png "Before changes")

New changes:
![]({{ site.baseurl }}/images/new_local_changes.png "After changes!")

Now I commit my new changes to the local branch.

```shell
$ git add waffle.py
$ git commit -m "added new method"
```
Here comes the twist in the story. Now I just realise that a colleage has just merged her changes to the remote/main and they also edited the same file `waffle.py`. Now `waffle.py` looks like:

![](/images/colleague_changes_waffle.png "Colleague's changes")

Now I want to incorporate the new changes to my branch. What can I do? 

I first try:
```shell
$ git fetch
$ git rebase origin/main
```

The response is:
![]({{ site.baseurl }}/images/failed_rebase.png "Failed rebase")

The rebase operation fails (:roll_eyes:) because there are changes to the same file and same locations and Git is not able to figure out a clean way to update `waffle.py` with my colleage's changes on my feature branch.

Then I abort the rebase operation by doing
```shell
$ git rebase --abort
```
One solution to this problem is:
1. Undo the last commit 
2. Use `stash` option to save my changes
3. Update the branch with my colleages changes by using `rebase`
4. Use `stash pop` to apply my changes on the top of the updated brnach
5. Resolve conflicts
6. Commit the final changes agter resolving conflicts
7. Push my changes to the remote and merge

We will go over this proces ste-by-step now

##### 1. Undo last commit
To undo the last commit to my branch, I do
```shell
$ git reset --soft HEAD~1
```
The number `1` indicates that we want to go back one commit and by using the flag, `soft` we are asking Git to not discard the changes we made. 

##### 2. Stash my changes
Now I stash my changes by doing
```shell
$ git stash
```
This step keeps my changes safe and now I can use rebase to update my branch. 


##### 3. Update the brnach with latest changes
For this I do
```shell
$ git rebase origin/main
```
Reabse now works without any complaints :smiley:

![]({{ site.baseurl }}/images/successful_rebase_after_reset.png "Successful rebase")

##### 4. Apply my changes again using Stash pop

Now it is time to apply my changes on to the feature brnach I am working on. First, I do

```shell
$ git stash pop
```

Now `waffle.py` will look like:
![]({{ site.baseurl }}/images/after_stash_pop.png "After Stash pop")

So Git is expecting us to resolve the conflicts manually as the changes happened at the same place. Although these are different methods, Git is not so smart to figure that out. So it is our job to resolve the conflicts i.e. keep the changes we want and discard the changes we don't.


##### 5. Resolve conflicts
As I want to keep both the methods, I just remove the <<<<<< and ======= and >>>>>>> symbols and now commit all the changes to the current branch


##### 6. Commit the overall changes
Now as all the issues are resolved I simply do

```shell
$ git add waffle.py
$ git commit -m "That was close! All good now :sweat_smile:"
```

##### 7. Push changes to remote and merge my changes

Now we need to push the local brnach to remote so that it can be merged with the `main` branch. For this I do

```shell
$ git push -u origin KR-my-new-feature
```

What this does is creates a new remote brnach with the same name as the local one and pushes all the changes there. 

The resposne of the command:
![]({{ site.baseurl }}/images/fin..png "Fin")

Now I can happily create a PR and merge my new feature with the main branch. 

I cant resist but end this post with one of xkcd comics on Git. This captures my emotions about git for most part. While it is a game changing tool that spearheaded collaborative development, it can take a bit of time to get a hang of the capabilities and for most part, one may end up using only 10-15% of the core features of Git and there is a lot I don't understand and keep learning as I run into problems :smiley: 

![](https://imgs.xkcd.com/comics/git.png "XKCD on Git")
