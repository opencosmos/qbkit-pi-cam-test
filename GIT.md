# Git troubleshooting

This document uses "Github" to refer to the Git server, but should work with any Git provider (including self-hosted).

References to using Git locally imply git-pulling the relevant branches to your computer, then using Git on your computer to resolve a problem, before pushing the result back to Github.  This includes tools such as:

 * `git` command-line suite

 * gitk

 * gitg

 * git-cola

 * git-dag

 * proprietary offline Git GUIs (e.g. from Atlassian, Github, Microsoft).

The expected workflow involves repository which are either:

 * External content, only *read* (pulled) by the qbkit, but never written by the qbkit.

 * Generated content, only *written* (pushed) by the qbkit, but never pulled by the qbkit, and never written to by anything other than the qbkit.

Following this pattern of read-only / write-only repositories (which are never written to by _both_ the qbkit and by something else) should avoid any of the problems listed below.

## I try to pull from Github, but the files aren't appearing on the qbkit.  I have commits in Github from elsewhere.

**Cause:**

A common cause of this is if you created an empty repository, then added commits to it which aren't on the qbkit.
Then you created commits on the qbkit.

**Problem:**

Git will fail to merge the new commits from Github since they have no common ancestry.

**Solution 1: cherry pick**

This will add the commits from other sources into your history _after_ the commits from the qbkit.

 1. Via qbapp, push the commits from the qbkit to a new branch.

 2. Via Git locally, cherry-pick the commits from the original branch (which were not created on the qbkit) to the qbkit's branch.

 3. Push the result back to Github.

 4. Now pull that branch onto the qbkit.  It will do a fast-forward, including the new commits that you cherry-picked.

**Solution 2: rebase, reinstall (less safe)**

This will add the commits from other sources into your history _before_ the commits from the qbkit.

Only do this if you are familiar with rebasing and the risk it poses to your data.
If things go wrong, the reflog is your friend.

 1. Via qbapp, push the commits from the qbkit to a new branch.

 2. Via Git locally, rebase the new branch onto the previous branch.
    This re-writes the history of the new branch to include the original branch before the commits of the new branch.

 3. Delete the Git volume from the qbkit, then re-create it.
    Issue a pull command.
	    The newly-pulled volume will now contain the commits which did not originate on the qbkit, followed by the commits pushed from the qbkit.

## I try to push to Github but it fails.  I have commits in Github from elsewhere.

**Cause:**

You cannot push from qbkit to Github if the qbkit is not up-to-date.

**Solution 1: fast-forward the qbkit**

 1. Via qbapp, pull the new commits in.
    This will bring in the new commits, as a fast-forward.
    Your new commits on the qbkit will be "on top of" the external commits.

 2. Via qbapp, push.
    This will push your new commits from the qbkit.

This method will fail on merge conflicts, i.e. if both the qbkit's commits and the external commits make changes to the same file in nearby regions (or if one set modifies the file while another deletes it).
In which case, push from qbkit to a new branch, then resolve the merge via Git locally.
