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
    [the UW AFS upstream area](../software/rpm-development-guide.md#upstream-source-cache)
2.  In [a checkout of our packaging code](../software/rpm-development-guide.md#revision-control-system),
    update [the reference to the upstream file](../software/rpm-development-guide.md#upstream) and,
    as needed, [the RPM spec file](../software/rpm-development-guide.md#osg)
3.  Use [osg-build](../software/osg-build-tools.md#osg-build) to perform a scratch build of the updated package
4.  Verify that the build succeeded; if not, redo previous steps until success
5.  Optionally, lightly test the new RPM(s); if there are problems, redo previous steps until success
6.  Use [osg-build](../software/osg-build-tools.md#osg-build) to perform an official build of the updated package
    (which will go into the development repos)
7.  Perform standard developer testing of the new RPM(s) — see below for details
8.  Obtain permission from the Software Manager to promote the package
9.  Promote the package to testing — see below for details

Versioning Guidelines
---------------------

OSG-owned software should contain three digits, X.Y.Z, where X represents the major version, Y the minor version,
and Z the maintenance version.
New releases of software should increment one of the major, minor, or maintenance according to the following guidelines:

-   **Major:** Major new software, typically (but not limited to) full rewrites, new architectures, major new features;
    can certainly break backward compatibility (but should provide a smooth upgrade path). Worthy of introduction into Upcoming.
-   **Minor:** Notable changes to the software, including significant feature changes, API changes, etc.;
    may break compatibility, but must provide an upgrade path from other versions within the same Major series.
-   **Maintenance:** Bug fixes, minor feature tweaks, etc.;
    must not break compatibility with other versions within the same Major.Minor series.

If you are unsure about which version number to increment in a software update, consult the Software Manager.

Build Procedures
----------------

<!-- TODO not written yet
### Verifying builds through GitHub Actions
-->

### Building packages for multiple OSG release series

The OSG Software team supports multiple release series, independent but in parallel to a large degree.
In many cases, a single package is the same across release series, and therefore we want to build the package once
and share it among the series.
The procedure below suggests a way to accomplish this task.

Current definitions:

- maintenance: OSG 3.4 ( **trunk** )
- current: OSG 3.5 ( **branches/osg-3.5** )

<!--
- future: OSG 3.6 ( *branches/osg-3.6* )
-->

Procedure:


1.  Make changes to **trunk**
2.  Optionally, make and test a scratch build from **trunk**
3.  Commit the changes
4.  Make an official build from **trunk** (e.g.: `osg-build koji <PACKAGE>`)
5.  Perform the standard 4 tests for the **current** series (see below)
6.  Merge the relevant commits from **trunk** into the **maintenance** branch (see below for tips)
7.  Optionally, make and test a scratch build from the **maintenance** branch
8.  Commit the merge
9.  Make an official build from the **maintenance** branch (e.g.: `osg-build koji --repo=3.4 <PACKAGE>`)
10. Perform the standard 4 tests for the **maintenance** series (see below)
11. As needed (or directed by the Software manager), perform the cross-series tests (see below)

!!! note
    Do not change the RPM Release number in the **maintenance** branch before rebuilding;
    the `%dist` tag will differ automatically, and hence the **maintenance** and **current** NVRs will not conflict.

<!--
1. Make changes to the *trunk*
2. Optionally, make and test a scratch build from the *trunk*
3. Commit the changes
4. Make an official build from the *trunk* (e.g.: `osg-build koji <PACKAGE>`)
5. Perform the standard 4 tests for the *current* series (see below)
6. Merge the relevant commits from the *trunk* into the *future* branch (see below for tips)
7. Optionally, make and test a scratch build from the *future* branch
8. Commit the merge
9. Make an official build from the *future* branch (e.g.: `osg-build koji --repo=3.6 <PACKAGE>`)
10. Perform install tests for the *future* series (see below)
11. As needed (or directed by the Software manager), perform the cross-series tests (see below)
-->

### Merging changes from one release series to another

These instructions assume that you are merging from `trunk` to `branches/osg-3.5`.
They also assume that the current directory you are in is a checkout of `branches/osg-3.5`.
I will use `$pkg` to refer to the name of your package.

First, you will need the commit numbers for your changes:

    svn log \^/native/redhat/trunk/$pkg | less

Write down the commits you want to merge.

If you only have one commit, merge that commit with -c as follows:

    svn merge -c $commit_num \^/native/redhat/trunk/$pkg $pkg

Where `$commit_num` is the SVN revision number of that commit (e.g. 17000).
Merging an individual change like this is referred to as "cherry-picking".

If you have a range of commits and you wish to merge all commits within that range, then do the following:

    svn merge -r $start_num:$end_num \^/native/redhat/trunk/$pkg $pkg

Where `$start_num` is the SVN revision of the commit *BEFORE* your first commit,
and `$end_num` is the SVN revision of your last commit in that range.
**Note:** Be very careful when merging a range from trunk into the maintenance branch
so that you do not introduce more changes to the maintenance branch than are necessary.

If you have multiple commits but they are not contiguous (i.e. there are commits made by you or someone else in that range
that you do not want to merge), you will need to cherry-pick each individual commit.

    svn merge -c $commit1 \^/native/redhat/trunk/$pkg $pkg
    svn merge -c $commit2 \^/native/redhat/trunk/$pkg $pkg
    ...

Where `$commit1`, `$commit2` are the commit numbers of the individual changes.

Note that merge tracking in recent versions of SVN (1.5 or newer) should prevent commits from accidentally being merged multiple times.
You should still look out for conflicts and examine the changes via `svn diff` before committing the merge.

Testing Procedures
------------------

Before promoting a package to a testing repository, each build must be tested lightly from the development repos
to make sure that it is not completely broken, thereby wasting time during acceptance testing.
Normally, the person who builds a package performs the development testing.

**If you are not doing your own development testing for a package**, contact the Software Manager
and/or leave a comment in the associated ticket; otherwise, your package may never be promoted to testing and hence never released.

### The "Standard 4" tests, defined

In most cases, the Software manager will ask a developer to perform the “standard 4” tests
on an updated package in a release series before promotion.
This is a shorthand description for a standard set of 4 test runs:

-   Fresh install on el6
-   Fresh install on el7
-   Update install on el6
-   Update install on el7

An “update install” is a fresh install of the relevant package (or better yet, metapackage that includes it)
**from the production repository**, followed by an update to the new build **from the development repository**.

For each test run, the amount of functional testing required will vary.

-   For very simple changes, it may be sufficient to verify that each installation succeeds and that the expected files are in place
-   For some changes, it may be sufficient to run osg-test on the resulting installation
-   For some changes, it will be necessary to perform careful functional tests of the affected component(s)

If you have questions, check with the Software Manager to determine the amount of testing that is required per test run.

### The "Cross-Series" test, defined

The cross-series test may need to be run for packages that have been built for multiple release series of the OSG software stack (i.e. 3.4 and 3.5):

-   On el7, install from the 3.4 repositories, then update from the 3.5 repositories

Viewed another way, this test is similar to the update installs, above, except from 3.4-release to 3.5-development.

### The "Long Tail" tests, defined

These tests may need to be run when updating a package that's also in the old, unsupported (3.3) branch. They will consist of:

-   Install from 3.3-release and update to 3.5-development (on el7 only)

### The "full set of tests", defined

All of the tests mentioned above.

### Running the tests in VM Universe

In the case that the package you're testing is covered by osg-tested-internal,
you can run the full set of tests in a manual VM universe test run.
Make sure you meet the [pre-requisites](https://github.com/opensciencegrid/vm-test-runs) required
to submit VM Universe jobs on `osghost.chtc.wisc.edu`.
After that's done, prepare the test suite with a comment describing the test run.
For example, if you were testing a new `htcondor-ce` package:

``` console
osg-run-tests 'Testing htcondor-ce-3.2.1-1'
```

After you `cd` into the directory specified in the output of the previous command,
you will need to edit the `*.yaml` files in `parameters.d` to reflect the tests that you will want to run,
i.e. clean installs, upgrade installs and upgrade installs between OSG versions.

Once you're satisfied with your list of parameters, submit the dag:

``` console
condor_submit_dag master-run.dag
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
A promotion request goes into at least one affected JIRA ticket and will be answered there as well.
Below are some tips for writing a good promotion request:

-   Make sure that relevant information about goals, history, and resolution is in the associated ticket(s)
-   Include globs for the NVRs to be promoted (or a detailed list, if it is that complicated, which it almost never is)
-   If you ran automated tests:
    -   Link to the results page(s)
    -   Verify that relevant tests ran successfully (as opposed to being skipped or failing) – briefly summarize your findings
    -   Note whether the automated tests are just regression tests or actually test the current change(s)
    -   If there are **any** failures, explain why they are not important to the promotion request
-   If you ran manual tests:
    -   Summarize your tests and findings
    -   If there were failures, explain why they are not important to the promotion request
-   If there are critical build dependencies that we typically check, include reports from the `built-against-pkgs` tool
    -   Note: This step is really just for known, specific cases, like the {HTCondor, BLAHP} set
    -   Occasionally, the OSG Software manager will request the tool to be run for other cases
-   If other packages depend on the to-be-promoted package, explain whether the dependent packages must be rebuilt or, if not, why not

For example (hypothetical promotion request for HTCondor-CE):

> May I promote `htcondor-ce-2.3.4-2.osg3*.el*`? I ran a complete set of automated tests <LINK THE PRECEDING TEXT OR SEPARATELY HERE\>;
> the HTCondor-CE tests ran and passed in all cases. There were some spurious failures of RSV in the All condition for RHEL 6,
> but this is a known failure case that is independent of HTCondor-CE. I also did a few spot checks manually
> (one VM each for SL 6 and SL 7), and in each case setting `use_frobnosticator = true` in the configuration resulted in
> the expected behavior as defined in the description field above.
> The `built-against-pkgs` tool shows that I built against all the latest HTCondor and BLAHP builds, see below.
> <JIRA-formatted table comes after\>

### Promoting

Follow these steps to request promotion, promote a package, and note the promotion in JIRA:

1.  Make sure the package update has at least one associated JIRA ticket;
    if there is no ticket, at least create one for releasing the package(s)
2.  Obtain permission to promote the package(s) from the Software Manager (see above)
3.  Use [osg-promote](../software/osg-build-tools.md#osg-promote) to promote the package(s) from development to testing
4.  Comment on the associated JIRA ticket(s) with osg-promote's JIRA-formatted output (or at least the build NVRs) and,
    if you know, suggestions for acceptance testing
1.  Update the JIRA ticket description with a bulleted list describing changes in the promoted version(s) compared to
    the currently released version(s)
5.  Mark each associated JIRA ticket as “Ready For Testing”
