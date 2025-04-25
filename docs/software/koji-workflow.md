Koji Workflow
=============

This covers the basics of using and understanding the [OSG Koji](https://koji.osg-htc.org/koji) instance. It is meant primarily for OSG Software team members who need to interact with the service.

Terminology
-----------

Using and understanding the following terminology correctly will help in the reading of this document:

**Package**  
This refers to a named piece of software in the Koji database. An example would be "lcmaps".

**Build**  
A specific version and release of a package, and an associated state. A build state may be successful (and contain RPMs), failed, or in-progress. A given build may be in one or more tags. The build is associated with the output of the latest build task with the same version and release of the package.

**Tag**  
A named set of packages and builds, parent tags, and reference to external repositories. An example would be the "osg-3.3-el6-development" tag, which contains (among others) the "lcmaps" package and the "lcmaps-1.6.6-1.1.osg33.el6" build. There is an inheritance structure to tags: by default, all packages/builds in a parent tag are added to the tag. A tag may contain a reference to (possibly inherited) external repositories; the RPMs in these repositories are added to repositories created from this tag. Examples of referenced external repositories include CentOS base, EPEL, or JPackage.

!!! note
    A tag is NOT a yum repository.

**Target**  
A target consists of a build tag and a destination tag. An example is "osg-3.3-el6", where the build tag is "osg-3.3-el6-build" and the destination tag is "osg-3.3-el6". A target is used by the build task to know what repository to build from and tag to build into.

**Task**  
A unit of work for Koji. Several common tasks are:

-   build  
    This task takes a SRPM and a target, and attempts to create a complete Build in the target's destination tag from the target's build repository. This task will launch one buildArch task for each architecture in the destination tag; if each subtask is successful, then it will launch a tagBuild subtask.

    !!! note
        If the build task is marked as "scratch", then it won't result in a saved Build.

-   buildArch  
    This task takes a SRPM, architecture name, and a Koji repository as an input, and runs `mock` to create output RPMs for that arch. The build artifacts are added to the Build if all buildArch tasks are successful.

-   tagBuild  
    This adds a successful build to a given tag.

-   newRepo  
    This creates a new repository from a given tag.

**Build artifacts**  
The results of a buildArch task. Their metadata are recorded in the Koji database, and files are saved to disk. Metadata may include checksums, timestamps, and who initiated the task. Artifacts may include RPMs, SRPMs, and build logs.

**Repository**  
A yum repository created from the contents of a tag at a specific point in time. By default, the yum repository will contain all successful, non-blocked builds in the tag, plus all RPMs in the external repositories for the tag.


Obtaining Access
----------------

Building OSG packages in Koji requires these privileges:

-   access to the software-packaging Git repository at https://github.com/osg-htc/software-packaging
-   access to `osgsw-ap.chtc.wisc.edu` for uploading to the upstream sources directory
-   access to Koji via a Kerberos credential

Send email to <help@osg-htc.org> with the subject "Requesting access to Koji" with the following information:
-   your GitHub username
-   your Kerberos principal


Initial Setup
-------------

You will be using the [OSG Build Tools](../software/osg-build-tools.md) to interact with Koji.
See the installation guide on that page for getting started.

Using Koji
----------

### Creating a new build

We create a new build in Koji from the package's directory in OSG Software subversion.

If a successful build already exists in Koji (regardless of whether it is in the tag you use), you cannot replace the build. Two builds are the same if they have the same NVR (Name-Version-Release). You *can* do a "scratch" build, which recompiles, but the results are not added to the tag. This is useful for experimenting with koji.

To do a build, execute the following command from within the OSG Software subversion checkout:

```console
[you@host]$ osg-build koji <PACKAGE NAME>
```

To do a scratch build, simply add the `--scratch` command line flag.

When you do a non-scratch build, it will build with the *osg-el6* and *osg-el7* targets. This will assign your build the *osg-3.4-el6-development* and *osg-3.4-el7-development* tags (and your package will be assigned the *osg-el6* and *osg-el7* tags). If successful, your build will end up in the Koji *osg-minefield* yum repos and will eventually show up in the *osg-development* yum repos. This is a high latency process.

### Build task Results

#### How to find build results

The most recent build results are always shown on the home page of Koji:

<https://koji.osg-htc.org/koji/index>

Clicking on a build result brings you to the build information page. A successful build will result in the build page having build logs, RPMs, and a SRPM.

If your build isn't in the recent list, you can use the search box in the upper-right-hand corner. Type the exact package name (or use a wildcard), and it will bring up a list of all builds for that package. You can find your build from there. For example, the "lcmaps" package page is here:

<https://koji.osg-htc.org/koji/packageinfo?packageID=56>

And the lcmaps-1.6.6-1.1.osg33.el6 build is here:

<https://koji.osg-htc.org/koji/buildinfo?buildID=7427>

#### Trying out your build

Because it takes a while for your build to get into one of the regular repositories, it's simplest to download your RPM directly (see the previous section on How to find build results), and install it with:

```console
[root@host]# yum localinstall <RPM>
```

#### How to get the resulting RPM into a repository

Once a package has been built, it is added to a tag. We then must turn the tag into a yum repository. This is normally done automatically and you do not need to deal with it yourself. Three notes:

-   The kojira daemon creates a repository automatically post-build on the koji-hub host. Eventually, the development repository will be the one hosted by koji-hub.
-   The koji-hub repository can be created manually by running

        :::console
        [you@host]$ osg-koji regen-repo <TAG NAME>

    For example, the tag name for osg-development in 3.4 on el6 is "osg-3.4-el6-development". Likely, you won't need to do this when kojira is working.
-   Repositories are created on external hosts with the `mash` tool. These are usually triggered by cron jobs, but may be run by hand too. Documentation for running mash is on the TODO list.
    -   You can create your own personal repository using `mash`.

#### Debugging build issues

-   Failed build tasks can be seen from the Koji homepage. The logs from the tasks are included. Relevant logs include:

    -   `root.log`  
        This is the log of mock trying to create an appropriate build root for your RPM. This will invoke yum twice: once to create a generic build root, once for all the dependencies in your BuildRequires. All RPMs in your build root will be logged here. If mock is unable to create the build root, the reason will show up here.

        -   404 Errors

            If you see `Error downloading packages` and `HTTP Error 404 - Not Found` errors in your `root.log`,
            this commonly indicates that an rpm repo mirror was updated and our build repo is out-of-date.
            This can be fixed by regenerating the relevant build repos for your builds.

            This is usually something like `osg-3.4-el7-build` or `osg-upcoming-el7-build`;
            but you can find the exact build tag by clicking the Build Target link for the koji task,
            and whatever is listed for the Build Tag is the name of the repo to regen.

            Regenerate each repo that failed with 404 errors:

                :::console
                $ osg-koji regen-repo <BUILD TAG>

    -   `build.log`  
        The output of the rpmbuild executable. If your package fails to compile, the reason will show up here.

-   One input to the buildArch task is a repository, which is based on a Koji tag. If the repository hasn't been recreated for a dependency you need (for example, when kojira isn't working), you may not have the right RPMs available in your build root.
    -   One common issue is building a chain of dependencies. For example, suppose build B depends on the results of build A. If you build A then build B immediately, B will likely fail. This is because A is not in the repository that B uses. The proper string of events building A, starting the regeneration of the destination and build repo (which should happen in a few minutes of the build A task completing), then submitting build task B.

        !!! note
            if you submit build task B while the build repository task is open, it will not start until the build task has finished.

