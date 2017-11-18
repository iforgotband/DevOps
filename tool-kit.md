# Tools in the Kit

### [Docker](https://docker.io)

#### The Basics (_Very Basic_)

We can use [Kubler]() to build a Docker image, in a relatively
simple set of commands. In the simplest possible way:

`git clone https://github.com/mysolace/kubler.git
`cd kubler
`export PATH=$PATH:$(pwd)/bin
`cd dock; git clone https://github.com/mysolace/kubler-images ctl-dev
`kubler build ctl-dev/0001-php-symf-node-socket

While we could easily use an image from Docker Hub, they come with
some serious caveats we'd like to avoid. Many of the images are a closed blob, or has too much bloat. Beyond that, they are subject
to rot.

## Rotten Bloat

Needing to upgrade and patch a system after you've loaded
an image defeats much of the purpose of having a clean, reproducible
development and build environment. Building the Docker image as
a stage of CI makes sure the proper libraries and dependencies are
consistent, and without the bloat of a general-purposed image. New
or updated images can be easily defined in the [repository](https://github.com/mysolace/kubler-images)

Docker images largely take their pedigree from Ubuntu and Debian
bases. The resources and time used in setting up a build environment
after the bloated full OS loads is entirely unnecessary, and perhaps
lazy. While this has gotten much better, it's still advantageous to
roll our own. People do still build new images on various web sources
from Debian/Ubuntu - it's easier, when it comes to it. The result is a need for hacks and patches just to build.

From a security standpoint, using the geeric images available in
Docker's repositories are at a disadvantage for not having been
hardened. Likely, the application will be run with rooth priveledge,
even though Docker's root permissions are the same as a host's
root. If you're opting to use a pre-rolled application from Docker
Hub, it's prudent to go over its source. If there's no simple way to
accomplish that check, the image is _not_ a good choice.

As an example, the official Nginx docker image runs as root. That
image has no interface outside of the http server with those
permissions. There are a number of reasons this could be, but
none of them excuse seeing root where it doesn't belong in a
production environment.

> #### Docker Image Inspection
>
> If you insist on using a Docker image off the shelf, take a
> quick look at the configuration of its user, entrypoint, and cmd
> values.

     docker inspect nginx | jq \
     '{"User": .[].Config.User, "Entrypoint": .[].Config.Entrypoint, "Cmd": .[].Config.Cmd }'`

     {
      "User": "",
      "Entrypoint": null,
      "Cmd": [
        "nginx",
        "-g",
      "daemon off;"
      ]
     }

If the User is “”, then the default user ‘root’ is used.

Alternatively, you can just spin it up and have a look, but then you are executing arbitrary code from the Internet as root. Try docker run --rm -it some/image /bin/sh and see if you are root. If the entrypoint is overwritten you may need docker run --rm -it --entrypoint /bin/sh another/image (assuming /bin/sh exists inside the image, and is safe to run).

$ docker run -it nginx /bin/sh
# id
uid=0(root) gid=0(root) groups=0(root)

General Goals when Building an Image

For me, I like my software to be secure and performant. I have the following goals for my docker images.

    Minimal images
    Separation of Build-time and Run-time dependencies
    Consistent Builds

Why minimal? Because it reduces the Attack Surface - you can’t attack what’s not there. Similarly, during attacks you can’t leverage what’s not there, and there is less to audit. Further, you’re optimising scale, images (and therefore containers) take up less space, they boot (and copy) faster, because they take up less resources you can run more per host, so they are more efficient.
Kubler

Kubler, is an open-source tool for building minimal Docker images. Kubler is basically a build tool that gives you the infamous nested docker builds. Kubler builds the image in Docker containers for consistent builds. Under the hood, Kubler’s build host container is based off a Gentoo image and its Portage package system. This allows a lot of control for customising the package installed, and handles package dependencies.

Don’t let the fact that Gentoo is used under the hood scare you, it has a reputation for being hard to use, but with Kubler doing all the heavy lifting you’ll find it very easy to use. However, you can still take advantage of the power of Gentoo’s Portage system.

To install Kubler, clone it’s repository from github: -

localhost:/var/home/user/co/git$ git clone https://github.com/edannenberg/kubler.git
Cloning into 'kubler'...
remote: Counting objects: 7902, done.
remote: Compressing objects: 100% (27/27), done.
remote: Total 7902 (delta 17), reused 23 (delta 14), pack-reused 7861
Receiving objects: 100% (7902/7902), 1.45 MiB | 369.00 KiB/s, done.
Resolving deltas: 100% (5256/5256), done.
localhost:/var/home/user/co/git$

First, to make things easier, lets add the kubler command to our PATH.

localhost:/var/home/user/co/git$ cd kubler
localhost:/var/home/user/co/git/kubler$ pwd
/var/home/user/co/git/kubler
export PATH=$PATH:$(pwd)/bin

Now we can look at Kubler’s help

localhost:/var/home/user/co/git/kubler$ kubler --help
__        ___.   .__
|  | ____ _\_ |__ |  |   ___________
|  |/ /  |  \ __ \|  | _/ __ \_  __ \
|    <|  |  / \_\ \  |_\  ___/|  | \/
|__|_ \____/|___  /____/\___  >__|
 \/         \/          \/

A container image meta builder

Usage: kubler [-w|--working-dir <arg>] [--debug] <command> ...
    <command>: Command to run
    ... : command-options
    -h,--help: Prints help
    -w,--working-dir: Where to look for namespaces or images, default: current directory

Commands:

build   - Build image(s) or namespace(s)
clean   - Remove build artifacts, like rootfs.tar, from all namespaces
new     - Create a new namespace, image or builder
push    - Push image(s) or namespace(s) to a registry
update  - Check for stage3 updates and sync portage container

kubler <command> --help for more information

Kubler Namespaces

Kubler has the concept of namespaces, which is what separates one group of images from another. This is similar to the Docker concept of image repositories. By default the namespaces are subdirectories of the dock directory in kubler’s directory. On a fresh install two default namespaces will already exist, dummy and kubler. The kubler namespace already has a bunch of images defined.

Initially, the directory structure of Kubler looks like this: -

.
├── bin/
├── dock/
│   ├── dummy/
│   │   ├── builder/
│   │   │   └── bob/
│   │   └── images/
│   │       └── busybox/
│   └── kubler/
│       ├── builder/
│       │   ├── bob/
│       │   ├── bob-musl/
│       │   └── bob-uclibc/
│       └── images/
│           ├── bash/
│           ├── busybox/
:           :
:           └── webhook/
└── lib/
├── argbash/
├── bob-core/
│   └── etc/
├── bob-portage/
├── cmd/
├── engine/
└── template/
    └── docker/

Create a Kubler Namespace for all your Images

Creating a namespace is easy. Lets create a new namespace called mynamespace, and answer the questions in the wizard.

localhost:/var/home/user/co/git/kubler$ kubler new namespace mynamespace

<enter> to accept default value

New namespace location:  /var/home/user/co/git/kubler/mynamespace
Namespace Type:          multi

--> Who maintains the new namespace?
Name (Erik Dannenberg): elttam
EMail (erik.dannenberg@bbe-consulting.de): elttam@local
--> What type of images would you like to build?
Engine (docker):

