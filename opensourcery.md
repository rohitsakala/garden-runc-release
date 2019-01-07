# Opensourcery

## runc

Sometimes it is necessary for us to contribute to runc (to fix a bug or
add/improve a feature). For general contribution guidelines please see runc's
own contributing guide.

In order to run runc's test suite you will need a Linux machine with docker
installed. From the runc directory, just run `sudo make test` (yes, tests
require sudo).

Once you've made a commit locally that you wish to contribute back to runc,
push that change to a branch on cloudfoundry-incubator/runc and submit the PR.
It may be necessary, for urgent bug fixes, to include our change in a shipped
garden-runc-release before our PR in merged into runc proper. Currently we are
experimenting with using git patches to achieve that.

### Shipping with a patched runc

While we should try to avoid this, sometimes it is necessary. The process for
shipping garden-runc-release with a runc patch is thus:

1. from the PR branch, produce a git patch with `git format-patch -1 HEAD`
1. move the patch produced into `garden-runc-release/src/patches/runc`
1. patches are applied in ascending ASCII order, if necessary rename the patch so that it begins with the next numerical increment
1. test that the patch can be cleanly applied to our submoduled version of runc with `patch -p1 < /path/to/patch`
1. author a chore to delete the patch once the PR is merged

Make sure that we're only producing patches from actual commits to PR branches.
If we update the PR - we should also update the patch.

It is possible that the PR commit cannot be cleanly applied as a patch to our
submoduled version of runc. An example situation is shown below whereby there
are intermediate commits between our submoduled runc and our commit to runc
master that would cause a merge conflict when trying to apply the patch.

```
* e66cf02 (cf/awesome-pr, cf/HEAD, awesome-pr) awesome PR
* fbf261d (origin/master, origin/HEAD, master) some runc commit  # <-- caused merge conflict applying e66cf02 over 4643da3
| * e66cf02-patch
|/
* 4643da3 (tag: v1.0.0-rc6) some other runc commit  # <-- submoduled version
```

A solution that has worked for us here is to produce a compatibility patch,
which resolves the merge conflict. So then our pseudo-tree looks like:

```
* e66cf02 (cf/awesome-pr, cf/HEAD, awesome-pr) awesome PR
* fbf261d (origin/master, origin/HEAD, master) some runc commit
| * e66cf02-patch
| * e66cf02-compatibility-patch
|/
* 4643da3 (tag: v1.0.0-rc6) some other runc commit  # <-- submoduled version
```

And the contents of `garden-runc-release/src/patches/runc` would look like:

```
0001-awesome-pr.compatibility.patch
0002-awesome-pr.patch
```

You can think of it like resolving any git merge conflict, except we resolve it
via a patch so that our PR remains mergeable with runc master. Note that if you
do create a compatibility commit, we should:

1. change the minimum possible to facilitate a patch merge
1. bump the submoduled runc to a later version when possible and remove the compatibility patch
