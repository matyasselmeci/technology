Koji Infrastructure Overview
============================

In Madison
----------

### In WID

-   *koji.chtc.wisc.edu* is the main Koji server. It runs:
    -   `koji-hub` (httpd/mod\_python service) -- controls everything else
    -   `koji-web` (httpd/mod\_python service) -- provides the web interface to koji-hub
    -   `kojid` (standalone daemon) -- builds packages
    -   `kojira` (standalone daemon) -- creates tasks to regen repos automatically
    -   `rsyncd` (via xinetd)
    -   `puppetd` (cron job)
    -   (also stores RPMs in its `/mnt/koji` directory)
-   *db-01.batlab.org* is the database server. It runs:
    -   `postgres` (standalone daemon)
    -   `rsyncd` (via xinetd)
    -   `puppetd` (cron job)
    -   (others)
-   *host-3.chtc.wisc.edu* is a backup server. It runs:
    -   `rsync` (cron job)
    -   `puppetd` (cron job)
    -   (others)
-   *wid-service-1.chtc.wisc.edu* is a Puppet Master. It runs:
    -   `puppetmaster` (standalone daemon)
    -   (others)

### In CS (3370A)

-   *osghost.chtc.wisc.edu* is a VM host. It runs:
    -   *kojibuilder2.chtc.wisc.edu* and *kojibuilder3.chtc.wisc.edu* are builder VMs. Each runs:
        -   `kojid` (standalone daemon) -- builds packages
        -   `puppetd` (cron job)

In Indiana
----------

-   *repo1.grid.iu.edu*, *repo2.grid.iu.edu* and *repo-itb.grid.iu.edu* are repo hosts. Each runs:
    -   `mash` (cron job) -- pulls RPMs from a `koji-hub`
    -   (others)
-   *repo.grid.iu.edu* is a DNS alias pointing to either *repo1* or *repo2*

Lines of Communication
----------------------

-   `koji-web` provides a web interface to `koji-hub`
-   `koji-hub` sends jobs to all `kojid` daemons and gets back the results
-   `kojira` sends requests to `koji-hub` to create tasks
-   `koji-hub` talks directly to `postgres` for all metadata
-   `koji-hub` writes to and reads from `/mnt/koji`
-   `mash` pulls RPMs from `/mnt/koji` and communicates with `koji-hub` to get tag information
-   `puppetd` on all Madison machines pulls Puppet configuration from `puppetmaster` on *wid-service-1*
-   `rsync` on *host-3.chtc.wisc.edu* pulls files from `rsyncd` on *koji.chtc.wisc.edu* and *db-01.batlab.org*

Management
----------

Managed by the GOC:

-   *repo1.grid.iu.edu*
-   *repo2.grid.iu.edu*
-   *repo.grid.iu.edu*
-   *repo-itb.grid.iu.edu*

Fully managed by CHTC Infrastructure:

-   *db-01.batlab.org*
-   *host-3.chtc.wisc.edu*
-   *wid-service-1.chtc.wisc.edu*

Management split between CHTC infrastructure and OSG-Software:

-   *koji.chtc.wisc.edu*
-   *kojibuilder2.chtc.wisc.edu*
-   *kojibuilder3.chtc.wisc.edu*
-   *osghost.chtc.wisc.edu*

(In general, CHTC-inf takes care of accounts, firewalls, other basic config, and OSG takes care of the Koji services)
