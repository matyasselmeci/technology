<!-- TODO This page needs an overhaul -->

Software Development Process
============================

This page is for the OSG Software team and other contributors to the OSG software stack.
It is meant to be the central source for all development processes for the Software team.
(But right now, it is just a starting point.)

Overall Development Cycle
-------------------------

For a typical update to an existing package, the overall development cycle is roughly as follows:

1.  Download the new upstream source (tarball, source RPM, checkout) into
    [the CHTC upstream area](../software/rpm-development-guide.md#upstream-source-cache)
2.  In [a checkout of our packaging code](../software/rpm-development-guide.md#revision-control-system),
    update [the reference to the upstream file](../software/rpm-development-guide.md#upstream) and,
    as needed, [the RPM spec file](../software/rpm-development-guide.md#osg)
3.  Use [osg-build](../software/osg-build-tools.md#osg-build) to perform a scratch build of the updated package
4.  Verify that the build succeeded; if not, redo previous steps until success
5.  Optionally, lightly test the new RPM(s); if there are problems, redo previous steps until success
6.  Use [osg-build](../software/osg-build-tools.md#osg-build) to perform an official build of the updated package
    (which will go into the development repos)
7.  Perform standard developer testing of the new RPMs; this generally means running the VM Universe tests - see below for details
8.  Have another software team member review your testing and give you permission to promote the package
9.  Promote the package to testing — see below for details


Build Procedures
----------------

<!-- TODO not written yet
### Verifying builds through GitHub Actions
-->

### Initial repo setup

1.  Create your own fork of the <https://github.com/osg-htc/software-packaging> repository.
2.  Clone your fork, then add the upstream repository as a remote (replacing `<YOURNAME>` with your GitHub username):

        :::console
        $ git clone https://github.com/<YOURNAME>/software-packaging
        $ git remote add upstream https://github.com/osg-htc/software-packaging


### Basic building

<!-- TODO flesh this out -->

1.  Make changes
2.  Make a scratch build
3.  If you have permissions, push directly upstream

        :::console
        $ git push upstream main

    If you do not have permissions, or want your changes reviewed,
    push to a branch on your own fork and make a Pull Request
4.  Make a non-scratch build


### Building packages for multiple OSG release series

The OSG Software team supports two release series in parallel;
many of the packages are identical or very similar between release series,
and you will be making the same change to a package across both release series.

We recommend using a diffing tool such as Meld (Linux) or WinMerge (Windows) that is capable of comparing
directories and not just individual files, to make sure all the changes are carried over.

This is one example workflow, using the "24-main" and "25-main" release series,
and the package "xrdcl-pelican" which is identical between the two release series.

1.  Make changes to `24-main/xrdcl-pelican` 
2.  Make a scratch build (e.g. `osg-build koji --scratch 24-main/xrdcl-pelican`)
3.  Open `24-main/xrdcl-pelican` and `25-main/xrdcl-pelican` in a diffing tool, and copy
    over all the changes
4.  `git add` and `git commit` the changes; committing the changes to both series in the same commit
    means you don't have to write the commit message twice,
    and `git blame` will be accurate for both release series
5.  `git push` to the upstream repo
6.  Make non-scratch builds; use the following procedure to submit both at the same time:

        :::console
        $ osg-build koji --nowait 25-main/xrdcl-pelican
        $ osg-build koji --nowait 24-main/xrdcl-pelican
        $ osg-koji watch-task --mine


If your development process is more iterative and you need multiple commits,
you could do something like the following (using `xrootd` as an example):

1.  Make changes to `24-main/xrootd`
2.  Make a scratch build as above
3.  Commit your changes as a "checkpoint"
4.  Repeat 1-3 as necessary until you think you are ready for a non-scratch build
5.  Copy your changes to `25-main` using a diffing tool as above
6.  You now have two options:

    -   If you haven't pushed, then you can do an interactive rebase to squash your changes to
        `24-main` and `25-main` down to one commit

    -   If you have already pushed, get a log of the recent changes to `24-main/xrootd`:

            :::console
            $ git --no-pager log --oneline --since='last week' --reverse -- 24-main/xrootd

        Copy and paste that to a text editor.
        Then, when you write the commit message for the commit to `24-main/xrootd`,
        reference the commits from that command.
        For example:

            25-main/xrootd: Update xrootd to 5.8.4 and add various patches

            Includes the following commits from 24-main/xrootd:

            8061d7d40 24-main/xrootd: update to 5.8.4; update patches, bring spec file closer to upstream
            13b858dce 24-main/xrootd: add some more patches
            773f57f21 24-main/xrootd: Add TPC worker pool patch

        Keeping the Git hashes will make future archaeology easier.


Testing Procedures
------------------

Before promoting a package to a testing repository, each build must be tested lightly from the development repos
to make sure that it is not completely broken, thereby wasting time during acceptance testing.
Normally, the person who builds a package performs the development testing.

**If you are not doing your own development testing for a package**, contact the Software Manager
and/or leave a comment in the associated ticket; otherwise, your package may never be promoted to testing and hence never released.

Testing should be performed for all distro versions the package is built for,
and all supported release series.
In addition to a fresh installation, it is also important to test upgrades
from a previous version of the software.

### VM Universe (VMU) tests

These are OSG Software's automated tests, which run in VMs powered by HTCondor's VM universe.
Each run tests installation and/or upgrades of the listed software,
using the [OSG-Test test suite](https://github.com/opensciencegrid/osg-test) of integration tests.

Even if there are no functionality tests for your package in OSG-Test,
you should run the VMU tests to validate the RPM installation and upgrade process.

Make sure you meet the [pre-requisites](https://github.com/opensciencegrid/vm-test-runs) required
to submit VM Universe jobs on `osgsw-ap.chtc.wisc.edu`.
If you do not have permission to submit VMU runs,
mention "@Software and Release" on the Jira ticket and a Software Team member will run the tests for you.

After that's done, prepare the test suite with a comment describing the test run.
For example, if you were testing a new `htcondor-ce` package:

``` console
osg-run-tests 'Testing htcondor-ce-3.2.1-1 (SOFTWARE-####)'
```

After you `cd` into the directory specified in the output of the previous command,
you will need to edit the `*.yaml` files in `parameters.d` to reflect the tests that you will want to run,
i.e. clean installs, upgrade installs and upgrade installs between OSG versions.

Once you're satisfied with your list of parameters, submit the dag:

``` console
./master-run.sh
```



Promoting a Package to Testing
------------------------------

Once development and development testing is complete, the final OSG Software step is to promote the package(s)
to our testing repositories.
After that, the Release team takes over with acceptance testing and ultimately release.
Of course if they discover problems, the ticket(s) will be returned to OSG Software for further development,
essentially restarting the development cycle.

### Preparing a Good Promotion Request

Developers must obtain permission from the OSG Software manager to promote a package from development to testing.
A promotion request goes into at least one affected Jira ticket and will be answered there as well.
Below are some tips for writing a good promotion request:

-   Make sure that relevant information about goals, history, and resolution is in the associated ticket(s)
-   List the NVRs of the builds to be promoted, as well as the tags they will be promoted into.
    If a version was not built for a particular platform, mention why not.
-   Link to the results page of the VMU tests.
    Make sure the tests actually installed the version to be promoted.
    Explain any failures.
-   If you ran manual tests, summarize your tests and findings, and explain any failures.

For example (hypothetical promotion request for XRootD):

> VMU test results are at <LINK TO TEST RESULTS PAGE\>;
> tests passed except for StashCache tests on EL10 because the package is
> not available on that platform.  I tested turning on the new knob in a container
> and was able to pull a file from the server successfully.
> @Software and Release, permission to promote `xrootd-5.9.9-9` to `24-main-testing`
> and `25-main-testing`?

### Promoting

Follow these steps to request promotion, promote a package, and note the promotion in JIRA:

1.  Make sure the package update has at least one associated Jira ticket;
    if there is no ticket, create one for releasing the package(s).
2.  Obtain permission to promote the package(s) from one of the Software Team members,
    by mentioning "@Software and Release" in the Jira ticket.
3.  Use [osg-promote](../software/osg-build-tools.md#osg-promote) to promote the package(s) from development to testing
4.  Comment on the associated Jira ticket(s) with osg-promote's Jira-formatted output (or at least the build NVRs) and,
    if you know, suggestions for acceptance testing.
6.  Update the Jira ticket description with a short blurb about the important changes in the new version vs. the old version,
    so it can be used in the release notes.
7.  Mark the Jira ticket for the package as “Ready For Testing”;
    tickets related to this version (e.g. for specific changes) should also be marked “Ready For Testing”.
