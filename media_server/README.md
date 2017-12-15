# 96Boards Home Media Server

If you are like me, you have hundreds if not thousands of photos, music, and movies sitting on a hard drive in your house, but no easy way to access them. Wouldn’t it be nice to make all those photos and home movies available to everyone in the house at any time? Or how about playing all your own music over the home network? This is easier than you might think with a 96Boards! Using a 96Boards such as the DragonBoard DB410c and some freely available open source software that supports DLNA, you can create a transparent local Media Server that provides easy simultaneous access to all this information to friends and family. It’s easier than you think and it turns out to be quite fun to have all your digital media available at your fingertips.

# Table of Contents

- [1) Hardware](#1-hardware)
   - [1.1) Hardware requirements](#11-hardware-requirements)
- [2) Software](#2-software)
   - [2.1) Operating System](#21-operating-system)
   - [2.2) Software Dependencies](#22-software-dependencies)
- [3) 96Boards Home Media Server](#3-96boards-home-media-server)
   - [3.1) Hardware setup](#31-hardware-setup)
   - [3.2) Software Setup](#32-software-setup)
   - [3.3) Video Demonstration](#33-video-demonstration)

# 1) Hardware

## 1.1) Hardware requirements

- [96Boards CE running Debian OS](https://www.96boards.org/products/ce/)
- [96Boards Compliant Power Supply](http://www.96boards.org/product/power/)
- External Hard Drive or USB Stick: Depending upon the capacity and storage requirements.
- Ethernet Dongle (Optional): My home WiFi was good enough to stream media to a single device at a time, but if you are planning to stream to multiple clients, its recommended to use an Ethernet dongle

NOTE: Depending upon the make and model of the external hard drive, you may require a powered USB hub as the current limit on the USB ports may prevent the hard drive from spinning up.

# 2) Software

## 2.1) Operating System

- Linaro Debian based OS
  - [ DragonBoard 410c (latest)](https://github.com/96boards/documentation/blob/master/ConsumerEdition/DragonBoard-410c/Downloads/Debian.md)


## 2.2) Software Dependencies

```shell
$ sudo apt update
$ sudo apt upgrade
```

# 3) 96Boards Home Media Server

## 3.1) Hardware Setup

- Make sure the 96Boards CE is powered off
- Connect I/O devices (Monitor, Keyboard, Mouse etc...)
- Connect USB Storage device
- Power on your 96Boards CE with compatible power supply
- Connect to WiFi using GUI or the ```shell nmtui ``` command.

## 3.2) Software Setup

  1. Install the minidlna service:

  ```shell
  $ sudo apt install minidlna
  ```

  2. Create mount-points for external drive:

  ```shell
  $ sudo mkdir /var/minidlna
  $ sudo chown minidlna:minidlna /var/minidlna
  ```

  3. Format USB storage as etx4: Make sure the USB storage medium is plugged in.: **WARNING: THIS WILL ERASE ALL DATA ON THE USB STORAGE**

    - Unmount USB Storage:

      ```shell
      $ sudo umount /dev/sda1
      ```

    - Create new partition table using fdisk:

      ```shell
      linaro@linaro-alip:~$ sudo fdisk /dev/sda

      Welcome to fdisk (util-linux 2.29.2).
      Changes will remain in memory only, until you decide to write them.
      Be careful before using the write command.

      Command (m for help): d
      Selected partition 1
      Partition 1 has been deleted.

      Command (m for help): n
      Partition type
        p   primary (0 primary, 0 extended, 4 free)
        e   extended (container for logical partitions)
      Select (default p):

      Using default response p.
      Partition number (1-4, default 1): #Press Enter to use default value
      First sector (2048-31678463, default 2048): #Press Enter to use default value
      Last sector, +sectors or +size{K,M,G,T,P} (2048-31678463, default 31678463):

      Created a new partition 1 of type 'Linux' and of size 15.1 GiB.
      Partition #1 contains a vfat signature.

      Do you want to remove the signature? [Y]es/[N]o: Y

      The signature will be removed by a write command.

      Command (m for help): t
      Selected partition 1
      Partition type (type L to list all types): 83
      Changed type of partition 'Linux' to 'Linux'.

      Command (m for help): w
      The partition table has been altered.
      Calling ioctl() to re-read partition table.
      Syncing disks.
      ```

    - Format partition as ext4:

      ```shell
      $ sudo mkfs.ext4 /dev/sda1
      mke2fs 1.43.4 (31-Jan-2017)
      Creating filesystem with 3959552 4k blocks and 991232 inodes
      Filesystem UUID: 38fc409e-3912-4042-93c3-72b5f1afa939
      Superblock backups stored on blocks:
	     32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

       Allocating group tables: done                            
       Writing inode tables: done                            
       Creating journal (16384 blocks): done
       Writing superblocks and filesystem accounting information: done
       ```

  4. Create Folders and set permissions:

  ```shell
  $ sudo mkdir -p /media/temp
  $ sudo mount /dev/sda1 /media/temp
  $ sudo chmod 666 /media/temp
  $ mkdir music
  $ mkdir pictures
  $ mkdir videos
  $ sudo mkdir -p /var/minidlna
  $ sudo chown -R minidlna:minidlna /var/minidlna
  ```

  5. Create fstab entry for auto mount:

    - Note UUID using

        ```shell
        $ sudo blkid /dev/sda1
        ```

    - Add to fstab

        ```shell
        $ sudo vi /etc/fstab
        ```

        ```shell
        UUID=bd113554-d64e-407c-ab44-39e7d8c02f28 /var/minidlna ext4 defaults 0 2
        ```
      Feel free to use any other text editor.

  6. Configure the Media (DLNA) Server: Edit the entries to match the text below

      ```shell
      $ sudo vi /etc/minidlna.conf

      # This is the configuration file for the MiniDLNA daemon, a DLNA/UPnP-AV media
      # server.
      #
      # Unless otherwise noted, the commented out options show their default value.
      #
      # On Debian, you can also refer to the minidlna.conf(5) man page for
      # documentation about this file.
      # Specify the user name or uid to run as.
      user=minidlna
      # Path to the directory you want scanned for media files.
      #
      # This option can be specified more than once if you want multiple directories
      # scanned.
      #
      # If you want to restrict a media_dir to a specific content type, you can
      # prepend the directory name with a letter representing the type (A, P or V),
      # followed by a comma, as so:
      #   * "A" for audio    (eg. media_dir=A,/var/lib/minidlna/music)
      #   * "P" for pictures (eg. media_dir=P,/var/lib/minidlna/pictures)
      #   * "V" for video    (eg. media_dir=V,/var/lib/minidlna/videos)
      media_dir=A,/var/minidlna/music
      media_dir=P,/var/minidlna/pictures
      media_dir=V,/var/minidlna/videos
      # Path to the directory that should hold the database and album art cache.
      db_dir=/var/cache/minidlna
      # Path to the directory that should hold the log file.
      log_dir=/var/log
      # Type and minimum level of importance of messages to be logged.
      #
      # The types are "artwork", "database", "general", "http", "inotify",
      # "metadata", "scanner", "ssdp" and "tivo".
      #
      # The levels are "off", "fatal", "error", "warn", "info" or "debug".
      # "off" turns of logging entirely, "fatal" is the highest level of importance
      # and "debug" the lowest.
      #
      # The types are comma-separated, followed by an equal sign ("="), followed by a
      # level that applies to the preceding types. This can be repeated, separating
      # each of these constructs with a comma.
      #
      # The default is to log all types of messages at the "warn" level.
      #log_level=general,artwork,database,inotify,scanner,metadata,http,ssdp,tivo=warn
      # Use a different container as the root of the directory tree presented to
      # clients. The possible values are:
      #   * "." - standard container
      #   * "B" - "Browse Directory"
      #   * "M" - "Music"
      #   * "P" - "Pictures"
      #   * "V" - "Video"
      # If you specify "B" and the client device is audio-only then "Music/Folders"
      # will be used as root.
      #root_container=.
      # Network interface(s) to bind to (e.g. eth0), comma delimited.
      # This option can be specified more than once.
      #network_interface=
      # IPv4 address to listen on (e.g. 192.0.2.1/24).
      # If omitted, the mask defaults to 24. The IPs are added to those determined
      # from the network_interface option above.
      # This option can be specified more than once.
      #listening_ip=
      # Port number for HTTP traffic (descriptions, SOAP, media transfer).
      # This option is mandatory (or it must be specified on the command-line using
      # "-p").
      port=8200
      # URL presented to clients (e.g. http://example.com:80).
      presentation_url=/
      # Name that the DLNA server presents to clients.
      # Defaults to "hostname: username".
      friendly_name=dm home
      # Serial number the server reports to clients.
      # Defaults to 00000000.
      serial=681019810597110
      # Model name the server reports to clients.
      model_name=Windows Media Connect compatible (MiniDLNA)
      # Model number the server reports to clients.
      # Defaults to the version number of minidlna.
      #model_number=
      # Automatic discovery of new files in the media_dir directory.
      inotify=yes
      # List of file names to look for when searching for album art.
      # Names should be delimited with a forward slash ("/").
      # This option can be specified more than once.
      album_art_names=Cover.jpg/cover.jpg/AlbumArtSmall.jpg/albumartsmall.jpg
      album_art_names=AlbumArt.jpg/albumart.jpg/Album.jpg/album.jpg
      album_art_names=Folder.jpg/folder.jpg/Thumb.jpg/thumb.jpg
      # Strictly adhere to DLNA standards.
      # This allows server-side downscaling of very large JPEG images, which may
      # decrease JPEG serving performance on (at least) Sony DLNA products.
      strict_dlna=no
      # Support for streaming .jpg and .mp3 files to a TiVo supporting HMO.
      enable_tivo=yes
      # Notify interval, in seconds.
      notify_interval=180
      # Path to the MiniSSDPd socket, for MiniSSDPd support.
      minissdpdsocket=/run/minissdpd.sock
      .
      .
      .
      ```

  7. Poweroff The Media Server

      ```shell
      $ sudo poweroff
      ```

  8. Add media to the external storage and power back on.

  9. You should now see the media server listed on upnp and mDLNA clients.

## 3.3) Video Demonstration

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/ldwtO2wN0zU/0.jpg)](https://www.youtube.com/watch?v=ldwtO2wN0zU)
