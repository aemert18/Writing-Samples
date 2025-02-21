# Working with Links in Linux

## Introduction

Understanding how soft and hard links work within Linux is an important skill for a system administrator. A review of the `ln` man page might be in order before embarking on this hands-on lab to know how to make a symbolic and hard link for files.

## Solution

Log in to the lab server using the credentials provided:

```
ssh cloud_user@<PUBLIC_IP_ADDRESS>
```

### Create a Symbolic (Soft) Link

 1. Create a symbolic (or soft) link from the `/etc/redhat-release` file to a new link file named `release` in the `cloud_user`'s home directory:

    ```
    ln -s /etc/redhat-release release
    ```   

 1. Verify the link is valid :

    ```
    ls -l
    ```

 1. See if you can read the file's contents:

    ```
    cat release
    ```

 1. See if you can read the link's contents:

    ```
    cat /etc/redhat-release
    ```

    They should be the same.

#### Check the Inode Numbers for the Link

 1. Look at the inode number of `/home/cloud_user/release`:

    ```
    ls -i release
    ```

 1. Check the inode number for `/etc/redhat-release`:

    ```
    ls -i /etc/redhat-release
    ```

    They should be different, as the symbolic link is just a new filesystem entry that references the original file.

### Create a Hard Link

 1. Create a directory called `docs` :

    ```
    mkdir docs
    ```

 1. Copy `/etc/services` into the `docs` directory:

    ```
    cp /etc/services docs/
    ```

 1. Create a hard link from the `/home/cloud_user/docs/services` file to a new link location named `/home/cloud_user/services`:

    ```
    ln docs/services services
    ```

 1. Verify the link's inode number as well as the inode number for the original `/etc/services`:

    ```
    ls -l
    ```

    This should verify for us that this is a hard link, not a soft link. It won't have an arrow pointing to the actual file it's linked to, like a soft link does. Just to verify, check these two with `cat` and make sure they're the same:

 1. View the contents of the inodes:

    ```
    ls -i services
    ```

    ```
    ls -i docs/services
    ```

    You should see they the same inode number, meaning they're essentially the same file.

### Attempt to Create a Hard Link Across Filesystems

 1. View the individual block devices:

    ```
    lsblk
    ```

    You should see something similar to the following:

    ```
    NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    xvda    202:0    0  10G  0 disk
    ├─xvda1 202:1    0   1M  0 part
    └─xvda2 202:2    0  10G  0 part /
    xvdb    202:16   0   2G  0 disk
    └─xvdb1 202:17   0   2G  0 part /opt
    ```

    You can see here `/` and `/opt` are on 2 separate partitions. Because each partition has its own set of inodes, hard links across partitions don't work. Soft links should, though.

 1. Try to make a hard link from `/home/cloud_user/docs/services` to `/opt/services`:

    ```
    ln /home/cloud_user/docs/services /opt/services
    ```

    You should get a `failed to create hard link` error .

### Attempt to Create a Soft Link Across Filesystems

 1. Try to make the same sort of cross-partition link, using the `-s` flag to make it a soft link:

    ```
    ln -s /etc/redhat-release /opt/release
    ```

    There shouldn't be any output, meaning it was successful.

 1. View the contents of the inodes again:

 1. View the inode numbers of `/etc/redhat-release` and `/opt/release`:

    ```
    ls -i /etc/redhat-release
    ```

    ```
    ls -i /opt/release
    ```

    You should see they have different inodes, but the linking will work.

## Conclusion

Congratulations on successfully completing this hands-on lab!