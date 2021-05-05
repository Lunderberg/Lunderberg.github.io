---
layout: post
title: Hard Drive Upgrade
tags: [linux]
---

I've been meaning to update my computer's hard drive for a while.
Dual-booting with only 500 GB total doesn't leave much space to spare,
both between a few new video games and the ever-present Windows
Updates.  Though my daily driver is Linux Mint, the flexibility of
dual booting has been useful in the past.  Since I'd rather keep the
dual boot, that means updating to new hardware.

Below is the current output of `fdisk -l` for the old hard drive at
`/dev/nvme1n1`.  My goal is to transfer the Linux partition to the new
2 TB hard drive located at `/dev/nvme0n1`, while maintaining the
Windows partition where it is.  The new hard-drive would have a
standalone boot partition with grub, which could boot either into the
Linux partition on the new hard drive, or the windows partition on the
old.

```
Disk /dev/nvme1n1: 476.96 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: INTEL SSDPEKKW512G7
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf6b35edd

Device         Boot     Start        End   Sectors   Size Id Type
/dev/nvme1n1p1           2048  409601657 409599610 195.3G 83 Linux
/dev/nvme1n1p2 *    409602048  410726399   1124352   549M  7 HPFS/NTFS/exFAT
/dev/nvme1n1p3      410726400  933261311 522534912 249.2G  7 HPFS/NTFS/exFAT
/dev/nvme1n1p4      933263358 1000214527  66951170  31.9G  5 Extended
/dev/nvme1n1p5      933263360 1000214527  66951168  31.9G 82 Linux swap / Solaris
```

As a first try, just copying the full disk contents from the old hard
drive to the new one.  This was done while on a live USB device, so
both drives are otherwise inactive.

```
dd if=/dev/nvme1n1 if=/dev/nvme0n1 status=progress
fdisk /dev/nvme0n1
```

While staring at the partition table in `fdisk`, I realized that I
would need some annoying shuffling.  Deleting the newly copied windows
partition `/dev/nvme0n1p3` would leave a gap in the occupied sectors,
and I'd need to do some copying around, including moving the boot
partition to the start of the disk.  This is mostly for cleanliness,
but would also leave a good amount of unallocated space at the back of
the drive that can be used for testing out additional Linux
distributions.

Instead, I decided to make the partition table in the desired order,
then copy data over on a partition-by-partition basis.  First, I set
up the partition table as follows using `fdisk`.

```
Disk /dev/nvme0n1: 1.88 TiB, 2048408248320 bytes, 4000797360 sectors
Disk model: INTEL SSDPEKNW020T8
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf6b35edd

Device         Boot    Start        End    Sectors  Size Id Type
/dev/nvme0n1p1 *        2048    1128447    1126400  550M  7 HPFS/NTFS/exFAT
/dev/nvme0n1p2       1128448   68237311   67108864   32G 82 Linux swap / Solaris
/dev/nvme0n1p3      68237312 2215720959 2147483648    1T 83 Linux
```

Then, I copied the data over from each of the desired partitions on
the old hard drive, resizing the `ext4` partition after the copy.

```
dd if=/dev/nvme1n1p2 if=/dev/nvme0n1p1 status=progress
dd if=/dev/nvme1n1p5 if=/dev/nvme0n1p2 status=progress
dd if=/dev/nvme1n1p1 if=/dev/nvme0n1p3 status=progress
e2fsck -f /dev/nvme0n1p3
resize2fs /dev/nvme0n1p3
```

