# GNOME, with fractional scaling

GNOME Shell packages for Ubuntu with patches for fractional scaling applied:

* [GNOME/mutter!3](https://gitlab.gnome.org/GNOME/mutter/merge_requests/3)
* [GNOME/gnome-shell!5](https://gitlab.gnome.org/GNOME/gnome-shell/merge_requests/5)

---

## Using these packages

Packages are published to a [Launchpad PPA](https://launchpad.net/~lukecarrier/+archive/ubuntu/gnome-fractional-scaling) and are currently only available for the 19.04 Disco Dingo series.

Add the PPA:

```
$ sudo apt-add-repository ppa:lukecarrier/gnome-fractional-scaling
```

Update the packages:

```
$ sudo apt upgrade
```

Enable the framebuffer scaling feature flag in Mutter:

```
$ gsettings set org.gnome.mutter experimental-features ['scale-monitor-framebuffer']
```

Now open the _Settings_ app, navigate to _Devices_ > _Screen Display_ and select the desired display. You should see 25% increments available in the _Scale_ option on supported displays.

## Known issues

* All non-GTK3 (Firefox, anything Electron based) applications will appear blurry due to the scaling implementation.
* Some theme decorations go a bit wonky (e.g. drop shadows) in some Shell themes (e.g. Arc).

## Building

Per the [Getting Set Up](https://packaging.ubuntu.com/html/getting-set-up.html) instructions:

```
$ sudo apt install gnupg pbuilder ubuntu-dev-tools apt-file
```

Enable the source repositories:

```
$ sudo sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list
$ sudo apt update
```

Install build dependencies:

```
$ sudo apt build-dep mutter gnome-shell
```

If this errors about `libmutter-2-dev` not being installable, work around APT by manually installing `libmutter-3-dev` and all of the dependencies listed by `apt-rdepends`:

```
$ sudo apt install apt-rdepends
$ apt-rdepends --build-depends gnome-shell --follow=DEPENDS \
          | grep -v libmutter-2-dev | awk '{print $2}' \
          | xargs sudo apt install -y
```

Set up a build environment:

```
$ pbuilder-dist disco create
```

Get the source for the packages:

```
$ cd work/mutter
$ pull-lp-source mutter disco
$ cd ../gnome-shell
$ pull-lp-source gnome-shell disco
```

Fetch the patches from the GNOME GitLab instance:

```
$ curl -o patches/mutter/resource-scale.patch https://gitlab.gnome.org/GNOME/mutter/merge_requests/3.patch
$ curl -o patches/gnome-shell/resource-scale.patch https://gitlab.gnome.org/GNOME/gnome-shell/merge_requests/5.patch
```

Apply the patches:

```
$ cd mutter-<version>/
$ edit-patch resource-scale
$> patch -p1 <<path>/patches/mutter/resource-scale.patch
```

Resolve merge conflicts, then repeat for GNOME Shell.

## Preparing to publish

Generate a PGP key:

```
$ gpg --full-generate-key
```

Then publish it to the Ubuntu keyserver:

```
$ gpg --send-keys --keyserver keyserver.ubuntu.com <keyid>
```

## Publishing

Build the source package:

```
$ debuild -S -d -us -uc
```

Use `pbuilder` to check that the build completes successfully:

```
$ pbuilder-dist disco build ../gnome-shell_3.30.1-2ubuntu4.dsc
```

Ensure we correctly sign the source package and associated files:

```
$ debsign *.changes -k<keyid>
```

Then push the source package to a Launchpad PPA. You should get an email confirming the job gets accepted -- rejects are usually down to signing issues or pushing a binary package:

```
$ dput ppa:<user>/<name> *.changes
```
