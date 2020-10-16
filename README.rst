========================
dCache developers guide
========================

A book containing useful information for developers.


Git configuration
-----------------

Fork and clone the repo from github.com:dCache/dcache.git. Then configure the repo::

    > git config --add  reviewboard.url https://rb.dcache.org
    > git remote add upstream git@github.com:dCache/dcache.git


Development procedure
---------------------

**1.**:  create branch locally from upstream master or branches, make changes and commit them to branch, submit the patch to ReviewBoard for review::

    > git checkout -b <newbranch> -t  upstream/master 

To check::

    > git branch -vv|grep <newbranch>
    * newbranch              80c082c [upstream/master] httpd: Fixed table headers in usageInfo

Do changes.

Write the commit message, e.g.::

 %(component)s:%(title)s
 
    Motivation:

    %(motivation)s

    Modification:

    %(modification)s

    Result:

    %(result)s

    Target: master
    Require-notes: yes
    Require-book: no
    Request: 3.6
    Request: 3.0
    Request: 2.16
    Request: 2.15
    Request: 2.14
    Request: 2.13
    Patch: https://rb.dcache.org/r/xxxx
    Committed: master@80c082c
    Pull-request: https://github.com/dCache/dcache/pull/xxx
    Pull-request: https://github.com/dCache/dcache/pull/xxx

Then commit::

    > git commit -F commit-msg 

To post::

    > rbt post -g -o --target-groups=all

To post and publish directly::

 > rbt post -g -o --target-groups=all -p

Patch should be created on  https://rb.dcache.org/r/<id>/

To update::

    > git commit --amend -F commit-msg 
    > rbt post -r <id> -g

**2.**: Wait for review (or bug/annoy people, accordingly ;-)

**3.**: Update patch and update RB (if appropriate)

**4.** Push changes to master.

Two possible contributions: new features and bug-fixes.  
New features we don't back-port (into stable branches) 
bug-fixes we do

The first rule-of-thumb is that a patch won't be accepted into a branch unless the equivalent functionality is in all subsequent branches.  For example, before the patch goes into 3.0, it needs to be in 3.1; before it goes into 3.1, it needs to be in 3.2 (which is currently `master`).  So, to get it into the current "golden release" (2.16), you'll need to also request 3.1 and 3.0.

The second rule-of-thumb is that we fix all our supported branches.  This is currently 3.1, 3.0, 2.16, 2.15, 2.14 and 2.13.

Now, we want to drop support for 2.13, 2.14 and 2.15 ... and should have done so already, by releasing 3.2 (dCache v3.2.0), but that hasn't happened.

squash several commits  before committing / pushing into dCache master.  The "important" (or desirable ;-) part is that there's a single commit when pushing into upstream `master`.

The next step is to get it into dCache master.

Update the git commit message to include a reference to the RB entry (add a `Patch: https://...` metadata entry

You should then do a merge into master -- I use `--ff-only` as a safety check::

> git checkout master
> git merge --ff-only <branch>

Your `master` branch should now be one commit ahead of `upstream/master`.

Then do::

 $ git push upstream HEAD
 Counting objects: 37, done.
 Delta compression using up to 4 threads.
 Compressing objects: 100% (10/10), done.
 Writing objects: 100% (10/10), 1.04 KiB | 0 bytes/s, done.
 Total 10 (delta 7), reused 0 (delta 0)
 remote: Resolving deltas: 100% (7/7), completed with 7 local objects.
 To git@github.com:dCache/dcache.git
    45a40c3..80c082c  HEAD -> master

To push your `master` branch into `upstream/master`. Note: You should have installed you ssh keys in github and access to the dcache.git repo should have been granted)

That should give you a commit id (in the example: 80c082c) for the patch in `upstream/master`.  Add that as metadata to the RB entry: `Committed: master@<id>` (e.g., see https://rb.dcache.org/r/10253/)

You then need to create the pull-requests for back-porting the patch into the stable branches::

  fix/<branch>/rb<rb-number>

Example::

 git checkout -b fix/3.1/rb10263 upstream/3.1
 git cherry-pick master
 git push origin HEAD

 git checkout -b fix/3.1/rb10374 upstream/3.1
 git cherry-pick master
 git push origin HEAD


If you point your web-browser at github.com/dcache/dcache then you should see a yellow-ish box near the top about a recently created branch.
That should allow you to create the pull-request (the 3.1 in fix/3.1/... is a handy reminder into which dCache branch it should go)

A bit of clicking should give you a pull-request for your patch into dCache v3.1.  You can copy that URL into RB as a Pull-request metadata.

Then do the same for the next branch::

 git checkout -b fix/3.0/rb10263 upstream/3.0
 git cherry-pick fix/3.1/rb10263
 git push origin HEAD

Doing the cherry-pick against the n-1 branch (e.g. `fix/3.1/rb10263` here) means that any adaptation and changes that are needed as you "go back in time" (earlier dCache releases) are available for subsequent (even more in the past) releases.