- Other errors

    -   `package <PACKAGE NAME> not in list for tag <TAG>`<br/>
        This happens when the name of the directory your package is in does not match the name of the package.
        You must rename one or the other and commit your changes before trying again.

### Promoting Builds from Development -> Testing

Software team members can promote any package to testing.

To promote from development to testing:

#### Using *osg-promote*

Before using `osg-promote`, [authenticate to Koji as above](#authenticating-to-koji).

If you want to promote the latest version:

```console
[you@host]$ osg-promote -r <OSGVER>-testing <PACKAGE NAME>
```
&lt;PACKAGE NAME&gt; is the bare package name without version, e.g. `gratia-probe`.


If you want to promote a specific version:

```console
[you@host]$ osg-promote -r <OSGVER>-testing <BUILD NAME>
```
&lt;BUILD NAME&gt; is a full `name-version-revision.disttag` such as `gratia-probe-1.17.0-2.osg33.el6`.


&lt;OSGVER&gt; is the OSG major version that you are promoting for (e.g. `3.4`).

`osg-promote` will promote both the el6 and el7 builds of a package. After promoting, copy and paste the JIRA code `osg-promote` produces into the JIRA ticket that you are working on.

 For `osg-promote`, you may omit the `.osg34.el6` or `.osg34.el7`; the script will add the appropriate disttag on.

See [OSG Building Tools](../software/osg-build-tools.md) for full details on `osg-promote`.

### Creating custom koji areas

Occasionally you may want to make builds of a package (or packages) which you
do not yet want to go into the main development repos.  In this case, you can
create a set of custom koji tags and build targets for these builds.  We have
a script in our
[osg-next-tools](https://github.com/opensciencegrid/osg-next-tools/) repo
called
[new-koji-area](https://github.com/opensciencegrid/osg-next-tools/blob/master/koji/new-koji-area)
that facilitates this set up.

Further reading
---------------

-   Official Koji documentation: <https://docs.pagure.org/koji/>
-   Fedora's koji documentation: <https://fedoraproject.org/wiki/Koji>
-   Fedora's "Using Koji" page: <https://docs.fedoraproject.org/en-US/package-maintainers/Using_the_Koji_Build_System/> Note that some instructions there may not apply to OSG's Koji. For the most part though, they are useful.
