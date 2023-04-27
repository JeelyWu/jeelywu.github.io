---
title: "How to checkout to a remote branch in git"
date: 2022-11-04T22:07:35+08:00
---

Hi there, in this short note, I'd like to show you how to checkout to a remote branch on your local machine.

Step 1: Fetch all the branches by using `git fetch`

This command will download the new branches to your local. For more detail about `git fetch`, you can look at the helping document by typing: `git fetch --help`.

After fetching, you can use the `git branch -r` to see all the remote branches to ensure that the branch you want to checkout has been downloaded.

Step 2:  Create and checkout to the new branch.

You can use the command:

`git checkout -b <your_local_branch> origin/<you_remote_branch>`

By this command, you will create a new local branch name <your_local_branch>, and it will bind to the origin/<you_remote_branch>.  And, in most cases, we use the same name for the local branch and remote branch, so the command in the real world may be like this:
``
`git checkout -b feature/some_new_feature origin/feature/some_new_feature`