With all the data transferred over, it was time to set up grub.
Following [the guide
here](https://howtoubuntu.org/how-to-repair-restore-reinstall-grub-2-with-a-ubuntu-live-cd),
it was relatively straightforward to do so.

```bash
# Ascend the mount-ain
mount -t ext4 /dev/nvme0n1p3 /mnt
mkdir -p /mnt/dev/pts /mnt/proc /mnt/sys
mount --bind /dev /mnt/dev
mount --bind /dev/pts /mnt/dev/pts
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys

# Obtain the holy grub
chroot /mnt
grub-install /dev/nvme0n1
grub-install --recheck /dev/nvme0n1
update-grub
exit

# Descend the mount-ain
umount /mnt/sys
umount /mnt/proc
umount /mnt/dev/pts
umount /mnt/dev
umount /mnt
```

This worked, but only sort of.  And not well.  And not with
predictable behavior.  I could boot into either hard drive, and see
the grub menu, but selecting the Linux Mint installation would swap
onto the other hard drive instead.  Despite showing the correct drive
location, it would load the OS from the other.  The Windows
partition, despite being untouched, refused to boot at all, asking for
a recovery disk.

I tried a few changes, including noticing that `/dev/nvme1n1p2` wasn't
a standalone grub partition on `exFAT`, but was instead a Windows
partition on `NTFS`.  There were a few misadventures under the
mistaken conclusion that maybe the BIOS needed to find a specific
filesystem.

Eventually, I stumbled on the issue.  Looking at the output of
`blkid`, it became pretty clear that something was wrong.

```
/dev/nvme0n1p1: LABEL="System Reserved" UUID="DED6D7FDD6D7D3BD" TYPE="ntfs" PARTUUID="f6b35edd-01"
/dev/nvme0n1p2: UUID="5636a194-5330-4d70-8741-58d661f76fc0" TYPE="swap" PARTUUID="f6b35edd-02"
/dev/nvme0n1p3: UUID="649d2986-3387-4ae7-a1aa-1e26e40613dd" TYPE="ext4" PARTUUID="f6b35edd-03"
/dev/nvme1n1p1: UUID="649d2986-3387-4ae7-a1aa-1e26e40613dd" TYPE="ext4" PARTUUID="f6b35edd-01"
/dev/nvme1n1p2: LABEL="System Reserved" UUID="DED6D7FDD6D7D3BD" TYPE="ntfs" PARTUUID="f6b35edd-02"
/dev/nvme1n1p3: UUID="58B2E3AFB2E38FB4" TYPE="ntfs" PARTUUID="f6b35edd-03"
/dev/nvme1n1p5: UUID="5636a194-5330-4d70-8741-58d661f76fc0" TYPE="swap" PARTUUID="f6b35edd-05"
```

Any time you see something that is supposed to be universally unique
and it isn't, that's a bad sign.  As it turns out, grub decides which
partition to boot based on the UUID, so it was choosing the wrong
partition entirely.  The same was likely true for the Windows
startup.

It was also about this time that I realized that I am, in fact, human,
and therefore needed to get some sleep.  The next day I was booted
into the old hard drive, so the contents needed to be copied over in
order to be up to date.  That also gave a good chance to do everything
one last time, and to do it right.

First up, `fdisk` to set up the final partitions.  I also used `fdisk`
to update the disk identifier (`x` for the expert menu, then `i`), so
those no longer conflicted.

```
Disk model: INTEL SSDPEKNW020T8
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xdfc2cbc1

Device         Boot    Start        End    Sectors  Size Id Type
/dev/nvme0n1p1 *        2048    1128447    1126400  550M 83 Linux
/dev/nvme0n1p2       1128448   68237311   67108864   32G 82 Linux swap / Solaris
/dev/nvme0n1p3      68237312 2215720959 2147483648    1T 83 Linux
```

Now, time to copy data over, but avoid duplicate UUIDs.

```bash
# Boot partition, new filesystem
mkfs.ext4 /dev/nvme0n1p1
tune2fs -U random /dev/nvme0n1p1

# Empty swap partition
mkswap /dev/nvme0n1p2

# Copy over the data, then resize and make new UUID
dd if=/dev/nvme1n1p1 if=/dev/nvme0n1p3 status=progress
e2fsck -f /dev/nvme0n1p3
resize2fs /dev/nvme0n1p3
tune2fs -U random /dev/nvme0n1p3
```

And with that, grub.  Setting it up followed the same steps as
previously, except that I remembered that I wanted to generate `/boot`
in `/dev/nvme0n1p1`, not `p3`.  And that required some testing to
figure out which folders need to be set up for the `chroot`.  After
some experimentation, it needed `/dev`, `/dev/pts`, `/proc`, `/sys`,
`/bin`, `/usr`, `/lib`, `/lib64`, `/etc`, `/var`, and `/sbin`.  And
for some reason, `os-prober` couldn't find the installation on
`/dev/nvme0n1p3` unless the drive was mounted.

But when I tried it out, selecting the partition identified by
`update-grub` as the new hard drive's Linux partition instead loaded
the old hard drive.  Time to throw caution to the wind, ignore the big
"DO NOT EDIT" warnings at the top of `grub.cfg`, and see what can be
done with trusty, trusty emacs.

* Some of the menu entries for `/dev/nvme0n1p3` had the UUID for
  `/dev/nvme1n1p1` instead.  Let's just find/replace those to the
  correct ones.
  
* There was a menu entry for `/dev/nvme0n1p1`, even though that was
  intended to only include grub, and not to be a full OS in its own
  right.  Deleted.
  
* The Windows partitions weren't found at all, and no menu entries
  were there.  But, the `grub.cfg` from the old hard drive had those
  entries, so those can be copied over.
  
And after all that, the last thing was to set the boot order in the
BIOS.  This was made somewhat tricky because only one of the two M.2
hard drives was shown in the boot priority, even though either could
be selected as a boot menu.  Turns out, the priority set whether the
hard drives as an entire category would be ahead or behind the
SATA/USB drives, and there was a completely separate menu to choose
which of the hard drives would take priority over the other.

At the end of the day, I don't know whether this method was more or
less convenient than just installing a fresh OS on the new hard drive.
Certainly, having all installed packages and program settings is nice,
but most of those are backed up through `git` anyways.  But I
definitely learned quite a bit as I bumbled through, and everything
was backed up ahead of time, so the worst case scenario would have
been needing to install a fresh OS anyways.

Finally, if you're looking at this hoping for a guide that will give
step by step instructions, you're looking in the wrong place.  The
number of detours, wrong paths, and misunderstandings hopefully will
convince you that I have no idea what I'm doing in this respect and
should not be a source of any straightforward answers.  But hey, it's
fun.
