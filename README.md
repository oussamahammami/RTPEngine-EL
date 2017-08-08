# RTPengine for Enterprise Linux

> Documentation powred by [Rebtel](https://www.rebtel.com/en/), ENJOY :stuck_out_tongue_closed_eyes: !

## Installing from RPMs

There are three RPMs:

- *rtpengine*: the userspace daemon
- *rtpengine-iptables*: the iptables plugin
- *rtpengine-kernel*: the kernel module source

All of the RPMs have correctly set dependencies and if you just want the userspace daemon you can install it with yum (assuming you have access to a CentOS repository).

The *rtpengine-iptables* package is dependent on the *rtpengine*, and *rtpengine-kernel* packages. The *rtpengine-kernel* package has a dependency (make, gcc) so the kernel will compiled during the rpm installation .

Note: installing *rtpengine-kernel* builds a kernel module which requires the sources for the running kernel. The *kernel-devel* and *kernel-headers* packages are meta-packages that install the headers and source for the latest kernel version. This will be what what you want unless you are running a custom or older kernel. *ngcp-rtpengine-dkms* does not have *kernel-devel* and *kernel-headers* as dependencies as this could cause problems if you are using a custom or older kernel, so you need to install these manually.


## RPM Compliation

To build the RPMs you need all of the packages listed in the Manual Compilation section (except for *kernel-devel* and *kernel-headers*):

```
yum install epel-release 
yum groupinstall 'Development Tools'
yum install redhat-rpm-config rpm-build xmlrpc-c-devel zlib-devel hiredis-devel \
 glib2-devel libcurl-devel openssl-devel pcre-devel gcc make pkgconfig redhat-rpm-config nc \
 iptables-devel libcurl-devel glib2 glib2-devel pcre-devel xmlrpc-c-devel libevent-devel \
 openssl-devel libpcap libpcap-devel 
```

Prepare the `rpmbuild` directory:

```
# mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
# echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros
# tree rpmbuild/
rpmbuild/
├── BUILD
├── RPMS
│   ├── noarch
│   └── x86_64
├── SOURCES
│   └── rtpengine-5.4.1.tar.gz
├── SPECS
│   └── rtpengine.spec
└── SRPMS

7 directories, 2 files
```


To build the RPMs:
- Goto the `~/rpmbuild/SOURCES` directory
- Create a tar archive.  For example, from within the cloned directory you can
  use
  
  ```
  git archive --output ~/rpmbuild/SOURCES/ngcp-rtpengine-<version number>.tar.gz --prefix=ngcp-rtpengine-<version number>/ master
  ```
  
  where `<version number>` is the version number of the master branch
- Decompress the tar source file and change the directory name to `rtpengine-<version number>`
- Add files below in `el` directory:
    * `rtpengine-conf` default conf file.
    * `rtpengine-logrotation` log rotation conf.
    * `rtpengine-rsyslog` rsyslog conf.
    * `rtpengine.service` systemd service config.
    * `rtpengine-start` start/stop script required to start the service.
    * `rtpengine-stop-post` start/stop script required to start the service.
- Compress the file with `tar -czvf rtpengine-<version number>.tar.gz rtpengine-<version number>/` 
- Build the RPMs. For example,
   `rpmbuild -ta ~/rpmbuild/SOURCES/ngcp-rtpengine-<version number>.tar.gz`

Once the build has completed the binary RPMs will be in `~/rpmbuild/RPMS`.


## Packages

There are three parts to rtpengine, each of which can be found in the respective subdirectories.

* `daemon`

	The userspace daemon and workhorse, minimum requirement for anything to work. Running `RTPENGINE_VERSION="\"<version number>\"" make` will compile the binary, which will be called `rtpengine`.

* `iptables-extension`

	Required for in-kernel packet forwarding. Running `RTPENGINE_VERSION="\"<version number>\"" make` will compile the plugin for `iptables` and `ip6tables`. The file will be called `libxt_RTPENGINE.so` and should be copied into the directory `/lib/xtables/` in 32-bit environments and `/lib64/xtables/` in 64-bit environments.

* `kernel-module`

	Required for in-kernel packet forwarding. Compilation of the kernel module requires the kernel development packages for the kernel version you are using (see output of `uname -r`) to be installed. Running `RTPENGINE_VERSION="\"<version number>\"" make` will compile the kernel module.

	Successful compilation of the module will produce the file `xt_RTPENGINE.ko`. The module can be inserted into the running kernel manually through `insmod xt_RTPENGINE.ko` (which will result in an error if depending modules aren't loaded, for example the `x_tables` module), but it's recommended to copy the module into `/lib/modules/<version number>/updates/`, followed by running `depmod -a`. After this, the module can be loaded by issuing `modprobe xt_RTPENGINE`.

	Note: the *kernel-devel* and *kernel-headers* packages are meta-packages that install the headers and source for the latest kernel version. This will be what you want unless you are running a custom or older kernel.
	

## Reference

* [Rtpengine for Enterprise Linux](https://github.com/sipwise/rtpengine/blob/master/el/README.el.md)
* [Running the RTPEngine Under Systemd Control](https://github.com/Binan/rtpengine-systemd)
* [RTPEngine Manual Compilation and Installation In Fedora RedHat ](https://voipmagazine.wordpress.com/2015/02/17/rtpengine-compilation-and-installation-in-fedora-redhat/)
* [RTPEngine RPM creation from the Tar file or source code.](https://sillycodes.com/rtpengine-rpm-creation-from-tar-file-or/)
