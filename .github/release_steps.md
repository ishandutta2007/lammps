# LAMMPS Release Steps

The following notes chronicle the current steps for preparing and
publishing LAMMPS releases.  For definitions of LAMMPS versions and
releases, please refer to [the corresponding section in the LAMMPS
manual](https://docs.lammps.org/Manual_version.html).

## LAMMPS Feature Release

A LAMMPS feature release is currently prepared after about 500 to 750
commits to the 'develop' branch or after a period of four weeks up to
two months.  This is not a fixed rule, though, since external
circumstances can cause delays in preparing a release, or pull requests
that are desired to be merged for the release are not yet completed.

### Preparing a 'next\_release' branch

Create a 'next\_release' branch off 'develop' and make the following changes:

- set the LAMMPS\_VERSION define to the planned release date in
  src/version.h in the format "D Mmm YYYY" or "DD Mmm YYYY"
- remove the LAMMPS\_UPDATE define in src/version.h
- update the release date in doc/lammps.1
- update all TBD arguments for ..versionadded::, ..versionchanged::
  ..deprecated:: to the planned release date in the format "DMmmYYYY" or
  "DDMmmYYYY"
- check release notes for merged new features and check if
  ..versionadded:: or ..versionchanged:: are missing and need to be
  added

Submit this pull request, rebase if needed.  This is the last pull
request merged for the release and should not contain any other
changes. (Exceptions: this document, last minute trivial(!) changes).

This PR shall not be merged before **all** pending tests have completed
and cleared.  We currently use a mix of automated tests running on
either Temple's Jenkins cluster or GitHub workflows.  Those include time
consuming tests not run on pull requests.  If needed, a bugfix pull
request should be created and merged to clear all tests.

### Create release on GitHub

When all pending pull requests for the release are merged and have
cleared testing, the 'next\_release' branch is merged into 'develop'.

Check out or update the 'develop' branch locally, pull the latest
changes, merge them into 'release' with a fast forward(!) merge, and
apply a suitable release tag (for historical reasons the tag starts with
"patch_" followed by the date, and finally push everything back to
GitHub.  There should be no commits made to 'release' but only
fast forward merges.  Example:

```
git checkout develop
git pull
git checkout release
git pull
git merge --ff-only develop
git tag -s -m "LAMMPS feature release 19 November 2024" patch_19Nov2024
git push git@github.com:lammps/lammps.git --tags develop release
```

Applying this tag will trigger two actions on the Temple Jenkins cluster:
- The online manual at https://docs.lammps.org/ will be updated to the
  state of the 'release' branch.  Merges to the 'develop' branch will
  trigger updating https://docs.lammps.org/latest/ so by reviewing the
  version of the manual under the "latest" URL, it is possible to preview
  what the updated release documentation will look like.
- A downloadable tar archive of the LAMMPS distribution that includes the
  html format documentation and a PDF of the manual will be created and
  uploaded to the download server at https://download.lammps.org/tars
  Note that the file is added, but the `index.html` file is not updated,
  so it is not yet publicly visible.

Go to https://github.com/lammps/lammps/releases and create a new (draft)
release page with a summary of all the changes included and references
to the pull requests they were merged from or check the existing draft
for any necessary changes from pull requests that were merged but are
not listed.  Then select the applied tag for the release in the "Choose
a tag" dropdown list. Go to the bottom of the list and select the "Set
as pre-release" checkbox.  The "Set as the latest release" button is
reserved for stable releases and updates to them.

If everything is in order, you can click on the "Publish release"
button.  Otherwise, click on "Save draft" and finish pending tasks until
you can return to edit the release page and publish it.

### Prepare pre-compiled packages, update packages to GitHub

Build a fully static LAMMPS installation using a musl-libc
cross-compiler, install into a lammps-static folder, and create a
tarball called lammps-linux-x86_64-19Nov2024.tar.gz (or using a
corresponding date with a future release) from the lammps-static folder.
A suitable build environment is provided with the
https://download.lammps.org/static/fedora37_musl.sif container image.

``` sh
rm -rf release-packages
mkdir release-packages
cd release-packages
wget https://download.lammps.org/static/fedora41_musl.sif
apptainer shell fedora41_musl.sif
git clone -b release --depth 10 https://github.com/lammps/lammps.git lammps-release
cmake -S lammps-release/cmake -B build-release -G Ninja -D CMAKE_INSTALL_PREFIX=$PWD/lammps-static -D CMAKE_TOOLCHAIN_FILE=/usr/musl/share/cmake/linux-musl.cmake -C lammps-release/cmake/presets/most.cmake -C lammps-release/cmake/presets/kokkos-openmp.cmake -D DOWNLOAD_POTENTIALS=OFF -D BUILD_MPI=OFF -D BUILD_TESTING=OFF -D CMAKE_BUILD_TYPE=Release -D PKG_ATC=ON -D PKG_AWPMD=ON -D PKG_MANIFOLD=ON -D PKG_MESONT=ON -D PKG_MGPT=ON -D PKG_ML-PACE=ON -D PKG_ML-RANN=ON -D PKG_MOLFILE=ON -D PKG_PTM=ON -D PKG_QTB=ON -D PKG_SMTBQ=ON
cmake --build build-release --target all
cmake --build build-release --target install
/usr/musl/bin/x86_64-linux-musl-strip lammps-static/bin/*
tar -czvvf lammps-linux-x86_64-19Nov2024.tar.gz lammps-static
exit # fedora 41 container
```

The resulting tar archive can be uploaded to the GitHub release page with:

```
gh release upload patch_19Nov2024 lammps-linux-x86_64-19Nov2024.tar.gz
```

Make sure you have the `flatpak` and `flatpak-builder` packages
installed locally (they cannot be used from the container) and build a
LAMMPS and LAMMPS-GUI flatpak bundle in the `release-packages` folder
with:

``` sh
flatpak --user remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak-builder  --force-clean --verbose --repo=$PWD/flatpak-repo --install-deps-from=flathub --state-dir=$PWD --user --ccache --default-branch=release flatpak-build lammps-release/tools/lammps-gui/org.lammps.lammps-gui.yml
flatpak build-bundle --runtime-repo=https://flathub.org/repo/flathub.flatpakrepo --verbose $PWD/flatpak-repo LAMMPS-Linux-x86_64-GUI-19Nov2024.flatpak org.lammps.lammps-gui release
```

The resulting flatpak bundle file can be uploaded to the GitHub release page with:

```
gh release upload patch_19Nov2024 LAMMPS-Linux-x86_64-GUI-19Nov2024.flatpak
```

Also build serial executable packages that also include LAMMPS-GUI for
Linux, macOS, and Windows, and upload them to the GitHub release.

Clean up:

``` sh
cd ..
rm -r release-packages
```

TODO:
- add detailed commands for building GUI packages on Ubuntu 20.04LTS (move to 22.04LTS?),
  macOS, and Windows cross-compiler and upload to GitHub
- build all Windows cross-compiled installer packages using lammps-packages repo

### Update download website

Check out the LAMMPS website repo
https://github.com/lammps/lammps-website.git and edit the file
`src/download.txt` for the new release.  Test translation with `make
html` and review `html/download.html` Then add and commit to git and
push the changes to GitHub.  The Temple Jenkis cluster will
automatically update https://www.lammps.org/download.html accordingly.

Notify Steve of the release so he can update `src/bug.txt` on the
website from the available release notes.

## LAMMPS Stable Release

A LAMMPS stable release is prepared about once per year in the months
July, August, or September.  One (or two, if needed) feature releases
before the stable release shall contain only bug fixes or minor feature
updates in optional packages.  Also substantial changes to the core of
the code shall be applied rather toward the beginning of a development
cycle between two stable releases than toward the end.  The intention is
to stablilize significant change to the core and have outside users and
developers try them out during the development cycle; the sooner the
changes are included, the better chances for spotting peripheral bugs
and issues.

### Prerequesites

Before making a stable release all remaining backported bugfixes shall
be released as a (final) stable update release (see below).

A LAMMPS stable release process starts like a feature release (see
above), only that this feature release is called a "Stable Release
Candidate" and no assets are uploaded to GitHub.

### Synchronize 'maintenance' branch with 'release'

The state of the 'release' branch is then transferred to the
'maintenance' branch (which will have diverged significantly from
'release' due to the selectively backported bug fixes).

### Fast-forward merge of 'maintenance' into 'stable' and apply tag

At this point it should be possible to do a fast-forward merge of
'maintenance' to 'stable' and then apply the stable\_DMmmYYYY tag.

### Push branches and tags



## LAMMPS Stable Update Release
