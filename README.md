# Docker Issues and Tips (aufs/overlay/btrfs..)

Picked up and categorized subjectively from https://github.com/docker/docker/issues. Comments and pull requests are welcome.

:white_large_square: = Open (maybe not up-to-date, please check the link by yourself!) 

:white_square_button: = Mostly resolved (ditto, plus subjective)

:white_check_mark: = Resolved

## Storage Drivers
### AUFS
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#783](https://github.com/docker/docker/issues/783)|Cannot access to a directory due to a permission error|:neutral_face: Medium|:smiley: Easy|Expected AUFS behavior. `dirperm1` mount option fixes this issue. |[Update the kernel (AUFS >= 2008xxxx?) and Docker daemon (>= 1.7)](https://github.com/docker/docker/issues/783#issuecomment-146455363)|Confirm: `docker info | grep "Dirperm1 Supported: true"`|
|:white_check_mark: [#18180](https://github.com/docker/docker/issues/18180)|A process becomes a zombie and hangs up |:scream: High|:scream: Hard(multiprocessor)<br> :smiley: Easy(uniprocessor)|Compatibility between the kernel and AUFS|[Update the kernel (AUFS >= 20160111)](https://github.com/docker/docker/issues/18180#issuecomment-187583209)|Java apps and MongoDB are known to be affected|
|:white_check_mark: [#20199](https://github.com/docker/docker/issues/20199)|`fcntl(F_SETFL, O_APPEND)` is ignored and hence data can be corrupted|:scream: High|:smiley: Easy|AUFS bug|[Update the kernel (AUFS >= 20160301)](https://github.com/docker/docker/issues/20199#issuecomment-191020308)|Dovecot is known to be affected|
|:white_check_mark: [#20240](https://github.com/docker/docker/issues/20240)|Weird permission even though `dirperm1` is enabled|:neutral_face: Medium|:scream: Hard|AUFS bug|[Update the kernel (AUFS >= 20160905)](https://github.com/docker/docker/issues/20240#issuecomment-244639386)||
|:white_large_square: [AUFS ML 2016-03-08](https://sourceforge.net/p/aufs/mailman/message/34917261/)|Hang up related to `O_DIRECT`|:scream: High|:smiley: Easy|Unanalyzed|None|Percona is known to be affected|
|:white_large_square: [#24309](https://github.com/docker/docker/issues/24309)|Unable to remove files previously committed |:scream: High|:smiley: Easy|Unanalyzed|[This article seems related, but perhaps slightly different](https://github.com/yokogawa-k/aufs-directories-undeletable-bug)(Japanese)|
|:white_square_button: [#34361](https://github.com/docker/docker/issues/34361)|AUFS + XFS hangs up |:scream: High|:smiley: Easy|AUFS bug|[Update AUFS](https://github.com/moby/moby/issues/34361#issuecomment-320861506)|
|:white_check_mark: [#22207](https://github.com/moby/moby/issues/22207)|Orphaned diffs are eating my free space |:neutral_face: Medium|:scream: Hard|AUFS bug|[Update to 17.06.2-ce](https://github.com/moby/moby/issues/22207#issuecomment-327457187)||

Non-bug issues:
* AUFS is not available in the mainline kernel．Only a few distros (Ubuntu, Boot2Docker, ..) support AUFS, but even for Ubuntu, Canonical says ["AUFS will disappear"](https://lists.ubuntu.com/archives/ubuntu-devel/2012-March/034880.html).
* No support for extended attributes ("xattrs"), and [might not ever get support](http://lkml.iu.edu/hypermail/linux/kernel/0902.3/01324.html) ([#1070](https://github.com/docker/docker/issues/1070), [#8460](https://github.com/docker/docker/issues/8460)).
* `rename(2)` is not fully supported ( see also [#aufs--overlay-common](#aufs--overlay-common) )

### Overlay
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#10180](https://github.com/docker/docker/issues/10180)|RPMDB corruption|:scream: High|:neutral_face: Medium|[Expected overlay behavior](https://bugzilla.redhat.com/show_bug.cgi?id=1213602#c0)|Use yum-{utils,plugins-ovl}-1.1.31-33.el7 (included in RHEL 7.2) or later. [Kernel patch](https://github.com/portworx/overlayfs) is also available.|[Linux 4.6](https://github.com/torvalds/linux/commit/fb5bb2c3b73df060d588b6521de5ab03589283f7) or later prints human-friendly dmesg|
|:white_check_mark: [#12080](https://github.com/docker/docker/issues/12080)|Cannot use UNIX domain sockets|:neutral_face: Medium|:smiley: Easy|Overlay Bug|Use [Linux 4.7-rc4](https://github.com/torvalds/linux/commit/30402c8949934fbaca07d9c20074d0d7a5a8385f) or later||
|:white_check_mark: [#12327](https://github.com/docker/docker/issues/12327)|pip fails|:scream: High|:smiley: Easy|Overlay Bug|Use [Linux 4.5](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=45d11738969633ec07ca35d75d486bf2d8918df6) or later||
|:white_check_mark: [#19082](https://github.com/docker/docker/issues/19082)|Weird behavior after removing the current directory|:smiley: Low|:smiley: Easy|Overlay Bug|Use [Linux 4.5](https://github.com/torvalds/linux/commit/ce9113bbcbf45a57c082d6603b9a9f342be3ef74) or later||
|:white_square_button: [#19647](https://github.com/docker/docker/issues/19647), [coreos/bugs#1095](https://github.com/coreos/bugs/issues/1095)|Untar fails intermittently|:scream: High|:scream: Hard|Overlay Bug|[Use Linux 4.13 with OVERLAY_FS_INDEX=y](https://github.com/coreos/bugs/issues/1095#issuecomment-318773389)|Analysis is in progress in [coreos/bugs#1095](https://github.com/coreos/bugs/issues/1095#issuecomment-233793361)|
|:white_large_square: [#20640](https://github.com/docker/docker/issues/20640)|Container cannot be started|:neutral_face: Medium|:scream: Hard|Unanalyzed|None|Possibly identical to [#16902](https://github.com/docker/docker/issues/16902)|
|:white_check_mark: [#20950](https://github.com/docker/docker/issues/20950)|/dev/console: operation not permitted|:scream: High|:smiley: Easy|[Kernel Bug](https://github.com/docker/docker/issues/20950#issuecomment-201335412)|Use recent Linux kernels||
|:white_check_mark: [#21555](https://github.com/docker/docker/issues/21555)|`docker build` fails intermittently (overlay1)|:scream: High|:scream: Hard|DiffDriver bug|Use [Docker 1.13](https://github.com/docker/docker/pull/27209) or later|Overlay2 doesn't have this issue by design
|:white_check_mark: [#24913](https://github.com/docker/docker/issues/24913)|permissions broken after chown|:neutral_face: Medium|:smiley: Easy|Overlay Bug|Use [Linux 4.6](https://github.com/docker/docker/issues/24913#issuecomment-240666349) or later|The overlay2 issue [#28391](https://github.com/docker/docker/issues/28391) is due to the identical bug|
|:white_check_mark: [#25244](https://github.com/docker/docker/issues/25244)|opaque flag not reset after directory copy up|:neutral_face: Medium|:smiley: Easy|Overlay Bug|Resolved in [Linux 4.8](https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/log/fs/overlayfs/copy_up.c?id=refs/tags/v4.8-rc5&showmsg=1) and backported to [4.4.21](https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/log/fs/overlayfs/copy_up.c?id=refs/tags/v4.4.21&showmsg=1) and [4.7.4](https://git.kernel.org/cgit/linux/kernel/git/stable/linux-stable.git/log/fs/overlayfs/copy_up.c?id=refs/tags/v4.7.4&showmsg=1)|npm is known to be affected|
|:white_check_mark: [machine#3327](https://github.com/docker/machine/issues/3327)|chmod fails with EPERM|:smiley: Low|:smiley: Easy|Overlay Bug|Use [Linux 4.5](https://github.com/torvalds/linux/commit/b81de061fa59f17d2730aabb1b84419ef3913810) or later||
|:white_check_mark:[#27358](https://github.com/docker/docker/issues/27358)|file removal weird on overlay + XFS (ftype=0)|:scream: High|:smiley: Easy|[Expected behavior](https://github.com/torvalds/linux/commit/e7c0b5991dd1be7b6f6dc2b54a15a0f47b64b007)|[Format xfs with ftype=1](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.2_Release_Notes/technology-preview-file_systems.html)||
|:white_check_mark:[#34320](https://github.com/moby/moby/pull/34320)|`docker build` produces weird images with `CONFIG_OVERLAY_FS_REDIRECT_DIR=y`|:scream: High|:smiley: Easy|DiffDriver issue|Apply [#34342](https://github.com/moby/moby/pull/34342) (Docker 17.08?)||

Non-bug issues:
* :scream: High inode usage (resolved in overlay2, which will be available in [Docker 1.12](https://github.com/docker/docker/pull/22126/files))
* Red Hat says  ["OverlayFS remains a Technology Preview in Red Hat Enterprise Linux 7.3 under most circumstances"](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.3_Release_Notes/technology_previews_file_systems.html)
* `rename(2)` is not fully supported ( see also [#aufs--overlay-common](#aufs--overlay-common) )
* MySQL doesn't work without `touch`-ing files under `/var/lib/mysql`: https://github.com/docker/for-linux/issues/72#issuecomment-319904698

### AUFS / Overlay common

Non-bug issue: `rename(2)` is not fully supported [#25409](https://github.com/docker/docker/issues/25409)

reports about the incompatible behavior of `rename(2)` from the real world

|Software|Report|
|---|---|
|Apache Kudu|https://issues.apache.org/jira/browse/KUDU-1419|
|CernVM-FS|https://sft.its.cern.ch/jira/browse/CVM-651|
|GPG|https://github.com/docker/docker/issues/26317|
|NPM|https://github.com/npm/npm/issues/9863|
|Samba|https://bugzilla.samba.org/show_bug.cgi?id=9966|



### BtrFS
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#19073](https://github.com/docker/docker/issues/19073)|`sendfile(2)` can be unkillable|:smiley: Low|:smiley: Easy|BtrFS bug|None|Not likely to happen in production, but needs consideration for public PaaS|
|:white_large_square: [#20080](https://github.com/docker/docker/issues/20080)|cgroups kmem limit leads crash and data corruption|:scream: High|:smiley: Easy?|Btrfs bug|Avoid kmem limit configuration?||

Non-bug issues:

 * Slow [#10161](https://github.com/docker/docker/issues/10161)
 * No page sharing (e.g. same DLLs are loaded redundantly) http://comments.gmane.org/gmane.comp.sysutils.docker.devel/1384
 * Docker says BtrFS is [Experimental](https://docs.docker.com/engine/userguide/storagedriver/btrfs-driver/). Red Hat says BtrFS is [Tech Preview](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/7.2_Release_Notes/technology-preview-file_systems.html).

### ZFS

|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#20153](https://github.com/docker/docker/issues/20153)|Some operations fail due to `EBUSY`|:neutral_face: Medium|:neutral_face: Medium|Daemon bug|[Update Docker daemon](https://github.com/docker/docker/commit/803e3d4d1e7b9f029bbe31a80197403bf4d27252)||

Non-bug issues:

 * Docker says ZFS is [not recommended for production](https://docs.docker.com/engine/userguide/storagedriver/zfs-driver/).

### DeviceMapper

|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#4036](https://github.com/docker/docker/issues/4036)|Mount fails|:scream: High|:smiley: Easy|udev sync disabled|Use a Docker daemon binary which supports udev sync|Confirm: `docker info | grep "Udev Sync Supported: true"`|
|:white_large_square: [#20401](https://github.com/docker/docker/issues/20401)|Infinite “mount/remount” loop, which makes the system unresponsive|:scream: High|:scream: High|Unanalyzed (perhaps related to XFS)|None||

Non-bug issues:

 * Slow [#10161](https://github.com/docker/docker/issues/10161)
 * No page sharing (e.g. same DLLs are loaded redundantly) http://comments.gmane.org/gmane.comp.sysutils.docker.devel/1384

### Storage driver test tool

 * [dmcgowan/dsdbench](https://github.com/dmcgowan/dsdbench): Docker Storage Driver Benchmarks and Tests

### So which storage driver should I use?

It totally depends on your workload, but Docker, Inc. says AUFS and Devicemapper (direct-lvm) are "production-ready".

https://github.com/docker/docker/blob/master/docs/userguide/storagedriver/selectadriver.md#future-proofing

![driver-pros-and-cons.png](https://raw.githubusercontent.com/docker/docker/89923c1f32aeff5bf11fbb04723dd0e154559545/docs/userguide/storagedriver/images/driver-pros-cons.png)

Although not listed in the above table, VFS driver is also attractive for its robustness.

Links:

 * https://jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html#1
 * http://www.projectatomic.io/docs/filesystems/
 * https://blog.jessfraz.com/post/the-brutally-honest-guide-to-docker-graphdrivers/


#### Anyway...
You know, containers should be "immutable" and "disposable".

For persistent data and some special temporary data, you should better consider using an external volume (`docker run -v`).

Links:

* https://github.com/emccode/rexray


## Network
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_square_button: [#5618](https://github.com/docker/docker/issues/5618)|hang up with `unregister_netdevice: waiting for lo to become free`|:scream: High|:scream: Hard|Kernel bug|[Use Linux 4.8 or later](https://github.com/torvalds/linux/commit/751eb6b6042a596b0080967c1a529a9fe98dac1d)|[The patch will be backported to old kernels in major distros](https://github.com/docker/docker/issues/5618#issuecomment-248317966)|
|:white_check_mark: [#18776](https://github.com/docker/docker/issues/18776)|TCP checksums are ignored|:scream: High|:scream: Hard|Kernel bug|[Use Linux 4.4 or later](https://github.com/torvalds/linux/commit/ce8c839b74e3017996fad4e1b7ba2e2625ede82f)|[blog](https://medium.com/vijay-pandurangan/linux-kernel-bug-delivers-corrupt-tcp-ip-data-to-mesos-kubernetes-docker-containers-4986f88f7a19)|

## Logging
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#19209](https://github.com/docker/docker/issues/19209)|GELF driver saturates CPU|:scream: High|:smiley: Easy|[Compression](https://github.com/docker/docker/issues/19209#issuecomment-226551113)|Disable compression||
|:white_check_mark: [#18057](https://github.com/docker/docker/issues/18057),[#20600](https://github.com/docker/docker/issues/20600)|`cat /dev/zero` leads to out of memory|:scream: High|:smiley: Easy|logger's stdio handling issue|[Use Docker 1.13 or later](https://github.com/docker/docker/issues/18057#issuecomment-239643581) (or just disable the logging)|Related:  [#21181](https://github.com/docker/docker/issues/21181)|
|:white_large_square: [#22497](https://github.com/docker/docker/issues/22497)|container cannot be stopped if many logs are being printed |:scream: High|:scream: Hard|logger's stdio handling issue|||
|:white_check_mark: [#22502](https://github.com/docker/docker/issues/22502)|logging blocks the container|:scream: High|:smiley: Easy|logger's stdio handling issue|[Use Docker 1.11 or later](https://github.com/docker/docker/issues/22502#issuecomment-217993807)|affected versions:  1.10.0|

## Others
|Issue|Abstract|Impact|Reproducibility|Cause|Solution|Notes|
|---|---|---|---|---|---|---|
|:white_check_mark: [#17720](https://github.com/docker/docker/issues/17720)|Docker daemon 1.9 serious performance issue|:scream: High|:scream: Hard|?|Use Docker 1.10||
|:white_large_square: [#19758](https://github.com/docker/docker/issues/19758)|soft lockup related to `show_mountinfo()`, after frequent `docker run`|:scream: High|:scream: Hard|Unanalyzed (Kernel bug related to the number of processors?)|None||
|:white_check_mark: [#20670](https://github.com/docker/docker/issues/20670)|/dev/pts unmounted on the HOST when you are using `-v /dev:/dev` (After that you can no longer open SSH nor xterm)|:scream: High|:smiley: Easy|daemon bug related to mount namespace|Use Docker 1.11.1. (Or Spawn the docker daemon from systemd. Or do not use `-v /dev:/dev`)||
|:white_check_mark: [#20836](https://github.com/docker/docker/issues/20836)|Daemon hangs up after frequent `docker run`|:scream: High|:scream: Hard|Daemon bug|Use Docker 1.11.1||
|:white_check_mark: [#28936](https://github.com/docker/docker/issues/28936)|Strange permission issues with named containers on 1.12.3 |:scream: High|:smiley: Easy|Daemon bug related to SELinux)|Use [Docker 1.12.4](https://github.com/docker/docker/pull/23024)||
|:white_check_mark: [Ubuntu linux-azure #1719045](https://bugs.launchpad.net/ubuntu/+source/linux-azure/+bug/1719045)|`fatal error: unaligned sysUnused` on Azure |:scream: High|?|Ubuntu linux-azure kernel bug|Use [linux-azure 4.11.0-1013.13 or later](https://bugs.launchpad.net/ubuntu/+source/linux-azure/+bug/1719045)||

Non-bug issues:
 * `docker ps` is sometimes slow due to lock: [#19328](https://github.com/docker/docker/issues/19328) (Mitigated in Docker 17.07, [#31273](https://github.com/moby/moby/pull/31273)
 * `EBUSY` on `docker rm` in Linux < 3.19: [#26510](https://github.com/docker/docker/issues/26510)