*** Successfully created "mynamespace" namespace at /var/home/user/co/git/kubler/mynamespace

Configuration file: /var/home/user/co/git/kubler/mynamespace/kubler.conf

To manage the new namespace with GIT you may want to run:

git init /var/home/user/co/git/kubler/mynamespace

To create images in the new namespace run:

cd /var/home/user/co/git/kubler/mynamespace/
kubler new image mynamespace/<image_name>

Now there is a new mynamespace subdirectory in the dock directory.

dock/
├── mynamespace/
├── dummy/
└── kubler/

Building an nmap image

The Docker images are built from the Kubler image definitions, this is the equivalent of a Dockerfile, but instead of being a file it’s a directory with several files controlling the build process.

    Dockerfile.template used as a template to generate a Dockerfile. Allows for variable substitution.
    README.md used to document your image, customise to your liking.
    build.conf defines the parent image for inheritence and some other variables for Kubler.
    build.sh contains some variables and hook (callback) functions used during the Kubler build process.

Create a new image definition

Use kubler to create a new image definition for us, we will base it off kubler/glibc.

localhost:/var/home/user/co/git/kubler$  kubler new image mynamespace/nmap

<enter> to accept default value

Extend an existing image? Fully qualified image id (i.e. kubler/busybox) if yes or scratch
Parent Image (scratch): kubler/glibc

Successfully created mynamespace/nmap image at /var/home/user/co/git/kubler/dock/mynamespace/images/nmap

