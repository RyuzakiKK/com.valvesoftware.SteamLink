Steam Link - Flatpak packaging
==============================

The Steam Link app is not open-source and does not accept external
contributions. This Flatpak packaging downloads precompiled binaries
from Valve and adds them to the Flatpak app's directory hierarchy as-is.

Please note that Valve has not given permission to redistribute the
Steam Link app binaries, other than via the official Flatpak package
on Flathub.

Layout
------

`com.valvesoftware.SteamLink.yml` is the top-level manifest file
(install Flatpak and look at flatpak-manifest(5)).

At the moment, it builds:

* a specially patched version of `qtbase` with more controller support
    * `patches/steamlink/qt-everywhere-src-5.14.1.patch` is from the
        Steam Link SDK and is not used directly
    * `patches/steamlink/qtbase.patch` is a version of that patch that
        has been put through `filterdiff` (see the comment at the top)
        to only include the `qtbase` parts
    * `patches/org.kde.Sdk/` are Flatpak integration improvements taken from
        https://gitlab.com/freedesktop-sdk/flatpak-kde-runtime/-/tree/qt5.14/
    * TODO: Can we update this version of Qt to something more modern?
    * TODO: Has some/all of the controller support gone upstream?
* `qtsvg`
    * this is straight from org.kde.Sdk, and not patched

and also pulls together:

* precompiled dependencies
    * suitable snapshots of SDL (these are patched, upstream SDL is not enough)
    * a suitable snapshot of FFmpeg
* the Steam Link main executable
* metadata:
    * icons
    * [AppStream](https://www.freedesktop.org/software/appstream/docs/) metadata
    * a [Desktop Entry](https://specifications.freedesktop.org/desktop-entry-spec/latest/)
        to launch it with

Test-building this package locally
----------------------------------

### Install dependencies on build machine

* `apt install flatpak-builder` or your distro's equivalent
* `flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
    * Add `--user` if you prefer to install into
        `~/.local/share/flatpak` instead of `/var/lib/flatpak`
* `flatpak install org.freedesktop.Platform/x86_64/21.08`
* `flatpak install org.freedesktop.Sdk/x86_64/21.08`

### Build

Run `flatpak-builder(1)` according to its documentation. For convenience,
`./build.sh` has a suitable command which will do the build in `./_build`.
flatpak-builder will maintain a cache in `./.flatpak-builder` so that it
doesn't have to rebuild Qt every time.

To deploy the resulting app on a test machine, the easiest way is to
tell Flatpak to build a standalone "bundle" (a `.flatpak` file).
Again, `./build.sh` has a suitable command.

### Install dependencies on test machine

* `apt install flatpak` or your distro's equivalent
* Log out and back in, if you didn't already have Flatpak installed
    (this makes sure your desktop environment will load the files
    "exported" by Flatpak apps, like their `.desktop` files)
* `flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
    * Add `--user` if you prefer to install into
        `~/.local/share/flatpak` instead of `/var/lib/flatpak`
* `flatpak install org.freedesktop.Platform/x86_64/21.08`

### Install on test machine

* Copy `_build/steamlink.flatpak` to the test machine
* `flatpak install /path/to/steamlink.flatpak`
    * Add `--user` if you prefer to install into
        `~/.local/share/flatpak` instead of `/var/lib/flatpak`
