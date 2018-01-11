# emby-unlocked
Emby with the premium Emby Premiere features unlocked.

## Releases
Releases including the patch are available below:

- [Arch Linux](https://aur.archlinux.org/packages/emby-server-unlocked/)
- [Docker](https://hub.docker.com/r/nvllsvm/emby-unlocked/)

## Why?
At some point, the developers of Emby added a nasty nag screen before playback.
Worse, you need to wait a few seconds before being able to dismiss it.
They refuse to address removing it as seen [here](https://github.com/MediaBrowser/Emby/issues/2469).

This is **bullshit** for a GPLv2 licensed product. Of course someone is going to fork Emby to remove it.

So while my main motivation was to **remove** the nag screen, unlocking all the premium features proved the simplest way.

## Help! Premiere feature x does not work.
While this patch makes your local server believe Emby Premiere features are unlocked, some features may still not function.
For example, both tv.emby.media and the mobile apps rely upon validation with the Emby-owned mb3admin.com server.

## Modifications

[PluginSecurityManager.cs.patch](https://github.com/nvllsvm/emby-unlocked/blob/master/patches/PluginSecurityManager.cs.patch)

Before compilation, simply patch the existing file:
```
patch -N -p1 -r - Emby.Server.Implementations/Security/PluginSecurityManager.cs < ../PluginSecurityManager.cs.patch
```

[connectionmanager.js](https://github.com/nvllsvm/emby-unlocked/blob/master/replacements/connectionmanager.js)

Not really sure what this unlocks outside of removing the nag on the **Sync** screen.
Sync doesn't seem to work afterwards. 
Regardless - your own Emby server has zero need to contact the Emby-owned validation URL: [https://mb3admin.com/admin/service/registration/validateDevice](https://mb3admin.com/admin/service/registration/validateDevice).

The included version of this in the source distribution is minified. Thus, making a patch is difficult.
The only difference boils down to replacing ``self.getRegistrationInfo`` with this:

```
self.getRegistrationInfo = function(feature, apiClient) {
    var cacheKey = "regInfo-" + apiClient.serverInfo().Id;
    appStorage.setItem(cacheKey, JSON.stringify({
        lastValidDate: new Date().getTime(),
        deviceId: self.deviceId()
    }));
    return Promise.resolve();
}
```

## Debian package

This repository also includes a working `debian` folder useful for building custom Debian packages.

1. Copy the `debian` folder in its entirety to the main `Emby` source directory.
2. Modify `debian/changelog` if the version has changed and to set your information.
3. Build the package in the root of the source. The following command will work well on a quad-core processor (adjust the `-j` option to suit your system):
```
dpkg-buildpackage -us -uc -j4
```
4. Install the resulting package:
```
sudo dpkg -i emby-server_3.2.60.12-ul1_all.deb
```

This `debian/` folder was originally sourced from the 3.2.60.0 source package with the addition of the `emby-unlocked` patches. To reproduce it:

1. Follow the repo instalation directions at the [official build repository](https://software.opensuse.org/download.html?project=home%3Aemby&package=emby-server).
2. Modify the created list file (assuming Debian Stretch) at `/etc/apt/sources.list.d/emby-server.list` and add a `deb-src` line:
```
deb http://download.opensuse.org/repositories/home:/emby/Debian_9.0/ /
deb-src http://download.opensuse.org/repositories/home:/emby/Debian_9.0/ /
```
3. Update your repositories and download the source package:
```
sudo apt-get update
sudo apt-get source emby-server
```