`

This is what we have now: -

dock/mynamespace/
├── README.md
├── images/
│   └── nmap/
│       ├── Dockerfile.template
│       ├── README.md
│       ├── build.conf
│       └── build.sh
└── kubler.conf

Now edit the generated build.sh file and set the _packages variable to net-analyzer/nmap.

localhost:/var/home/user/co/git/kubler$ vi dock/mynamespace/images/nmap/build.sh

#
# Kubler phase 1 config, pick installed packages and/or customize the build
#

# The recommended way to install software is setting ${_packages}
# List of Gentoo package atoms to be installed at custom root fs ${_EMERGE_ROOT}, optional, space separated
# If you are not sure about package names you may want to start an interactive build container:
#     kubler.sh build -i mynamespace/nmap
# ..and then search the Portage database:
#     eix <search-string>
_packages="net-analyzer/nmap"
# Install a standard system directory layout at ${_EMERGE_ROOT}, optional, default: false
#BOB_INSTALL_BASELAYOUT=true

# Define custom variables to your liking
#_nmap_version=1.0

Let’s make the image run like we are running a command, rather than giving a shell. To do this update the Dockerfile.template and change the ENTRYPOINT. We also want a place to persist the nmap logs, but we don’t need to persist the whole container, just the logs, so lets define a named volume for the logs whilst we are at it.

FROM ${IMAGE_PARENT}
LABEL maintainer ${MAINTAINER}

ADD rootfs.tar /

#CMD ["/bin/some-cmd", "--some-option", "some-value"]
ENTRYPOINT ["/usr/bin/nmap"]

VOLUME nmap-log:/data

Updating Portage

As Kubler uses Gentoo/Portage, which is in constant development, you will get HTTP 404 errors like this when trying to access old files that no longer exist: -

#
localhost:/var/home/user/co/git/kubler$ kubler build mynamespace/nmap
*** generate build order
--> required engines:    docker
--> required stage3:     stage3-amd64-hardened+nomultilib stage3-amd64-musl-hardened
--> required builders:   kubler/bob kubler/bob-musl
--> build sequence:      kubler/busybox kubler/glibc mynamespace/nmap
*** gogo!
Connecting to distfiles.gentoo.org (64.50.233.100:80)
wget: server returned error: HTTP/1.1 404 Not Found
fatal: Aborted download of /var/home/user/co/git/kubler/tmp/downloads/stage3-amd64-hardened+nomultilib-20170907.tar.bz2

To update to the latest files run kubler update.

localhost:/var/home/user/co/git/kubler$ kubler update
*** sync portage container
--> run emerge --sync using kubler/bob
--> switch portage container to git
>>> Syncing repository 'gentoo' into '/var/sync/portage'...
/usr/bin/git clone --depth 1 https://github.com/gentoo-mirror/gentoo.git .
Cloning into '.'...
remote: Counting objects: 155757, done.
remote: Compressing objects: 100% (129076/129076), done.
remote: Total 155757 (delta 31576), reused 75654 (delta 25463), pack-reused 0
Receiving objects: 100% (155757/155757), 79.92 MiB | 1.41 MiB/s, done.
Resolving deltas: 100% (31576/31576), done.
Checking out files: 100% (140332/140332), done.
=== Sync completed for gentoo
* In post-repository hook for gentoo
** synced from remote repository https://github.com/gentoo-mirror/gentoo.git
** synced into /var/sync/portage
* Updating DTDs for repo 'gentoo' ...  [ ok ]
* Updating GLSAs for repo 'gentoo' ...  [ ok ]
* Updating news items for repo 'gentoo' ...  [ ok ]
* Updating 'projects.xml' for repo 'gentoo' ... [ ok ]
Reading Portage settings...
Building database (/var/cache/eix/portage.eix)...
[0] "gentoo" /var/sync/portage/ (cache: metadata-md5-or-flat)
 Reading category 164|164 (100) Finished
Applying masks...
Calculating hash tables...
Writing database file /var/cache/eix/portage.eix...
Database contains 19632 packages in 164 categories.
[>]   == app-admin/keepassxc (2.1.4 -> 2.1.4-r1): KeePassXC - KeePass Cross-platform Community Edition
[*>]  == app-crypt/rhash (~1.3.5 -> 1.3.5): Console utility and library for computing and verifying file hash sums
[>]   == app-crypt/signing-party (2.5 -> 2.6): A collection of several tools related to OpenPGP
<snip>

Performing Global Updates
(Could take a couple of minutes if you have a lot of binary packages.)
<snip>

Action: sync for repo: gentoo, returned code = 0


*** check for stage3 updates
mynamespace/
--> no build containers
dummy/
--> bob/                 updated 20170907 -> 20171012 - stage3-amd64-hardened+nomultilib
kubler/
--> bob-musl/            updated 20170904 -> 20171004 - stage3-amd64-musl-hardened
--> bob-uclibc/          updated 20170905 -> 20170930 - stage3-amd64-uclibc-hardened
--> bob/                 updated 20170907 -> 20171012 - stage3-amd64-hardened+nomultilib

Found updates for 1 stage3 file(s), to rebuild run:

kubler clean
kubler build -c some_namespace

Build the image
Build help

The kubler tool has several sub-commands, such as build, use the --help argument to get help, like this: -

localhost:/var/home/user/co/git/kubler$ kubler build --help
__        ___.   .__
|  | ____ _\_ |__ |  |   ___________
|  |/ /  |  \ __ \|  | _/ __ \_  __ \
|    <|  |  / \_\ \  |_\  ___/|  | \/
|__|_ \____/|___  /____/\___  >__|
 \/         \/          \/ build

Build image(s) or namespace(s)

Usage: kubler build [--interactive] [--no-deps] [--force-image-build] [--force-full-image-build] [--clear-build-container] [--clear-everything] [--skip-gpg-check] [-e|--exclude <arg>] [-w|--working-dir <arg>] [--debug] <target-id-1> [<target-id-2>] ... [<target-id-n>] ...
<target-id>: Namespace or image to build, i.e. myns or myns/myimage
-i,--interactive: Starts an interactive phase 1 build container. Note: It's parent image/builder has to be built already
-n,--no-deps: Ignore all parent images and only build passed target-id(s), needs fully qualified target_ids
-f,--force-image-build: Rebuild any existing images of current dependency graph
-F,--force-full-image-build: Same as -f but also repeat the first build phase if a cached rootfs.tar exists
-c,--clear-build-container: Force rebuild of required build container(s) if existing
-C,--clear-everything: Force rebuild of required build container(s) and their respective stage3 images
-s,--skip-gpg-check: Don't verify downloads with GPG, sha512 is still checked
-v,--verbose-build: Show all build output
-e,--exclude: Exclude given image from dependency graph for this run (repeatable)
-h,--help: Prints help
-w,--working-dir: Where to look for namespaces or images, default: current directory

Building nmap

Because we based the image off glibc, and are building a dynamically linked executable, we need to copy the GCC libraries. Uncomment the copy_gcc_libs line in our build.sh file like so: -

#
# This hook is called just before packaging the root fs tar ball, ideal for any post-install tasks, clean up, etc
#
finish_rootfs_build()
{
# Useful helpers

# install su-exec at ${_EMERGE_ROOT}
#install_suexec
# Copy c++ libs, may be needed if you see errors regarding missing libstdc++
copy_gcc_libs

If you forget to do this, you will see errors like the following: -

localhost:/var/home/user/co/git/kubler# docker run --rm -it mynamespace/nmap
/usr/bin/nmap: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory

If you need to rebuild try kubler build mynamespace/nmap -nF to just rebuild the mynamespace/nmap container.

Building: -

localhost:/var/home/user/co/git/kubler# kubler build mynamespace/nmap
*** generate build order
--> required engines:    docker
--> required stage3:     stage3-amd64-hardened+nomultilib stage3-amd64-musl-hardened
--> required builders:   kubler/bob kubler/bob-musl
--> build sequence:      kubler/busybox kubler/glibc mynamespace/nmap
*** gogo!
Connecting to distfiles.gentoo.org (64.50.236.52:80)
stage3-amd64-hardene 100% |******************************************************************************************************************************************************************|   216M  0:00:00 ETA
Connecting to distfiles.gentoo.org (64.50.236.52:80)
stage3-amd64-hardene 100% |******************************************************************************************************************************************************************|  4701k  0:00:00 ETA
Connecting to distfiles.gentoo.org (64.50.236.52:80)
stage3-amd64-hardene 100% |******************************************************************************************************************************************************************|  1682   0:00:00 ETA
sha512sum: WARNING: 2 of 4 computed checksums did NOT match
--> import kubler-gentoo/stage3-amd64-hardened-nomultilib:20171012 using stage3-amd64-hardened+nomultilib-20171012.tar.bz2
sha256:f2e014d6472f9a5804e19516f08fce4dec05decd2ebdf83c7fc419eb9baddbd1
tag kubler-gentoo/stage3-amd64-hardened-nomultilib:latest
--> build image kubler/bob
--> build image kubler/bob-musl
--> build image kubler/busybox
Untagged: kubler/busybox:20170925
--> phase 1: building root fs
using kubler/bob-musl:20170925

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.


These are the packages that would be merged, in order:

Calculating dependencies... done!
[binary  N     ] sys-apps/busybox-1.25.1-1::musl to /emerge-root/ USE="make-symlinks static -debug -ipv6 -livecd -math -mdev -pam -savedconfig (-selinux) -sep-usr -syslog (-systemd)" 0 KiB

Total: 1 package (1 new, 1 binary), Size of downloads: 0 KiB

>>> Emerging binary (1 of 1) sys-apps/busybox-1.25.1::musl for /emerge-root/
>>> Installing (1 of 1) sys-apps/busybox-1.25.1::musl to /emerge-root/
>>> Jobs: 1 of 1 complete                           Load avg: 1.53, 0.65, 0.24
>>> Auto-cleaning packages...

>>> Using system located in ROOT tree /emerge-root/

>>> No outdated packages were found on your system.

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.

commit kubler-busybox-874-8355 as kubler/bob-musl-busybox:20170925
sha256:31cc2c7bbdb2e1119881736e64769ac7ef6be93b674f1b9c19dd18a07cf567bd
kubler-busybox-874-8355
tag kubler/bob-musl-busybox:latest
--> phase 2: build kubler/busybox:20170925
Sending build context to Docker daemon   1.37MB
Step 1/4 : FROM scratch
--->
Step 2/4 : LABEL maintainer Erik Dannenberg <erik.dannenberg@xtrade-gmbh.de>
---> Using cache
---> 04ce577ae834
Step 3/4 : ADD rootfs.tar /
---> edb1aa80484b
Removing intermediate container 47657a7125c4
Step 4/4 : CMD /bin/sh
---> Running in c384628b54fb
---> cb004f747882
Removing intermediate container c384628b54fb
Successfully built cb004f747882
Successfully tagged kubler/busybox:20170925
tag kubler/busybox:latest
--> build image kubler/glibc
Untagged: kubler/glibc:20170925
--> phase 1: building root fs
using kubler/bob:20170925
* Generating locale-archive: forcing # of jobs to 1
* Generating 2 locales (this might take a while) with 1 jobs
*  (1/2) Generating en_US.ISO-8859-1 ...                                                                                                                                                                    [ ok ]
*  (2/2) Generating en_US.UTF-8 ...                                                                                                                                                                         [ ok ]
* Generation complete

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.


These are the packages that would be merged, in order:

Calculating dependencies... done!
[binary  N     ] sys-apps/gentoo-functions-0.12-1::gentoo to /emerge-root/ 0 KiB
[binary  N     ] virtual/libintl-0-r2-1::gentoo to /emerge-root/ 0 KiB
[binary  N     ] sys-libs/timezone-data-2017a-1::gentoo to /emerge-root/ USE="nls -leaps_timezone" 0 KiB
[binary  N     ] sys-libs/glibc-2.23-r4-1:2.2::gentoo to /emerge-root/ USE="hardened rpc -audit -caps -debug -gd (-multilib) -nscd (-profile) (-selinux) -suid -systemtap (-vanilla)" 0 KiB

Total: 4 packages (4 new, 4 binaries), Size of downloads: 0 KiB

>>> Running pre-merge checks for sys-libs/glibc-2.23-r4
* glibc-2.23-r4-1.xpak MD5 SHA1 size ;-) ...                            [ ok ]
>>> Emerging binary (1 of 4) sys-apps/gentoo-functions-0.12::gentoo for /emerge-root/
>>> Installing (1 of 4) sys-apps/gentoo-functions-0.12::gentoo to /emerge-root/
>>> Emerging binary (2 of 4) virtual/libintl-0-r2::gentoo for /emerge-root/
>>> Installing (2 of 4) virtual/libintl-0-r2::gentoo to /emerge-root/
>>> Emerging binary (3 of 4) sys-libs/timezone-data-2017a::gentoo for /emerge-root/
>>> Installing (3 of 4) sys-libs/timezone-data-2017a::gentoo to /emerge-root/
>>> Emerging binary (4 of 4) sys-libs/glibc-2.23-r4::gentoo for /emerge-root/
>>> Installing (4 of 4) sys-libs/glibc-2.23-r4::gentoo to /emerge-root/
>>> Recording sys-libs/glibc in "world" favorites file...
>>> Jobs: 4 of 4 complete                           Load avg: 1.35, 0.68, 0.27

* Messages for package sys-libs/glibc-2.23-r4 merged to /emerge-root/:

* Defaulting /etc/host.conf:multi to on
>>> Auto-cleaning packages...

>>> Using system located in ROOT tree /emerge-root/

>>> No outdated packages were found on your system.

* IMPORTANT: config file '/emerge-root/etc/locale.gen' needs updating.
* See the CONFIGURATION FILES and CONFIGURATION FILES UPDATE TOOLS
* sections of the emerge man page to learn how to update config files.

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.

tar: Removing leading `/' from member names
tar: Removing leading `/' from member names
tar: Removing leading `/' from member names
commit kubler-glibc-874-27201 as kubler/bob-glibc:20170925
sha256:0b9c1e7e5568e4cad7500dae9dd63c237c60b9b74c2b327aaaed6ee1cd2b6264
kubler-glibc-874-27201
tag kubler/bob-glibc:latest
--> phase 2: build kubler/glibc:20170925
Sending build context to Docker daemon  9.574MB
Step 1/5 : FROM kubler/busybox
---> cb004f747882
Step 2/5 : LABEL maintainer Erik Dannenberg <erik.dannenberg@xtrade-gmbh.de>
---> Running in ff28c82de7f3
---> 593e959110bc
Removing intermediate container ff28c82de7f3
Step 3/5 : ENV LANG en_US.utf8
---> Running in 7739b3e66065
---> 0bb42bba7083
Removing intermediate container 7739b3e66065
Step 4/5 : ADD rootfs.tar /
---> fa54790fe94c
Removing intermediate container a66e43c977a9
Step 5/5 : RUN ldconfig
---> Running in ff022bacf5e9
---> 3f9b13d37be2
Removing intermediate container ff022bacf5e9
Successfully built 3f9b13d37be2
Successfully tagged kubler/glibc:20170925
tag kubler/glibc:latest
--> build image mynamespace/nmap
Untagged: mynamespace/nmap:20170925
--> phase 1: building root fs
using kubler/bob-glibc:20170925

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.


