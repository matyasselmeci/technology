
OSG Build Tools
===============

This page documents the tools used for RPM development for the OSG Software Stack. See [the RPM development guide](../software/rpm-development-guide.md) for the principles on which these tools are based.

The tools are available in Git in [osg-htc/osg-build on GitHub](https://github.com/osg-htc/osg-build).
First, acquire the tools via one of the following methods:

- [Apptainer/Singularity](#install-apptainer)
- [Docker/Podman](#install-docker)
- Local install
  - [Rootly install with Make](#install-local-root)
  - [Install via Pip](#install-local-pip)


<a id="install-apptainer"></a>
Quick start with Apptainer
--------------------------
This quick start guide shows how to use the OSG build tools via Apptainer. (Singularity should work too.)
This assumes you will use Kerberos for authentication with UW-Madison or Fermilab credentials.
To get UW-Madison Kerberos credentials, you will need to be inside the UW Campus network, either via campus wifi or VPN.

In addition, this assumes that the repository you want to build packages from is from a subdirectory under your home directory.
If not, pass the appropriate `--bind` (`-B`) argument to the `apptainer run` command.

The image is available from hub.opensciencegrid.org.  Pull the image and run it:
```
apptainer pull oras://hub.opensciencegrid.org/osg-htc/osg-build:v2-sif osg-build.sif
apptainer run osg-build.sif
```
you will be in a shell inside the image.

Set up your configuration for accessing the OSG Koji environment.
```
osg-koji setup
```
Select Kerberos as your auth method.
Use `kinit` to get a credential.

For UW-Madison:
```
kinit <username>@AD.WISC.EDU
```
replacing `<username>` with the name of your UW NetID.

For Fermilab:
```
kinit <username>@FNAL.GOV
```
replacing `<username>` with your Fermilab user name.

Verify that you can successfully authenticate to Koji:
```
osg-koji hello
```

If that succeeds, you will be ready to use the rest of the `osg-build` and `osg-koji` commands.

Note: the `mock` command does not work in Apptainer/Singularity due to permissions issues.
For testing builds, use `osg-build koji --scratch` instead.


<a id="install-docker"></a>
Quick start with Docker
-----------------------
!!!note 
    The Docker image and helper scripts are a work in progress.

This quick start guide shows how to use the OSG build tools via Docker. (Podman should work too.)
This assumes you will use Kerberos for authentication with UW-Madison or Fermilab credentials.
To get UW-Madison Kerberos credentials, you will need to be inside the UW Campus network, either via campus wifi or VPN.

First, pull the GitHub repo hosting the image and helper scripts.

```
git clone https://github.com/osg-htc/docker-osg-build
```

Next, run the `initbuilder` script, giving it the directory under which the repo containing the software packaging is.
For example, if you have your checkout of the [https://github.com/osg-htc/software-packaging](software-packaging repo)
under `~/software-packaging`, run

```
cd docker-osg-build
./initbuilder ~/software-packaging
```
Answer the configuration questions from initbuilder.

From then on, use the `osg-build-shell` command to enter into a shell inside the image.

Use `kinit` to get a credential.

For UW-Madison:
```
kinit <username>@AD.WISC.EDU
```
replacing `<username>` with the name of your UW NetID.

For Fermilab:
```
kinit <username>@FNAL.GOV
```
replacing `<username>` with your Fermilab user name.

Verify that you can successfully authenticate to Koji:
```
osg-koji hello
```

If that succeeds, you will be ready to use the rest of the `osg-build` and `osg-koji` commands.


Installation into a Linux system
--------------------------------

You may install the software locally, either via `pip` or via a script distributed by an RPM.
If using `pip`, you will need to install some dependencies by hand.


<a id="install-local-root"></a>
### Install as root via OSG RPM (EL8, EL9, EL10)

The OSG 24-internal repositories contain a package called "osg-build-deps".
[Install the OSG 24 repositories](https://osg-htc.org/docs/common/yum/).

Then, install osg-build-deps:
```
yum install --enablerepo=osg-internal-development osg-build-deps
```
This will pull in the dependencies necessary to run the OSG Build Tools.
Then, to install the OSG Build Tools themselves into `/usr/local`, run
```
install-osg-build.sh
```
This will clone osg-build into `/usr/local/src/osg-build` and install the software under the `/usr/local` tree.


<a id="install-local-pip"></a>
### Install via pip (EL8, EL9, EL10, Ubuntu, others)

To use Kerberos authentication you will need the client tools to run `kinit`.
On Enterprise Linux variants and Feodra, run:
```
yum install krb5-workstation
```
On Ubuntu, run:
```
apt install krb5-user
```

Before installing via pip, you will need to install the `requests-gssapi` library by hand because it contains compiled binaries.

On EL8/EL9/EL10, run
```
yum install python3-requests-gssapi
```
on Ubuntu 24.04, run
```
apt install python3-requests-gssapi
```
If `requests-gssapi` is not available, you will have to compile it by hand; see below.

To install the OSG Build Tools themselves, run:

```
pip install --user git+https://github.com/osg-htc/osg-build@V2-branch
```

!!! note
    If you installed requests-gssapi via RPM/Deb, and you want to run osg-build in a virtualenv or via pipx,
    pass `--system-site-packages` when creating the virtualenv or running `pipx install`.


#### Building requests-gssapi from source

If the `requests-gssapi` library is not available or you don't want to install it via a package manager,
you must install the dependencies that will allow pip to build it from source.

On EL8/EL9 run:
```
yum install gcc make python3-devel krb5-devel
```

On Ubuntu run
```
apt install gcc make libpython-dev libkrb5-dev
```

Afterwards, pip install the OSG Build Tools as previously:

```
pip install --user git+https://github.com/osg-htc/osg-build@V2-branch
```


Configuring Koji access and OSG Build
-------------------------------------
Access to Koji is performed using Kerberos.
Set up your configuration for accessing the OSG Koji environment.
```
osg-koji setup
```
You will be prompted for your authentication mehtod; select Kerberos and enter your Kerberos principal at the prompt.

Use `kinit` to get a credential.

For UW-Madison:
```
kinit <username>@AD.WISC.EDU
```
replacing `<username>` with the name of your UW NetID.

For Fermilab:
```
kinit <username>@FNAL.GOV
```
replacing `<username>` with your Fermilab user name.

Verify that you can successfully authenticate to Koji:
```
osg-koji hello
```
If authentication is successful, you are ready to perform Koji build tasks.

Commonly used tools
-------------------

These are the tools you will be using for day-to-day builds.

### osg-build

#### Overview

This is the primary tool used in building source and binary RPMs.

> `osg-build <TASK> [options] <PACKAGE DIRECTORY> [...]`

*package\_directory* is a directory containing an `osg/` and/or an `upstream/` subdirectory. See [the RPM development guide](../software/rpm-development-guide.md) for how these directories are organized.

#### Tasks

##### koji

Builds the package on the Koji build service hosted at UW-Madison. <https://koji.osg-htc.org>.
You must specify the set of destination repos with `--repo`, described later.

##### lint

Runs `rpmlint` the package(s) to check for various problems. You will need to have `rpmlint` installed.

##### mock

Prebuilds the final source package, then builds it locally using `mock`, and stores the resulting source and binary RPMs in the package-specific `_build_results` directory.

##### prebuild

Prebuilds the final source package from upstream sources (if any) and local files (if any). May create or overwrite the `_upstream_srpm_contents` and `_final_srpm_contents` directories.

##### prepare

Prebuilds the final source package, then calls `rpmbuild -bp` on the result, extracting and patching the source files (and performing any other steps defined in the `%prep` section of the spec file. The resulting sources will be under `_final_srpm_contents`.

##### rpmbuild

Prebuilds the final source package, then builds it locally using `rpmbuild`, and stores the resulting source and binary RPMs in the package-specific `_build_results` directory.

##### quilt

Collects the upstream local sources and spec file, then calls `quilt setup` on the spec file, extracting the source files and adding the patches to a quilt series file. See [Quilt documentation (PDF link)](http://www.shakthimaan.com/downloads/glv/quilt-tutorial/quilt-doc.pdf) for more information on quilt; also look at the example in the Usage Patterns section below. 
The sources are unpacked into `_quilt`; they will be in pre-patch state and the various quilt commands can be used to apply and modify patches. Requires `quilt`.

#### Options

This section lists the common command-line options.

##### --help

Prints the built-in usage information and exits without doing anything else.

##### --version

Prints the version of `osg-build` and exits without doing anything else.

#### Options common to all tasks

##### --el8, --el9, etc.

Build for these specific distro versions.

##### -q, --quiet, -v, --verbose

Decrease or increase the amount of information printed.

#### Options specific to prebuild task

##### --full-extract

If set, all upstream tarballs will be extracted into `_upstream_tarball_contents/` during the prebuild step. This flag is now mostly redundant with the `prepare` and `quilt` tasks.

#### Options specific to rpmbuild and mock tasks

##### --distro-tag *dist*

Sets the `%dist` tag added on to the end of the release in the RPM ( `rpmbuild` and `mock` tasks only ).

##### -t, --target-arch *arch*

Specify an architecture to build packages for ( `rpmbuild`, `mock`, and scratch `koji` builds only ).

#### Options specific to mock task

##### --mock-clean, --no-mock-clean

Enable/disable deletion of the mock buildroot after a successful build.

Default is `true`.

##### -m, --mock-config *path*

Specifies the `mock` configuration file to use. This file details how to set up the build environment used by mock for the build, including Yum repositories from which to install dependencies and certain predefined variables (e.g., the distribution tag `%dist`).

See also `--mock-config-from-koji`.

##### --mock-config-from-koji *build tag*

Creates a mock config from a Koji build tag. This is the most accurate way to replicate the build environment that Koji will provide (outside of Koji). The build tag is based on the distro version (el6, el7) and the OSG major version (3.3, 3.4). For 3.4 on el6, it is: `osg-3.4-el6-build` Also requires the Koji command-line tools (package `koji`), obtainable from the osg repositories. Since this uses koji, some of the koji-specific options may apply, namely: `--koji-backend`, `--koji-login`, and `--koji-wrapper`.

#### Options specific to koji task

##### --dry-run

Do not actually run koji, merely show the command(s) that will be run. For debugging purposes.

##### --getfiles, --get-files

For scratch builds without `--vcs` only. Download the resulting RPMs and logs from the build into the `_build_results` directory.

##### --koji-target *target*

The koji target to use for building.

##### --koji-tag *tag*

The koji tag to add packages to. See the [Koji Workflow guide](../software/koji-workflow.md) for more information on the terminology. The special value `TARGET` uses the destination tag defined in the koji target.

Default is `osg-el6` or `osg-el7`.

##### --ktt, --koji-tag-and-target *arg*

Shorthand for setting both --koji-tag and --koji-target to *arg*.


##### --regen-repos

Start a `regen-repo` koji task on the build tag after each koji build, to update the build repository used for the next build. Not useful unless you are launching multiple builds. This enables you to launch builds that depend on each other. Doesn't work too well with `--no-wait`, since the next build may be started before the regen-repo task is complete. Waiting will keep the next build from being queued until the regen-repo is complete.

##### --scratch, --no-scratch

Perform scratch builds. A scratch build does not go into a repository, but the name-version-release (NVR) of the created RPMs are not considered used, so the build may be modified and repeated without needing a release bump. This has the same use case as the mock task: creating packages that you want to test before releasing. If you do not have a machine with mock set up, or want to test exactly the environment that Koji provides, scratch builds might be more convenient.

##### --vcs, --no-vcs, --svn, --no-svn

Have Koji check the package out from a version control system instead of creating an SRPM on the local machine and submitting that to Koji. Currently, SVN and Git are supported. If this flag is specified, you may use SVN URLs or URL@Revision pairs to specify the packages to build. You may continue specify package directories from an SVN checkout, in which case osg-build will use `svn info` to find the right URL@Revision pair to use and warn you about uncommitted changes. osg-build will also warn you about an outdated working directory.

`--vcs` defaults to `true` for non-scratch builds, and `false` for scratch builds.

##### --repo=*destination repository*

Selects the repository (24-main, 24-upcoming, etc.) to build packages for.
This is required for Koji builds.  See `--repo-list` for a list of repositories.

##### --repo-list

Lists the available repositories for the `--repo` argument.


### osg-koji

This is a wrapper script around the `koji` command line tool.
It automatically specifies parameters to access the OSG's koji instance.

An additional command, `osg-koji setup` exists, which sets up koji configuration in `~/.osg-koji`.


### osg-promote

#### Overview

Run this script to push packages from one set of repos to another (e.g. from development to testing), according to the OSG software promotion guidelines.

Once the packages are promoted, the script will generate code to cut and paste into a JIRA comment.

#### Synopsis

> `osg-promote [-r|--route <ROUTE>]... [options] <PACKAGE OR BUILD> [...]`

#### Examples

- Promote the latest build of `osg-version` to testing for the current release series  

        osg-promote -r testing osg-version

- Promote the latest builds of `osg-ce` to testing for the 3.3 and 3.4 release series  

        osg-promote -r 3.3-testing -r 3.4-testing osg-ce

- Promote `osg-build-1.5.0-1` to testing for the current release series  

        osg-promote -r testing osg-build-1.5.0-1

#### Arguments

##### -h

Display help and a list of valid routes.

##### *package or build*

A package (e.g. `osg-version`) or build (e.g. `osg-version-3.3.0-1.osg33.el6`) to promote. You may omit the dist tag (the `.osg33.el6` part).

If a package is specified, the most recent version of that package will be promoted.

If a build is specified, that build and the build that has the same *version*-*release* for the other distro version(s) will be promoted. That is, if you specify the route `3.3-testing` and the build `foo-1-1`, then `foo-1-1.osg33.el6` and `foo-1-1.osg33.el7` will be promoted.

This may be specified multiple times, to promote multiple packages. The NVRs of each set of builds for a package *must* match.

##### -r *ROUTE*, --route *ROUTE*

The promotion route to use. Use `osg-promote -h` to get a list of valid routes. This may be specified multiple times. For example, to promote for both 3.4 and 3.3, pass `-r 3.4-testing -r 3.3-testing`.

If not specified, the `testing` route is used, which corresponds to the testing route for the latest release series.

##### -n, --dry-run

Do not promote, just show what would be done.

##### --el6-only / --el7-only

Only promote packages for el6 / el7.

##### --no-el6 / --no-el7

Do not promote packages for el6 / el7.

##### --ignore-rejects

Ignore rejections due to version mismatch between dvers or missing package for one dver.

##### --regen

Regenerate the destination repos after promoting.

##### -y, --assume-yes

Do not prompt before promotion.


Minor tools
-----------

These tools are used less frequently or are for specialized purposes.

### koji-tag-diff

This script displays the differences between the latest packages in two koji tags.

Example invocation: `koji-tag-diff osg-3.4-el6-development osg-3.4-el7-testing`

This prints the packages that are in osg-3.4-el6-development but not in osg-3.4-el7-testing, or vice versa.

### osg-build-test

This script runs automated tests for `osg-build`. Only a few tests have been implemented so far.

### osg-import-srpm

This is a script to fetch an SRPM from a remote site, copy it into the upstream cache on AFS, and create an SVN package dir (if needed) with an `upstream/*.source` file. By default it will put downloaded files into the VDT upstream cache (/p/vdt/public/html/upstream), but you can pass `--upstream-root=<UPSTREAM DIR>` to put them somewhere else. If called with the `--extract-spec` or `-e` argument, it will extract the spec file from the SRPM and place it into the `osg` subdir in SVN. If called with the `--diff-spec` or `-d` argument, it will extract the spec file and compare it to the existing spec file in the `osg` subdir. **The script hasn't been touched in a while and needs a good deal of cleanup.** A planned feature is to allow doing a three-way diff between the existing RPM before OSG modifications, the new RPM before OSG modifications and the existing RPM after OSG modifications.

Common Usage Patterns
---------------------

#### Verify that all files necessary to build the package are in the right place

Run `osg-build prebuild <PACKAGEDIR>`.

#### Fetch and extract all source files for examination

Run `osg-build prebuild --full-extract <PACKAGEDIR>`. Look inside the `_upstream_tarball_contents` directory.

#### Get a post-patch version of the upstream sources for examination

Run `osg-build prepare <PACKAGEDIR>`. Look inside the `_build_results` directory.

#### See which patches work with a new version of a package, update or remove them

1.  Place the new source tarball into the upstream cache, edit the version in the spec file and \*.sources files as necessary
2.  Run `osg-build quilt <PACKAGEDIR>`.
3.  Enter the extracted sources inside the `_final_srpm_contents` directory. You should see a file called `series` and a symlink called `patches`.
4.  Type `quilt series` to get a list of patches in order of application.
5.  Type `quilt push` to apply the next patch.
    -   If the patch applies cleanly, continue.
    -   If the patch applies with some fuzz, type `quilt refresh` to update the offsets in the patch.
    -   If the patch does not apply and you wish to remove it, type `quilt delete <PATCH NAME>` (delete only removes it from the series file, not the disk)
    -   If the patch does not apply and you wish to fix it, either type `quilt push -f` to interactively apply the patch, or `quilt delete <PATCH NAME>` the patch and use `quilt new` / `quilt edit` / `quilt refresh` to edit files and make a new patch from your changes. Consult the `quilt(1)` manpage for more info.
6.  If you have a new patch, run `quilt import <PATCHFILE>` to add the patch to the series file, and run `quilt push` to apply it.
7.  If you have changes to make to the source code that you want to save as a patch, type `quilt new <PATCHNAME>`, edit the files, type `quilt add <FILE>` on each file you edited, then type `quilt refresh` to recreate the patch.
8.  Once you're all done, copy the patches in the `patches/` directory to the `osg/` dir in SVN, run `quilt series` to get the application order and update the spec file accordingly.

#### See if a package builds successfully for OSG 24 main

-   Run `osg-build koji --scratch <PACKAGEDIR> --repo 24-main`. You may download the resulting RPMs from kojiweb <https://koji.osg-htc.org/koji> or pass `--getfiles` to `osg-build koji` and they will get downloaded to the `_build_results` directory.

#### Check for potential errors in a package

Run `osg-build lint <PACKAGEDIR>`.

#### Create and test a final build of a package for all platforms for upcoming

1.  `svn commit` your changes in `branches/upcoming`.
2.  Type `osg-build koji --repo=upcoming <PACKAGEDIR>`
3.  Wait for the `osg-upcoming-minefield` repos to be regenerated containing the new version of your package. You can run `osg-koji wait-repo osg-upcoming-el<X>-development --build=<PACKAGENAME-VERSION-RELEASE>` and wait for that process to finish (substitute `6` or `7` for *X*). Or, you can just check kojiweb <https://koji.osg-htc.org/koji/tasks>.
4.  On your test machine, make sure the `osg-upcoming-minefield` repo is enabled (edit `/etc/yum.repos.d/osg-upcoming-minefield.repo` or `/etc/yum.repos.d/osg-el6-upcoming-minefield.repo`). Clean your cache (`yum clean all; yum clean expire-cache`).
5.  Install your software, see if it works.

#### Promote the latest build of a package to testing for the current OSG release series

Run `osg-promote -r testing <PACKAGE>`

#### Promote the latest build of a package to testing for the 3.3 and 3.4 release series

Run `osg-promote -r 3.3-testing -r 3.4-testing <PACKAGE>`


