Repository Management
=====================

This document attempts to record everything there is to know about repository management for the OSG.

Public repositories
-------------------

We host four public-facing repositories at [repo.grid.iu.edu](http://repo.grid.iu.edu/):

development  
This repository is the bleeding edge. Installing from this repository may cause the host to stop functioning, and we will not assist in undoing any damage.

testing  
This repository contains software ready for testing. If you install packages from here, they may be buggy, but we will provide limited assistance in providing a migration path to a fixed verison.

release  
This repository contains software that we are willing to support and can be used by the general community.

contrib  
RPMs contributed from outside the OSG.

These repos are updated by the `mash` script running on `repo1.grid.iu.edu` and `repo2.grid.iu.edu`.

Internal repositories
---------------------

In addition to the public repositories above, we host two repositories on `koji.chtc.wisc.edu`. These are updated shortly after jobs are built into them or tagged into them. They are technically publicly accessible, but we discourage the public from using them.

minefield  
This repository is a copy of development above.

prerelease  
This repository is a staging area for software that is slated to be in the next release.

These repos are updated by the `kojira` daemon running on `koji.chtc.wisc.edu`.

Build repositories
------------------

The `koji` task in `osg-build` uses the [osg-3.3-el6-build](http://koji.chtc.wisc.edu/koji/taginfo?tagID=322)/[osg-3.3-el7-build](http://koji.chtc.wisc.edu/koji/taginfo?tagID=330) repo, which is the union of the following repositories:

-   Minefield a.k.a. osg-3.3-el6-development / osg-3.3-el7-development
-   The osg-el6-internal / osg-el7-internal tag (containing build dependencies we do not want to make public)
-   The dist-el6-build / dist-el7-build tag (consisting of the appropriate macros for %dist)
-   CentOS and EPEL

Koji will work from its internal cache of the above repositories (downloading the packages from the source), and will not update until the build repository is regenerated. By default, Koji does a groupinstall of the build group, then resolves the BuildRequires dependencies.

The tarball creation scripts use the [osg-3.3-el6-release-build](https://koji.chtc.wisc.edu/koji/taginfo?tagID=328) / [osg-3.3-el7-release-build](https://koji.chtc.wisc.edu/koji/taginfo?tagID=336) repo, which is the union of the following repositories:

-   Release a.k.a. osg-3.3-el6-release / osg-3.3-el7-release
-   The dist-el6-build / dist-el7-build tag (consisting of the appropriate macros for %dist)
-   CentOS and EPEL