These are the packages that would be merged, in order:

Calculating dependencies... done!
[binary  N     ] app-arch/bzip2-1.0.6-r8-1:0/1::gentoo to /emerge-root/ USE="-static -static-libs" 0 KiB
[binary  N     ] app-misc/c_rehash-1.7-r1-1::gentoo to /emerge-root/ 0 KiB
[binary  N     ] sys-libs/zlib-1.2.11-r1-1:0/1::gentoo to /emerge-root/ USE="-minizip -static-libs" 0 KiB
[binary  N     ] net-libs/libpcap-1.8.1-1::gentoo to /emerge-root/ USE="-bluetooth -dbus -netlink -static-libs -usb" 0 KiB
[binary  N     ] dev-libs/liblinear-210-r1-1:0/3::gentoo to /emerge-root/ USE="-blas" 0 KiB
[binary  N     ] sys-apps/debianutils-4.7-1::gentoo to /emerge-root/ USE="-static" 0 KiB
[binary  N     ] sys-libs/ncurses-6.0-r1-2:0/6::gentoo to /emerge-root/ USE="cxx threads unicode -ada -debug -doc -gpm -minimal (-profile) -static-libs {-test} -tinfo -trace" 0 KiB
[binary  N     ] virtual/libintl-0-r2-1::gentoo to /emerge-root/ 0 KiB
[binary  N     ] app-misc/ca-certificates-20161130.3.30.2-1::gentoo to /emerge-root/ USE="-cacert -insecure_certs" 0 KiB
[binary  N     ] sys-libs/readline-6.3_p8-r3-1::gentoo to /emerge-root/ USE="-static-libs -utils" 0 KiB
[binary  N     ] dev-libs/libpcre-8.41-1:3::gentoo to /emerge-root/ USE="bzip2 cxx readline recursion-limit (unicode) zlib -jit -libedit -pcre16 -pcre32 -static-libs" 0 KiB
[binary  N     ] dev-libs/openssl-1.0.2l-1::gentoo to /emerge-root/ USE="asm sslv3 tls-heartbeat zlib -bindist -gmp -kerberos -rfc3779 -sctp -sslv2 -static-libs {-test} -vanilla" CPU_FLAGS_X86="(sse2)" 0 KiB
[binary  N     ] net-analyzer/nmap-7.40-1::gentoo to /emerge-root/ USE="nls nse ssl -ipv6 (-libressl) -ncat -ndiff -nmap-update -nping (-system-lua) -zenmap" LINGUAS="-de -fr -hi -hr -it -ja -pl -pt_BR -ru -zh" PYTHON_TARGETS="python2_7" 0 KiB

Total: 13 packages (13 new, 13 binaries), Size of downloads: 0 KiB

>>> Emerging binary (1 of 13) app-arch/bzip2-1.0.6-r8::gentoo for /emerge-root/
>>> Installing (1 of 13) app-arch/bzip2-1.0.6-r8::gentoo to /emerge-root/
>>> Emerging binary (2 of 13) app-misc/c_rehash-1.7-r1::gentoo for /emerge-root/
>>> Installing (2 of 13) app-misc/c_rehash-1.7-r1::gentoo to /emerge-root/
>>> Emerging binary (3 of 13) sys-libs/zlib-1.2.11-r1::gentoo for /emerge-root/
>>> Installing (3 of 13) sys-libs/zlib-1.2.11-r1::gentoo to /emerge-root/
>>> Emerging binary (4 of 13) net-libs/libpcap-1.8.1::gentoo for /emerge-root/
>>> Installing (4 of 13) net-libs/libpcap-1.8.1::gentoo to /emerge-root/
>>> Emerging binary (5 of 13) dev-libs/liblinear-210-r1::gentoo for /emerge-root/
>>> Installing (5 of 13) dev-libs/liblinear-210-r1::gentoo to /emerge-root/
>>> Emerging binary (6 of 13) sys-apps/debianutils-4.7::gentoo for /emerge-root/
>>> Installing (6 of 13) sys-apps/debianutils-4.7::gentoo to /emerge-root/
>>> Emerging binary (7 of 13) sys-libs/ncurses-6.0-r1::gentoo for /emerge-root/
>>> Installing (7 of 13) sys-libs/ncurses-6.0-r1::gentoo to /emerge-root/
>>> Emerging binary (8 of 13) virtual/libintl-0-r2::gentoo for /emerge-root/
>>> Installing (8 of 13) virtual/libintl-0-r2::gentoo to /emerge-root/
>>> Emerging binary (9 of 13) app-misc/ca-certificates-20161130.3.30.2::gentoo for /emerge-root/
>>> Installing (9 of 13) app-misc/ca-certificates-20161130.3.30.2::gentoo to /emerge-root/
>>> Emerging binary (10 of 13) sys-libs/readline-6.3_p8-r3::gentoo for /emerge-root/
>>> Installing (10 of 13) sys-libs/readline-6.3_p8-r3::gentoo to /emerge-root/
>>> Emerging binary (11 of 13) dev-libs/libpcre-8.41::gentoo for /emerge-root/
>>> Installing (11 of 13) dev-libs/libpcre-8.41::gentoo to /emerge-root/
>>> Emerging binary (12 of 13) dev-libs/openssl-1.0.2l::gentoo for /emerge-root/
>>> Installing (12 of 13) dev-libs/openssl-1.0.2l::gentoo to /emerge-root/
>>> Emerging binary (13 of 13) net-analyzer/nmap-7.40::gentoo for /emerge-root/
>>> Installing (13 of 13) net-analyzer/nmap-7.40::gentoo to /emerge-root/
>>> Recording net-analyzer/nmap in "world" favorites file...
>>> Jobs: 13 of 13 complete                         Load avg: 1.32, 0.76, 0.32
>>> Auto-cleaning packages...

