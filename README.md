# labwc with DirectionalCycleWindows
**Warning, this package is entirely vibe-coded**

An Arch Linux package for labwc with directional window selection inspired by
Openbox. This repository contains a build recipe, a patch and example bindings.

The package is named `labwc-directional`, provides `labwc`, and conflicts with
the repository package. It installs the usual `/usr/bin/labwc` executable,
manuals and login-session entry. Pacman tracks the installation, and normal
repository updates do not replace this differently named package.

## Pinned source

- Upstream: https://github.com/labwc/labwc
- Base commit: `fa6794a3d29593faf088447c30c1e9dd86cb94ed`
- Base version: labwc 0.20.2 plus 30 upstream commits
- Feature commit: `0aac8e8cfe33da75e24d15d5d8ee740f796f6153`
- Required wlroots ABI: 0.20

The feature commit identifies the patch exported from the development checkout;
it does not need to exist in the upstream repository. The build downloads only
the pinned upstream base and applies the patch stored here.

## Build

Build as your regular user on an up-to-date Arch Linux system with `base-devel`
installed. Review `PKGBUILD` before running it.

```sh
makepkg -s
```

This downloads source, installs missing build dependencies with your approval,
applies the patch, compiles labwc, runs tests and creates the package archive.
It does not install labwc. The tests use an isolated headless compositor and
virtual input; they do not interact with your running desktop. The geometry
tests run without cmocka. If cmocka is already installed, the upstream unit
tests run as well.

## Install

Once the build and tests pass:

```sh
sudo pacman -U ./labwc-directional-0.20.2.r30.gfa6794a3-1-x86_64.pkg.tar.zst
```

Pacman asks to remove the conflicting `labwc` package in the same transaction.
There is no need to uninstall it beforehand. Your user configuration is kept.
Save your work, log out and select the normal labwc session when logging back in.
Installing a package does not replace an already-running compositor.

Alternatively, `makepkg -si` builds, tests and then offers to install the package.

## Configure

Merge the bindings in `directional-keybinds.xml` or similar into the
`<keyboard>` section of `~/.config/labwc/rc.xml`. The example replaces the
default Super-arrow snapping shortcuts. Reload configuration with `labwc --reconfigure`.

Hold Super and press an arrow once per step. Releasing all modifiers commits
the selection; Escape cancels. Candidates include minimized windows on the
current workspace, across all monitors. Selection uses content centers and
the nearest coordinate along the requested axis. Ties use perpendicular
distance, then recent focus. There is no wrapping or automatic key repeat.

By default, selection runs `Focus`, `Raise`, and `Unshade`. A binding can replace
those defaults with a `<finalactions>` list. An empty list runs nothing. The
binding which starts selection supplies the final actions for the whole cycle.

The outline uses these settings in the theme's `themerc` or your
`~/.config/labwc/themerc-override`:

```ini
osd.window-switcher.preview.border.width: 2
osd.window-switcher.preview.border.color: #ffffff,#000000,#ffffff
```

These settings also affect ordinary window-switcher outlines. Directional
selection enforces a minimum border width of 2.

## Updates

Continue full system updates with `sudo pacman -Syu`. A separate package name
prevents the repository labwc package from silently replacing this build, but
library ABI changes can still require rebuilding it. The recipe retains Arch's
wlroots soname dependency so pacman can detect an incompatible wlroots change.
Other library changes may require a rebuild too.

For a dependency rebuild against the same source, increment `pkgrel`, update
`.SRCINFO`, then build and test again. Keep the previous package archive until
the new version has been tested in a real session.

For an upstream labwc update:

1. In the development checkout, rebase the feature commit onto the chosen new
   upstream revision, resolve conflicts, and test the resulting compositor.
2. Export the feature commit from that checkout:

   ```sh
   git format-patch -1 --stdout --no-signature > /path/to/this/repo/directional-cycle.patch
   ```

3. Update `_upstream_commit` to the feature commit's upstream parent, set an
   appropriate `pkgver`, and reset `pkgrel` to 1. Update the pinned-source
   details and installation filename in this README too. If the patch spans
   multiple commits, export the entire feature series rather than only `-1`.
4. Update the checksums and package metadata:

   ```sh
   updpkgsums
   makepkg --printsrcinfo > .SRCINFO
   ```

   `updpkgsums` is provided by `pacman-contrib`.

5. Build with `makepkg --cleanbuild -s`, test the new binary in a nested desktop,
   then install the new package. If an upstream ABI change is involved, update
   the recipe's dependencies to match the new Arch recipe before building.

Keep the source revision and patch pinned. This recipe intentionally does not
follow upstream `main` automatically.

## Return to the repository version

```sh
sudo pacman -S labwc
```

Accept replacement of `labwc-directional`, then log out and back in. To restore
an exact previous custom build, use `pacman -U` with its saved package archive,
provided its library dependencies are still installed.

## License

The labwc patch follows upstream's GPL-2.0-only license. The packaging files
and documentation in this repository are also GPL-2.0-only. The recipe is based
on Arch Linux's labwc packaging:
https://gitlab.archlinux.org/archlinux/packaging/packages/labwc
