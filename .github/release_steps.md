# LAMMPS Release Steps

The following notes chronicle the current steps for preparing and publishing LAMMPS releases.  For
definitions of LAMMPS versions and releases mean, please refer to [the corresponding section in the
LAMMPS manual](https://docs.lammps.org/Manual_version.html).

## LAMMPS Feature Release

A LAMMPS feature release is currently prepared after about 500 to 750 commits to the 'develop'
branch or after a period of four weeks up to two months.  This is not a fixed rule, though, since
external circumstances can cause delays in preparing a release, or pull requests that are desired to
be merged for the release are not yet completed.

### Preparing a 'next\_release' branch

Create a 'next\_release' branch off 'develop' and make the following changes:

- set the LAMMPS\_VERSION define to the planned release date in src/version.h in the format
  "D Mmm YYYY" or "DD Mmm YYYY"
- remove the LAMMPS\_UPDATE define in src/version.h
- update the release date in doc/lammps.1
- update all TBD arguments for ..versionadded::, ..versionchanged:: ..deprecated:: to the
  planned release date in the format "DMmmYYYY" or "DDMmmYYYY"
- check release notes for merged new features and check if ..versionadded:: or ..versionchanged::
  are missing and need to be added
Submit this pull request, rebase if needed.  This is the last pull request merged for the release
and should not contain any other changes. (Exceptions: this document, last minute trivial(!) changes).

This PR shall not be merged before **all** pending tests have completed and cleared. If needed, a
bugfix pull request should be created and merged to clear all tests.

### Create release on GitHub

When all pending pull requests for the release are merged and have cleared testing, the
'next\_release' branch is merged into 'develop'.

Check out 'develop' locally, pull the latest changes, merge them into 'release', apply a suitable
release tag (for historical reasons the tag starts with "patch_" followed by the date, and finally
push everything back to GitHub. Example:

```
git checkout develop
git pull
git checkout release
git pull
git merge --ff-only develop
git tag -s -m "LAMMPS feature release 19 November 2024" patch_19Nov2024
git push git@github.com:lammps/lammps.git --tags develop release
```

Go to https://github.com/lammps/lammps/releases and create a new (draft) release page or check the
existing draft for any necessary changes from pull requests that were merged but are not listed.
Then select the applied tag for the release in the "Choose a tag" dropdown list. Go to the bottom of
the list and select the "Set as pre-release" checkbox.  The "Set as the latest release" button is
reserved for stable releases and updates to them.

If everything is in order, you can click on the "Publish release" button.  Otherwise, click on "Save
draft" and finish pending tasks until you can return to edit the release page and publish it.

### Update download website, prepare pre-compiled packages, update packages to GitHub

Publishing the release on GitHub will trigger the Temple Jenkins cluster to update
the https://docs.lammps.org/ website with the documentation for the new feature release
and it will create a tarball for download (which contains the translated manual).

Build a fully static LAMMPS installation using a musl-libc cross-compiler, install into a
lammps-static folder, and create a tarball called lammps-linux-x86_64-19Nov2024.tar.gz (or using a
corresponding date with a future release) from the lammps-static folder and upload it to GitHub
with:

```
gh release upload patch_19Nov2024 ~/Downloads/lammps-linux-x86_64-19Nov2024.tar.gz
```


## LAMMPS Stable Release

A LAMMPS stable release is prepared about once per year in the months July, August, or September.
One (or two, if needed) feature releases before the stable release shall contain only bug fixes
or minor feature updates in optional packages.  Also substantial changes to the core of the code
shall be applied rather toward the beginning of a development cycle between two stable releases
than toward the end.  The intention is to stablilize significant change to the core and have
outside users and developers try them out during the development cycle; the sooner the changes
are included, the better chances for spotting peripheral bugs and issues.

### Prerequesites

Before making a stable release all remaining backported bugfixes shall be released as a (final)
stable update release (see below).

A LAMMPS stable release process starts like a feature release (see above), only that this feature
release is called a "Stable Release Candidate" and no assets are uploaded to GitHub.

### Synchronize 'maintenance' branch with 'release'

The state of the 'release' branch is then transferred to the 'maintenance' branch (which will
have diverged significantly from 'release' due to the selectively backported bug fixes).

### Fast-forward merge of 'maintenance' into 'stable' and apply tag

At this point it should be possible to do a fast-forward merge of 'maintenance' to 'stable'
and then apply the stable\_DMmmYYYY tag.

### Push branches and tags



## LAMMPS Stable Update Release