>>> Using system located in ROOT tree /emerge-root/

>>> No outdated packages were found on your system.

* IMPORTANT: 3 news items need reading for repository 'gentoo'.
* Use eselect news read to view new items.

tar: Removing leading `/' from member names
commit mynamespace-nmap-874-2470 as mynamespace/bob-nmap:20170925
sha256:23bdf8c49ca34ee8fd3319f7d527ab0b58691642f03c0c96a9d0df27ce3189af
mynamespace-nmap-874-2470
tag mynamespace/bob-nmap:latest
--> phase 2: build mynamespace/nmap:20170925
Sending build context to Docker daemon   37.2MB
Step 1/3 : FROM kubler/glibc
---> 3f9b13d37be2
Step 2/3 : LABEL maintainer El Ttam <el.ttam@local>
---> Running in f4a1a2cdc806
---> dd2aa41eff81
Removing intermediate container f4a1a2cdc806
Step 3/3 : ADD rootfs.tar /
---> ae56224c80ef
Removing intermediate container 6fc94f72ee0f
Successfully built ae56224c80ef
Successfully tagged mynamespace/nmap:20170925
tag mynamespace/nmap:latest

Now that the build is complete we get a PACKAGES.md that shows what packages/libraries and their versions it contains. An example is shown below: -
mynamespace/nmap:20170925

Built: Wed Oct 18 00:14:10 GMT 2017 Image Size: 44.6MB
Installed
Package	USE Flags
app-arch/bzip2-1.0.6-r8	-static -static-libs
app-misc/ca-certificates-20161130.3.30.2	-cacert -insecure
app-misc/c_rehash-1.7-r1	``
dev-libs/liblinear-210-r1	-blas
dev-libs/libpcre-8.41	bzip2 cxx readline recursion-limit (unicode) zlib -jit -libedit -pcre16 -pcre32 -static-libs
dev-libs/openssl-1.0.2l	asm sslv3 tls-heartbeat zlib -bindist -gmp -kerberos -rfc3779 -sctp -sslv2 -static-libs {-test} -vanilla
net-analyzer/nmap-7.40	nls nse ssl -ipv6 (-libressl) -ncat -ndiff -nmap-update -nping (-system-lua) -zenmap
net-libs/libpcap-1.8.1	-bluetooth -dbus -netlink -static-libs -usb
sys-apps/debianutils-4.7	-static
sys-libs/ncurses-6.0-r1	cxx threads unicode -ada -debug -doc -gpm -minimal (-profile) -static-libs {-test} -tinfo -trace
sys-libs/readline-6.3_p8-r3	-static-libs -utils
sys-libs/zlib-1.2.11-r1	-minizip -static-libs
Inherited
Package	USE Flags
FROM kubler/glibc
sys-apps/gentoo-functions-0.12	``
sys-libs/glibc-2.23-r4	hardened rpc -audit -caps -debug -gd (-multilib) -nscd (-profile) (-selinux) -suid -systemtap (-vanilla)
sys-libs/timezone-data-2017a	nls -leaps
FROM kubler/busybox
sys-apps/busybox-1.25.1	make-symlinks static -debug -ipv6 -livecd -math -mdev -pam -savedconfig (-selinux) -sep-usr -syslog -systemd
Purged

    [x] Headers
    [x] Static Libs

The above can be used to combat image rot, it is easy to see what versions of libraries and packages are being used in the image. This can easily be integrated for automated searching of vulnerable packages/libraries.
Building static nmap binaries

Now lets say for some reason you want a statically compiled nmap, ncat, and nping binaries. Say you want to run it on some target Linux system that doesn’t have Docker Engine and you don’t want to have to deal with incompatible libraries or copying the whole dependency graph of libraries over. You want a statically built nmap that will just run on just about any Linux.

A good guide can be found at ZeroSec’s blog post How to Statically Compile NMAP . However, the result is still dynamically compiled due to glibc. It’s not as portable as it could be, it won’t run out-of-the-box on Linux systems that use a different glibc.
Building static nmap binaries with Kubler

You can leverage Kubler to build a static nmap. Kubler builds Linux rootfs file systems, as part of building the Docker image. The resulting rootfs.tar can be copied to a target system, untarred, to extract the nmap binary, and it’s data files, and NSE scripts.

But first, a trap for young players is that the underlying Gentoo/Portage doesn’t have a static USE flag for nmap. So we’ll have to whip out some Gentoo kungfu and make it do what we want.

We need to modify the nmap ebuild so that it will build a static binary rather than a dynamically linked one. The diff we need is: -

diff --git a/net-analyzer/nmap/nmap-7.40.ebuild b/net-analyzer/nmap/nmap-7.40.ebuild
index 1582792..17f82d5 100644
--- a/net-analyzer/nmap/nmap-7.40.ebuild
+++ b/net-analyzer/nmap/nmap-7.40.ebuild
@@ -20,7 +20,7 @@ LICENSE="GPL-2"
 SLOT="0"
 KEYWORDS="alpha amd64 arm ~arm64 hppa ia64 ~mips ppc ppc64 ~s390 ~sh sparc x86 ~x86-fbsd ~amd64-linux ~arm-linux ~x86-linux ~ppc-macos ~x64-macos ~x86-macos ~sparc-solaris ~x86-solaris"

-IUSE="ipv6 libressl +nse system-lua ncat ndiff nls nmap-update nping ssl zenmap"
+IUSE="ipv6 libressl +nse system-lua ncat ndiff nls nmap-update nping ssl zenmap static"
 NMAP_LINGUAS=( de fr hi hr it ja pl pt_BR ru zh )
 IUSE+=" ${NMAP_LINGUAS[@]/#/linguas_}"

@@ -32,8 +32,14 @@ REQUIRED_USE="

 RDEPEND="
 	dev-libs/liblinear:=
-	dev-libs/libpcre
-	net-libs/libpcap
+	static? (
+		dev-libs/libpcre[static-libs(+)]
+		net-libs/libpcap[static-libs(+)]
+	)
+	!static? (
+		dev-libs/libpcre
+		net-libs/libpcap
+	)
 	zenmap? (
 		dev-python/pygtk:2[${PYTHON_USEDEP}]
 		${PYTHON_DEPS}
@@ -123,6 +129,14 @@ src_prepare() {
 }

 src_configure() {
+	# static
+	use static && append-cflags -static -static-libgcc
+	use static && append-cxxflags -static -static-libstdc++ -static-libgcc
+	use static && append-ldflags -Wl,-static -Wl,--eh-frame-hdr -fuse-ld=gold -static
+	elog "CFLAGS=$CFLAGS"
+	elog "CXXFLAGS=$CXXFLAGS"
+	elog "LDFLAGS=$LDFLAGS"
+
 	# The bundled libdnet is incompatible with the version available in the
 	# tree, so we cannot use the system library here.
 	econf \

How to manage the custom ebuild? There’s several ways to achieve this but I think the best way is to make a Gentoo Ebuild Repository, aka an “overlay”. This is how to create one from scratch based off the guide in the Handbook: -

kubler build -i mynamespace/nmap
OVERLAY=best-overlay
mkdir -p ${OVERLAY}/{metadata,profiles}
cd ${OVERLAY}
echo $OVERLAY > profiles/repo_name
cat > metadata/layout.conf <<EOF
masters = gentoo
auto-sync = false
EOF
rsync -av /var/sync/portage/net-analyzer/nmap net-analyzer/
bind '\C-i:self-insert'     # disable tab-completion
patch -p1 <<EOF
diff --git a/net-analyzer/nmap/nmap-7.40.ebuild b/net-analyzer/nmap/nmap-7.40.ebuild
index 1582792..17f82d5 100644
--- a/net-analyzer/nmap/nmap-7.40.ebuild
+++ b/net-analyzer/nmap/nmap-7.40.ebuild
@@ -20,7 +20,7 @@ LICENSE="GPL-2"
 SLOT="0"
 KEYWORDS="alpha amd64 arm ~arm64 hppa ia64 ~mips ppc ppc64 ~s390 ~sh sparc x86 ~x86-fbsd ~amd64-linux ~arm-linux ~x86-linux ~ppc-macos ~x64-macos ~x86-macos ~sparc-solaris ~x86-solaris"

-IUSE="ipv6 libressl +nse system-lua ncat ndiff nls nmap-update nping ssl zenmap"
+IUSE="ipv6 libressl +nse system-lua ncat ndiff nls nmap-update nping ssl zenmap static"
 NMAP_LINGUAS=( de fr hi hr it ja pl pt_BR ru zh )
 IUSE+=" ${NMAP_LINGUAS[@]/#/linguas_}"

@@ -32,8 +32,14 @@ REQUIRED_USE="

 RDEPEND="
 	dev-libs/liblinear:=
-	dev-libs/libpcre
-	net-libs/libpcap
+	static? (
+		dev-libs/libpcre[static-libs(+)]
+		net-libs/libpcap[static-libs(+)]
+	)
+	!static? (
+		dev-libs/libpcre
+		net-libs/libpcap
+	)
 	zenmap? (
 		dev-python/pygtk:2[${PYTHON_USEDEP}]
 		${PYTHON_DEPS}
@@ -123,6 +129,14 @@ src_prepare() {
 }

 src_configure() {
+	# static
+	use static && append-cflags -static -static-libgcc
+	use static && append-cxxflags -static -static-libstdc++ -static-libgcc
+	use static && append-ldflags -Wl,-static -Wl,--eh-frame-hdr -fuse-ld=gold -static
+	elog "CFLAGS=$CFLAGS"
+	elog "CXXFLAGS=$CXXFLAGS"
+	elog "LDFLAGS=$LDFLAGS"
+
 	# The bundled libdnet is incompatible with the version available in the
 	# tree, so we cannot use the system library here.
EOF
bind '\C-i:complete'        # reenable tab-completion
ebuild net-analyzer/nmap/nmap-7.40.ebuild manifest
git init
git add .
git config user.name 'El Ttam'
git config user.email 'el.ttam@local'
git commit -m 'Initial Commit'
git bundle create /config/best-overlay.bundle --all
exit

We now have created an overlay and saved it in a git bundle in the mounted /config directory which is the dock/mynamespace/images/nmap directory on the host. You can now clone it somewhere, e.g.: -

cd ..
git clone kubler/dock/mynamespace/images/nmap/best-overlay.bundle

Ideally you’d host your Gentoo Overlay git repository somewhere, like on GitHub. Or, you can use my ready made one at Berney’s Overlay.

Create a new image for the static nmap build, this time we are going to use musl instead of glibc.

localhost:/var/home/user/co/git/kubler$  kubler new image mynamespace/nmap-musl-static

<enter> to accept default value

Extend an existing image? Fully qualified image id (i.e. kubler/busybox) if yes or scratch
Parent Image (scratch): kubler/libressl-musl

Successfully created mynamespace/nmap-musl-static image at /var/home/user/co/git/kubler/dock/mynamespace/images/nmap-musl-static

Edit dock/mynamespace/images/nmap-musl-static/build.sh so that it’s like this: -

_packages="net-analyzer/nmap"

configure_bob()
{
    # Add our custom overlay
    add_overlay berne https://github.com/berney/gentoo-overlay.git

    # Packages installed in this hook don't end up in the final image but are available for depending image builds
    #emerge dev-lang/go app-misc/foo
    :
    # our package
    update_use 'net-analyzer/nmap' '+ipv6' '+libressl' '+ncat' '-ndiff' '-nls' '-nmap-update' '+nping' '+nse' '+ssl' '-system-lua' '-zenmap' '+static'
    # targeted
    update_use 'net-libs/libpcap' '+static-libs'
    update_use 'dev-libs/libpcre' '+static-libs'
    update_use 'dev-lang/python' '+sqlite'
    update_use 'dev-libs/libressl' '+static-libs'

    # Need to unprovide libressl so that it will be rebuilt
    # This can be useful to install a package from a parent image again, it may be needed at build time
    unprovide_package dev-libs/libressl

    # emerge in builder to pull in dependencies
    emerge -vt net-analyzer/nmap
}

#
# This hook is called just before starting the build of the root fs
#
configure_rootfs_build()                                                                                                                                                                                           {
    # Update a Gentoo package use flag..                                                                                                                                                                               #update_use 'dev-libs/some-lib' '+feature' '-some_other_feature'

    # ..or a Gentoo package keyword
    #update_keywords 'dev-lang/some-package' '+~amd64'

    # Add a package to Portage's package.provided file, effectively skipping it during installation
    #provide_package 'dev-lang/some-package'
    provide_package 'net-libs/libpcap'
    provide_package 'sys-libs/zlib'
    provide_package 'sys-libs/ncurses'
    provide_package 'sys-libs/readline'
    provide_package 'dev-libs/libpcre'
    provide_package 'dev-libs/liblinear'
    provide_package 'dev-libs/libressl'

    # This can be useful to install a package from a parent image again, it may be needed at build time
    #unprovide_package 'dev-lang/some-package'

    # Only needed when ${_packages} is empty, initializes PACKAGES.md
    #init_docs "berne/nmap-musl-static"

    # emerge in EMERGE_ROOT, we should already have all the dependencies built
    :
}

#
# This hook is called just before packaging the root fs tar ball, ideal for any post-install tasks, clean up, etc
#
finish_rootfs_build()
{
    # Useful helpers

    # install su-exec at ${_EMERGE_ROOT}
    #install_suexec
    # Copy c++ libs, may be needed if you see errors regarding missing libstdc++
    #copy_gcc_libs

    # Example for a manual build if _packages method does not suffice, a typical use case is a Go project:

    #export GOPATH="/go"
    #export PATH="${PATH}:/go/bin"
    #export DISTRIBUTION_DIR="${GOPATH}/src/github.com/berne/nmap-musl-static"
    #mkdir -p "${DISTRIBUTION_DIR}"

    #git clone https://github.com/berne/nmap-musl-static.git "${DISTRIBUTION_DIR}"
    #cd "${DISTRIBUTION_DIR}"
    #git checkout tags/v${_nmap-musl-static_version}
    #echo "building nmap-musl-static ${_nmap-musl-static_version}.."
    #go run build.go build
    #mkdir -p "${_EMERGE_ROOT}"/usr/local/{bin,share}

    # Everything at ${_EMERGE_ROOT} will end up in the final image
    #cp -rp "${DISTRIBUTION_DIR}/bin/*" "${_EMERGE_ROOT}/usr/local/bin"

    # Rice Rice Baby - rm stuff we don't want in the final image
    rm -rf "${_EMERGE_ROOT}"/etc
    rm -rf "${_EMERGE_ROOT}"/tmp
    rm -rf "${_EMERGE_ROOT}"/var
    # to run as a user you need /etc/{passwd,group}
    #mkdir -p "${_EMERGE_ROOT}"/etc
    # handle bug in portage when using custom root, user/groups created during install are not created at the custom root but on the host
    #cp -f /etc/{passwd,group} "${_EMERGE_ROOT}"/etc/

    # After installing packages manually you might want to add an entry to PACKAGES.md
    #log_as_installed "manual install" "nmap-musl-static-${_nmap-musl-static_version}" "https://nmap-musl-static.org/"
    :
}

Now build your the mynamespace/nmap-musl-static image, with kubler build mynamespace/nmap-musl-static. You can rebuild just the mynamespace/nmap-musl-static image without rebuilding all the dependencies by using the -nF arguments to kubler build, as shown below: -

localhost:/var/home/user/co/git/kubler$ kubler build mynamespace/nmap-musl-static -nF
--> build image mynamespace/nmap-musl-static
Untagged: mynamespace/nmap-musl-static:20170925
--> phase 1: building root fs
using kubler/bob-musl-libressl-musl:20170925
>>> Syncing repository 'berne' into '/var/lib/repos/berne'...
/usr/bin/git clone --depth 1 https://github.com/berney/gentoo-overlay.git .
Cloning into '.'...
remote: Counting objects: 24, done.
remote: Compressing objects: 100% (19/19), done.
remote: Total 24 (delta 0), reused 19 (delta 0), pack-reused 0
Unpacking objects: 100% (24/24), done.
=== Sync completed for berne
* In post-repository hook for berne
** synced from remote repository https://github.com/berney/gentoo-overlay.git
** synced into /var/lib/repos/berne
 * Updating metadata cache for overlay repo 'berne' ...                                                                                                                                                      [ ok ]
Reading Portage settings...
Building database (/var/cache/eix/portage.eix)...
[0] "gentoo" /var/sync/portage/ (cache: metadata-md5-or-flat)
     Reading category 164|164 (100) Finished
[1] "berne" /var/lib/repos/berne (cache: parse|ebuild*#metadata-md5#metadata-flat#assign)
     Reading category 164|164 (100) Finished
[2] "libressl" /var/lib/layman/libressl (cache: parse|ebuild*#metadata-md5#metadata-flat#assign)
     Reading category 164|164 (100) Finished
[3] "musl" /var/lib/layman/musl (cache: parse|ebuild*#metadata-md5#metadata-flat#assign)
     Reading category 164|164 (100) Finished
Applying masks...
Calculating hash tables...
Writing database file /var/cache/eix/portage.eix...
Database contains 19633 packages in 164 categories.
[>]   == media-video/ffmpeg (3.3.4(0/55.57.57)^d -> 3.3.4(0/55.57.57)^d[2]): Complete solution to record, convert and stream audio and video. Includes libavcodec

 * IMPORTANT: 11 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.


Action: sync for repo: berne, returned code = 0



 * IMPORTANT: 11 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.


These are the packages that would be merged, in reverse order:

Calculating dependencies... done!
[binary  N     ] net-analyzer/nmap-7.40-1::berne  USE="ipv6 libressl ncat nping nse ssl static -ndiff -nls -nmap-update (-system-lua) -zenmap" LINGUAS="-de -fr -hi -hr -it -ja -pl -pt_BR -ru -zh" PYTHON_TARGETS="python2_7" 0 KiB
[binary   R    ]  dev-libs/libpcre-8.41-1:3::gentoo  USE="cxx readline recursion-limit static-libs* (unicode) zlib -bzip2 -jit -libedit -pcre16 -pcre32" 0 KiB
[binary   R    ]  dev-libs/libressl-2.6.0-1:0/43::gentoo  USE="asm static-libs*" 0 KiB
[binary  N     ]  net-libs/libpcap-1.8.1-1::gentoo  USE="static-libs -bluetooth -dbus -netlink -usb" 0 KiB
[binary  N     ]  dev-libs/liblinear-210-r1-1:0/3::gentoo  USE="-blas" 0 KiB

Total: 5 packages (3 new, 2 reinstalls, 5 binaries), Size of downloads: 0 KiB

>>> Emerging binary (1 of 5) dev-libs/liblinear-210-r1::gentoo
>>> Installing (1 of 5) dev-libs/liblinear-210-r1::gentoo
>>> Emerging binary (2 of 5) net-libs/libpcap-1.8.1::gentoo
>>> Installing (2 of 5) net-libs/libpcap-1.8.1::gentoo
>>> Emerging binary (3 of 5) dev-libs/libressl-2.6.0::gentoo
>>> Installing (3 of 5) dev-libs/libressl-2.6.0::gentoo
>>> Emerging binary (4 of 5) dev-libs/libpcre-8.41::gentoo
>>> Installing (4 of 5) dev-libs/libpcre-8.41::gentoo
>>> Emerging binary (5 of 5) net-analyzer/nmap-7.40::berne
>>> Installing (5 of 5) net-analyzer/nmap-7.40::berne
>>> Recording net-analyzer/nmap in "world" favorites file...
>>> Jobs: 5 of 5 complete                           Load avg: 0.66, 0.49, 1.21
>>> Auto-cleaning packages...

>>> No outdated packages were found on your system.

 * IMPORTANT: 11 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.


 * IMPORTANT: 3 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.


These are the packages that would be merged, in order:

Calculating dependencies... done!
[binary  N     ] net-analyzer/nmap-7.40-1::berne to /emerge-root/ USE="ipv6 libressl ncat nping nse ssl static -ndiff -nls -nmap-update (-system-lua) -zenmap" LINGUAS="-de -fr -hi -hr -it -ja -pl -pt_BR -ru -zh" PYTHON_TARGETS="python2_7" 0 KiB

Total: 1 package (1 new, 1 binary), Size of downloads: 0 KiB

>>> Emerging binary (1 of 1) net-analyzer/nmap-7.40::berne for /emerge-root/
>>> Installing (1 of 1) net-analyzer/nmap-7.40::berne to /emerge-root/
>>> Recording net-analyzer/nmap in "world" favorites file...
>>> Jobs: 1 of 1 complete                           Load avg: 0.89, 0.54, 1.22
>>> Auto-cleaning packages...

>>> Using system located in ROOT tree /emerge-root/

>>> No outdated packages were found on your system.

 * IMPORTANT: 3 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.

commit mynamespace-nmap-musl-static-10855-13900 as mynamespace/bob-musl-nmap-musl-static:20170925
sha256:4208ec76dd6e5f13a62abda95144725dfc1e72754afc88c2a45fc336f56f2a42
mynamespace-nmap-musl-static-10855-13900
tag mynamespace/bob-musl-nmap-musl-static:latest
--> phase 2: build mynamespace/nmap-musl-static:20170925
Sending build context to Docker daemon   61.4MB
Step 1/5 : FROM kubler/libressl-musl
 ---> 365264edde16
Step 2/5 : LABEL maintainer El Ttam <el.ttam@local>
 ---> Using cache
 ---> 7720ced35795
Step 3/5 : ADD rootfs.tar /
 ---> 4e02070b0a7c
Removing intermediate container f4fc9ce3bc9f
Step 4/5 : ENTRYPOINT /usr/bin/nmap
 ---> Running in b12a020ee0b2
 ---> 3274cfc59a2e
Removing intermediate container b12a020ee0b2
Step 5/5 : VOLUME nmap-log:/data
 ---> Running in 41e109fe1de1
 ---> e7e1d299f392
Removing intermediate container 41e109fe1de1
Successfully built e7e1d299f392
Successfully tagged mynamespace/nmap-musl-static:20170925
tag mynamespace/nmap-musl-static:latest
localhost:/var/home/user/co/git/kubler$

Note that when rebuilding, packages that have been emerged previously will now be installed as binary packages. This speeds up rebuilds. New packages, or packages with different USE flags, if they haven’t been emerged previously will be built from source, but once built from source the binary packages are availabe for quick installation. This brings the flexibility of source-compiling and the efficiency of binary packages. When used, another build speed-up comes with the inheritance of Kubler images, the predecesor images only need to be built once, then leaf images can be built, and rebuilt, without rebuilding the predecessors.

Run the new mynamespace/nmap-musl-static image to test that it works; we can check the version and compilation options with nmap’s --version argument: -

localhost:/var/home/user/co/git/kubler$ docker run --rm -it mynamespace/nmap-musl-static --version

Nmap version 7.40 ( https://nmap.org )
Platform: x86_64-gentoo-linux-musl
Compiled with: liblua-5.3.3 openssl-2.6.0 libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12 ipv6
Compiled without:
Available nsock engines: epoll poll select
localhost:/var/home/user/co/git/kubler$

To extract the files, just untar the rootfs.tar file. e.g.; -

localhost:/var/home/user/co/git/kubler$ cd ..
localhost:/var/home/user/co/git$ mkdir nmap-musl-static
localhost:/var/home/user/co/git$ cd nmap-musl-static/
localhost:/var/home/user/co/git/nmap-musl-static$ tar xf ../kubler/dock/mynamespace/images/nmap-musl-static/rootfs.tar
localhost:/var/home/user/co/git/nmap-musl-static$ tree -AF --filelimit 12
.
└── usr/
    ├── bin/
    │   ├── ncat*
    │   ├── nmap*
    │   └── nping*
    └── share/
        ├── ncat/
        │   └── ca-bundle.crt
        └── nmap/
            ├── nmap-mac-prefixes
            ├── nmap-os-db
            ├── nmap-payloads
            ├── nmap-protocols
            ├── nmap-rpc
            ├── nmap-service-probes
            ├── nmap-services
            ├── nmap.dtd
            ├── nmap.xsl
            ├── nse_main.lua
            ├── nselib/ [126 entries exceeds filelimit, not opening dir]
            └── scripts/ [553 entries exceeds filelimit, not opening dir]

7 directories, 14 files

And the binaries are static: -

localhost:/var/home/user/co/git/nmap-musl-static$ file usr/bin/*
usr/bin/ncat:  ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped
usr/bin/nmap:  ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped
usr/bin/nping: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped
localhost:/var/home/user/co/git/nmap-musl-static$

Accepting Gentoo Unstable Keywords

Gentoo uses nmap-7.40 as the latest stable version on AM64, but the last stable (upstream) is 7.60. If you want the latest stable nmap, we need to accept the AMD64 testing keyword. In the build.sh file add update_keywords '=net-analyzer/nmap-7.60' '+~amd64' after the the add_overlay line and before the emerge line.

If you want to use bleeding edge (git) version of nmap, add update_keywords '=net-analyzer/nmap-9999' '+**'.
Feature/Size Comparisons

It can be hard to make a fair comparison between pre-built nmap images and Kubler built ones due to the version drift and feature set. The following table shows variants of Kubler built nmap images and some alternatives.
Build	Version	Image Size	Binary Size	Data Size	Features
berne/nmap-musl-static	7.60	31.1	6.0	19.6	liblua-5.3.3 openssl-2.6.0 libssh2-1.8.0 libz-1.2.8 libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12 ipv6
berne/nmap-musl-static:ipv6	7.60	13.0	4.0	8.4	libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12 ipv6
berne/nmap-musl-static:min	7.60	13.0	4.0	8.4	libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12
berne/nmap-musl-static:uzyexe	7.12	12.2	4.1	7.6	libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12 ipv6
uzyexe/nmap	7.12	16.7	4.3	7.6	nmap-libpcre-7.6 libpcap-1.7.4 nmap-libdnet-1.12 ipv6
berne/nmap-musl-static:andrew-d	6.49BETA1	22.0	5.8	16.5	liblua-5.2.3 openssl-2.6.0 libpcre-8.41 libpcap-1.8.1 nmap-libdnet-1.12 ipv6
andrew-d/static-binaries	6.49BETA1	N/A	5.7	N/A	nmap-liblua-5.2.3 openssl-1.0.2c nmap-libpcre-7.6 nmap-libpcap-1.7.3 nmap-libdnet-1.12 ipv6

    Image size is the size of the Docker image.
    Binary size is the size of the nmap binary and (if applicable) all dependent shared objects.
    Data size is the size of /usr/share/nmap, which includes OS Fingerprints, MAC Address and service lookup tables, and NSE scripts, and so on.
    All sizes are MiB.

Conclusion

Kubler is a flexible and powerfull tool to provide a clean build environment and enable repeatable builds of Docker images. In the in depth example above, we have used Kubler to create an image that is up-to-date, fully featured, and statically compiled. It is easy to select the desired feature set, enabling or disabling the desired features, such as in order to build tools for resource-constrained environments such as embedded devices.

Using Kubler we were able to build the smallest nmap static binary, which compared to the alternatives is the latest stable version and built with a hardened tool-chain. We can also produce a Docker image of the latest stable version with same feature-set as uzyexe/nmap, however it’s lighter despite being a newer version and a hardened tool-chain.
