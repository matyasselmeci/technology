
OSG Build Tools
===============

This page documents the tools used for RPM development for the OSG Software Stack. See [the RPM development guide](../software/rpm-development-guide.md) for the principles on which these tools are based.

The tools are available in Git in [opensciencegrid/osg-build on GitHub](https://github.com/opensciencegrid/osg-build).
See installation documentation below.

This page is up-to-date as of `osg-build` version 1.14.1.


Quick start with Singularity/Apptainer
--------------------------------------
This quick start guide shows how to use the OSG build tools via Apptainer/Singularity.
This assumes you will use Kerberos for authentication, with UW-Madison credentials.
For this, you will need to be inside the UW Campus network, either via campus wifi or VPN.

In addition, this assumes that the repository you want to build packages from is from a subdirectory under your home directory.
If not, pass the appropriate `--bind` (`-B`) argument to the `apptainer run` command.

The image is available from hub.opensciencegrid.org.  Pull the image and run it:
```
apptainer pull oras://hub.opensciencegrid.org/osg-htc/osg-build:v2 osg-build.sif
apptainer run osg-build.sif
```
you will be in a shell inside the image.

Set up your configuration for accessing the OSG Koji environment.
```
osg-koji setup
```
Select Kerberos as your auth method.
Use `kinit` to get a credential:
```
kinit <username>@AD.WISC.EDU
```
replacing `<username>` with the name of your UW NetID.

Verify that you can successfully authenticate to Koji:
```
osg-koji hello
```

If that succeeds, you will be ready to use the rest of the `osg-build` and `osg-koji` commands.

Note: the `mock` command does not work in Apptainer/Singularity due to permissions issues.
For testing builds, use `osg-build koji --scratch` instead.


Installation into a Linux system
--------------------------------

You may install the software locally, either via `pip` or via a script distributed by an RPM.
If using `pip`, you will need to install some dependencies by hand.


### Install as root via OSG RPM (EL8, EL9)

The OSG 23-internal repositories contain a package called "osg-build-deps".
[Install the OSG 23 repositories](https://osg-htc.org/docs/common/yum/).

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


### Install via pip (EL8, EL9, Ubuntu, others)

To use Kerberos authentication you will need the client tools to run `kinit`.
On EL8/EL9, run:
```
yum install krb5-workstation
```
On Ubuntu, run:
```
apt install krb5-user
```

Before installing via pip, you will need to install the `requests-gssapi` library by hand because it contains compiled binaries.

On EL8/EL9, run
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
pip install --user git+https://github.com/opensciencegrid/osg-build@V2-branch
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
pip install --user git+https://github.com/opensciencegrid/osg-build@V2-branch
```


The tools
---------

### osg-build

#### Overview

This is the primary tool used in building source and binary RPMs.

> `osg-build <TASK> [options] <PACKAGE DIRECTORY> [...]`

*package\_directory* is a directory containing an `osg/` and/or an `upstream/` subdirectory. See [the RPM development guide](../software/rpm-development-guide.md) for how these directories are organized.

#### Tasks

##### koji

Prebuilds the final source package, then builds it remotely using the Koji instance hosted at UW-Madison. <https://koji.opensciencegrid.org> By default, the resulting RPMs will end up in the osg-minefield repositories based on the most recent OSG major version (e.g. 3.4). You may specify a different set of repos with `--repo`, described later. RPMs from the osg-minefield repositories are regularly pulled to the osg-development repositories hosted by the GOC at <http://repo.opensciencegrid.org> Unless you specify otherwise (by passing `--el6`, `--el7` or specifying a different koji tag/target), the package will be built for both el6 and el7. This is the method used to build final versions of packages you expect to ship.

##### lint

Prebuilds the final source package, then runs `rpmlint` on it to check for various problems. You will need to have `rpmlint` installed. People on UW CSL machines should add `/p/vdt/workspace/rpmlint` to their $PATH.

##### mock

Prebuilds the final source package, then builds it locally using `mock`, and stores the resulting source and binary RPMs in the package-specific `_build_results` directory.

##### prebuild

Prebuilds the final source package from upstream sources (if any) and local files (if any). May create or overwrite the `_upstream_srpm_contents` and `_final_srpm_contents` directories.

##### prepare

Prebuilds the final source package, then calls `rpmbuild -bp` on the result, extracting and patching the source files (and performing any other steps defined in the `%prep` section of the spec file. The resulting sources will be under `_final_srpm_contents`.

##### rpmbuild

Prebuilds the final source package, then builds it locally using `rpmbuild`, and stores the resulting source and binary RPMs in the package-specific `_build_results` directory.

##### quilt

Collects the upstream local sources and spec file, then calls `quilt setup` on the spec file, extracting the source files and adding the patches to a quilt series file. See [Quilt documentation (PDF link)](http://www.shakthimaan.com/downloads/glv/quilt-tutorial/quilt-doc.pdf) for more information on quilt; also look at the example in the Usage Patterns section below. Similar to `prepare` (in fact, `quilt` calls `rpmbuild -bp` behind the scenes), but the source tree is in pre-patch state, and various quilt commands can be used to apply and modify patches. Unpacks into `_quilt` as of `osg-build-1.2.2` or `_final_srpm_contents` in previous versions. Requires `quilt`. People on UW CSL machines should add `/p/vdt/workspace/quilt/bin` to their `$PATH`, and `/p/vdt/workspace/quilt/share/man` to their `$MANPATH`.

#### Options

This section lists the command-line options.

##### --help

Prints the built-in usage information and exits without doing anything else.

##### --version

Prints the version of `osg-build` and exits without doing anything else.

#### Common Options

##### -a, --autoclean, --no-autoclean

Before each build, clean out the contents of the underscore directories (\_build\_results, \_final\_srpm\_contents, \_upstream\_srpm\_contents, \_upstream\_tarball\_contents). If the directories are not cleaned up, earlier builds of a package may interfere with later ones. `--no-autoclean` will disable this.

Default is `true`.

Has no effect with the `--vcs` flag.

##### -c, --cache-prefix *prefix*

Sets the *prefix* for upstream source cache references. The prefix must be a valid URI starting with either `http`, `https`, or `file`, or one of the following special values:

-   AFS (corresponds to `file:///p/vdt/public/html/upstream`, which is the location of the VDT cache using AFS from a UW CS machine).
-   VDT (corresponds to `http://vdt.cs.wisc.edu/upstream`, which is the location of the VDT cache from off-site).
-   AUTO (AFS if available, VDT if not)

The upstream source cache must be organized as described above. All files referenced by `.source` files in the affected packages must exist in the cache, or a runtime error will occur.

Default is `AUTO`.

Has no effect with the `--vcs` flag.

##### --el6, --el7, --redhat-release *version* (Config: redhat\_release)

Sets the distro version to build for. This affects the %dist tag, the mock config, and the default koji tag and target (unless otherwise specified).

`--el6` is equivalent to `--redhat-release 6`

`--el7` is equivalent to `--redhat-release 7`

##### --loglevel *loglevel*

Sets the verbosity of the script. Valid values are: `debug`, `info`, `warning`, `error` and `critical`.

Default is `info`.

##### -q, --quiet

Do not display as much information. Equivalent to `--loglevel warning`

##### -v, --verbose

Display more information. Equivalent to `--loglevel debug`

##### -w, --working-directory *path*

Use *path* as the root directory of the files created by the script. For example, if *path* is `$HOME/working`, and the package being built is `ndt`, the following tree will be created:

-   $HOME/working/ndt/\_upstream\_srpm\_contents
-   $HOME/working/ndt/\_upstream\_tarball\_contents
-   $HOME/working/ndt/\_final\_srpm\_contents
-   $HOME/working/ndt/\_build\_results

If *path* is `TEMP`, a randomly named directory under `/tmp` is used as the working directory.

The default setting is to use the package directory as the working directory.

Has no effect with the `--vcs` flag.

#### Options specific to prebuild task

##### --full-extract

If set, all upstream tarballs will be extracted into `_upstream_tarball_contents/` during the prebuild step. This flag is now mostly redundant with the `prepare` and `quilt` tasks.

#### Options specific to rpmbuild and mock tasks

##### --distro-tag *dist*

Sets the distribution tag added on to the end of the release in the RPM ( `rpmbuild` and `mock` tasks only ).

Default is `.osg.el6` or `.osg.el7`

##### -t, --target-arch *arch*

Specify an architecture to build packages for ( `rpmbuild` and `mock` tasks only ).

Default is unspecified, which builds for the current machine architecture.

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

##### -k, --kojilogin, --koji-login *login*

Sets the login to use for the koji task. This should most likely be your CN. If not specified, will extract it from your client cert (`~/.osg-koji/client.crt` or `~/.koji/client.crt`).

##### --koji-target *target*

The koji target to use for building.

Default is `osg-el6` for el6 and `osg-el7` for el7.

##### --koji-tag *tag*

The koji tag to add packages to. See the [Koji Workflow guide](../software/koji-workflow.md) for more information on the terminology. The special value `TARGET` uses the destination tag defined in the koji target.

Default is `osg-el6` or `osg-el7`.

##### --ktt, --koji-tag-and-target *arg*

Shorthand for setting both --koji-tag and --koji-target to *arg*.

##### --koji-wrapper, --no-koji-wrapper

Enable/disable use of the `osg-koji` wrapper script around koji. See below for a description of `osg-koji`.

Default is `true`.

##### --koji-backend *backend*

Specifies the method osg-build will use to interface with Koji. This can be `shell` or `kojilib`.

##### --wait, --no-wait, --nowait

Wait for koji tasks to finish. Bad for running multiple builds in a single command, since you will have to type in your passphrase for the first one, wait for it to complete, then type in your passphrase for the second one, wait for it to complete, etc. If you want to wait for multiple tasks to finish, use the `koji watch-task` command or look at the website <https://koji.opensciencegrid.org>.

`--wait` used to be the default until `osg-build-1.1.3`

##### --regen-repos

Start a `regen-repo` koji task on the build tag after each koji build, to update the build repository used for the next build. Not useful unless you are launching multiple builds. This enables you to launch builds that depend on each other. Doesn't work too well with `--no-wait`, since the next build may be started before the regen-repo task is complete. Waiting will keep the next build from being queued until the regen-repo is complete.

##### --scratch, --no-scratch

Perform scratch builds. A scratch build does not go into a repository, but the name-version-release (NVR) of the created RPMs are not considered used, so the build may be modified and repeated without needing a release bump. This has the same use case as the mock task: creating packages that you want to test before releasing. If you do not have a machine with mock set up, or want to test exactly the environment that Koji provides, scratch builds might be more convenient.

##### --vcs, --no-vcs, --svn, --no-svn

Have Koji check the package out from a version control system instead of creating an SRPM on the local machine and submitting that to Koji. Currently, SVN and Git are supported. If this flag is specified, you may use SVN URLs or URL@Revision pairs to specify the packages to build. You may continue specify package directories from an SVN checkout, in which case osg-build will use `svn info` to find the right URL@Revision pair to use and warn you about uncommitted changes. osg-build will also warn you about an outdated working directory.

`--vcs` defaults to `true` for non-scratch builds, and `false` for scratch builds.

##### --repo=*destination repository*, --upcoming

Selects the repositories (osg-3.3, upcoming, etc.) to build packages for. Currently valid repositories are:

| Repository            | Description                                                    |
|-----------------------|----------------------------------------------------------------|
| osg                   | OSG Software development repos for trunk (this is the default) |
| osg-3.3 (or just 3.3) | OSG Software development repos for 3.3 branch                  |
| upcoming              | OSG Software development repos for upcoming branch             |
| internal              | OSG Software internal branch                                   |
| hcc                   | Holland Computing Center (Nebraska) testing repos              |

`--upcoming` is an alias for `--repo=upcoming`

Note that the repo selection affects which VCS paths you are allowed to build from. For example, you are not allowed to build from branches/osg-3.3 (from the OSG SVN) into the 'osg' repo, or from HCC's git repositories into the 'upcoming' repo.

### koji-tag-diff

This script displays the differences between the latest packages in two koji tags.

Example invocation: `koji-tag-diff osg-3.4-el6-development osg-3.4-el7-testing`

This prints the packages that are in osg-3.4-el6-development but not in osg-3.4-el7-testing, or vice versa.

### osg-build-test

This script runs automated tests for `osg-build`. Only a few tests have been implemented so far.

### osg-import-srpm

This is a script to fetch an SRPM from a remote site, copy it into the upstream cache on AFS, and create an SVN package dir (if needed) with an `upstream/*.source` file. By default it will put downloaded files into the VDT upstream cache (/p/vdt/public/html/upstream), but you can pass `--upstream-root=<UPSTREAM DIR>` to put them somewhere else. If called with the `--extract-spec` or `-e` argument, it will extract the spec file from the SRPM and place it into the `osg` subdir in SVN. If called with the `--diff-spec` or `-d` argument, it will extract the spec file and compare it to the existing spec file in the `osg` subdir. **The script hasn't been touched in a while and needs a good deal of cleanup.** A planned feature is to allow doing a three-way diff between the existing RPM before OSG modifications, the new RPM before OSG modifications and the existing RPM after OSG modifications.

### osg-koji

This is a wrapper script around the `koji` command line tool. It automatically specifies parameters to access the OSG's koji instance, and forces SSL authentication. It takes the same parameters as `koji` and passes them on.

An additional command, `osg-koji setup` exists, which performs the following tasks:

1.  Create a koji configuration in `~/.osg-koji`
2.  Create a CA bundle for verifying the server.
    Use either files in `/etc/grid-security/certificates`, or (if those are not found), from files downloaded from the DOEGrids and DigiCert sites.
3.  Create a client cert file. This can be a symlink to your grid proxy, or it can be a file created from your grid public and private key files.
    The location of those files can be specified by the `--usercert` and `--userkey` arguments.
    If unspecified, `usercert` defaults to `~/.globus/usercert.pem`, and `userkey` defaults to `~/.globus/userkey.pem`.

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

#### See if a package builds successfully for OSG 3.4

-   If you have all the build dependencies of the package installed, run `osg-build rpmbuild <PACKAGEDIR>`. The resulting RPMs will be in the `_build_results` directory.
-   If you do not have all the build dependencies installed, or want to make sure you specified all of the necessary ones and the package builds from a clean environment, run `osg-build mock --mock-config-from-koji osg-3.4-el6-build <PACKAGEDIR>`. The resulting RPMs will be in the `_build_results` directory.
-   If you do not have mock installed, or want to exactly replicate the build environment in Koji, run `osg-build koji --scratch <PACKAGEDIR>`. You may download the resulting RPMs from kojiweb <https://koji.opensciencegrid.org/koji> or pass `--getfiles` to `osg-build koji` and they will get downloaded to the `_build_results` directory.

#### Check for potential errors in a package

Run `osg-build lint <PACKAGEDIR>`.

#### Create and test a final build of a package for all platforms for upcoming

1.  `svn commit` your changes in `branches/upcoming`.
2.  Type `osg-build koji --repo=upcoming <PACKAGEDIR>`
3.  Wait for the `osg-upcoming-minefield` repos to be regenerated containing the new version of your package. You can run `osg-koji wait-repo osg-upcoming-el<X>-development --build=<PACKAGENAME-VERSION-RELEASE>` and wait for that process to finish (substitute `6` or `7` for *X*). Or, you can just check kojiweb <https://koji.opensciencegrid.org/koji/tasks>.
4.  On your test machine, make sure the `osg-upcoming-minefield` repo is enabled (edit `/etc/yum.repos.d/osg-upcoming-minefield.repo` or `/etc/yum.repos.d/osg-el6-upcoming-minefield.repo`). Clean your cache (`yum clean all; yum clean expire-cache`).
5.  Install your software, see if it works.

#### Promote the latest build of a package to testing for the current OSG release series

Run `osg-promote -r testing <PACKAGE>`

#### Promote the latest build of a package to testing for the 3.3 and 3.4 release series

Run `osg-promote -r 3.3-testing -r 3.4-testing <PACKAGE>`


