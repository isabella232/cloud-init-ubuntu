# Joyent cloud-init package for Ubuntu 14.04

This document is specific to Joyent's custom cloud-init package for Ubuntu
14.04.  There should be a short-term need for this, as these changes should be
upstreamed soon and a new ubuntu-certified image should follow.

The upstream repo is https://git.launchpad.net/ubuntu/+source/cloud-init.

## Setting up your build environment

The following list of packages to install my be incomplete.

```
apt install -y git build-essential quilt python-setuptools python-mocker \
    python-nose python-httpretty python-unittest2 python-mock
git clone -b trusty-joyent https://github.com/joyent/cloud-init-ubuntu.git
```

## Making modifications

Modifications are made my adding patches in the `debian/patches` directory.
`quilt(1)` is used for managing the patches.  To prepare to use quilt, create
`$HOME/.quiltrc` as:

```
# vi: syntax=sh
if [ ! -d patches ]; then
  for where in ./ ../ ../../ ../../../ ../../../../ ../../../../../; do
    if [ -e ${where}debian/rules -a -d ${where}debian/patches ]; then
            export QUILT_PATCHES=debian/patches
            QUILT_PATCHES=debian/patches
            #QUILT_PATCH_OPTS="--unified-reject-files"
            QUILT_DIFF_OPTS="-p"
            QUILT_PATCH_OPTS="--reject-format=unified"
            QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
            QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
            QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
            break
    fi
  done
fi
```

To apply all of the patches:

```
quilt push -a
```

To undo that:

```
quilt pop -a
```

While `quilt` does have the ability to update patches (see `add` and `fold`
subcommands), non-quilters may find something like the following easier.

```
repo=$(pwd)
cd $(dirname $repo)
git clone $repo hack
cd hack
quilt push -a
git add .
git commit -m 'after quilt push -a'
vi cloudinit/sources/DataSourceSmartOS.py
...
git commit -a
git log -p -n 1 >$repo/debian/patches/my-new-patch.patch
cd $repo/debian/patches
echo my-new-patch.patch >> series
git add my-new-patch.patch series
cd $repo
```

When making modifications to this package, be sure to update `debian/changelog`.
The version string found in the first entry is used for setting the package
version.  To distinguish from official packages, the version string contains
'joyent'.  It also has another dot version to indicate the Joyent version of the
package.  Each new changelog entry should increment this last part of the
version string.  `debuild` is very particular about how changelog entries are
formatted.

```
vi debian/changelog
git add debian/changelog
git commit
```

## To build the package

See the statement above about bumping the version number.

From the top-level directory:

```
$ debuild -b -us -uc
dpkg-buildpackage -rfakeroot -D -us -uc -b
... [hundreds of lines elided]
W: cloud-init: binary-without-manpage usr/bin/cloud-init
W: cloud-init: binary-without-manpage usr/bin/cloud-init-per
W: ec2-init: empty-binary-package
N: 3 tags overridden (2 errors, 1 warning)
Finished running lintian.

$ echo $?
0
```

The resulting `.deb` file will be in the parent directory.

```
$ ls ../cloud-init*.deb
../cloud-init_0.7.5-0ubuntu1.22_all.deb
```
