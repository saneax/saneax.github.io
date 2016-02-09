---
layout: "post"
title: "poor mans lvm guide"
---

####My Guide To LVM
-----------------
This is the Logical Volume manager, by default there on Redhat. A nifty tool from redhat, which scores over every other storage system. just because you will run out of space some day. Firstly..

1. Use fdisk to create the partitions. Change the Type to hex 8e (0x8e), on the fdisk (prompt) you need to say '8e' only, when changing type. Remember your partitions ie. /dev/hda1 /dev/hda2 etc..

2. use the pvcreate command to initialize the physical volume.
  - `#>pvcreate /dev/hda1`
similarly more if you have more partitions.

3. Create a Volume Group, if you want a knew mount point with the new space. or you can also extend an existing Volume Group. First how we create a new VolumeGroup..
  - `#>vgcreate MyVolGroup01 /dev/hda1 /dev/hda2 ...`
  The command takes basically two parts, the first one has to be the new name of the Volume Group and the second has to be the Physical Volume which we have already created in step 1.
  Now, How to extend to an existing VolumeGroup
  - `#>vgextend VolGroup00 /dev/hdb1 /dev/hdb2 /dev/hdc1`
  which means the existing Volume Group called VolGroup00 will now encompass the Physical Volumes hdb1, hdb2 and hdc1.

4. Create the Logical Volume (this step is the final step, which creates the Logical Volume)
We can also extend a Logical Volume, we will be covering that next.
First before we create a Logical Volume, we should check the Total PE (Physical Extent available in the Volume Group)
  - `#>vgdisplay MyVolGroup01`
Note the PE amounts mentioned in 'Total PE' and 'Free PE'.
Now we can create the Logical Volume..
There are two ways.. we can also spell the Max Space in GB or we can also spell the Max PE's wanted.
    A. Max PE.
    - `#>lvcreate -l 340 MyVolGroup01 -n usrlv`
this will create the device /dev/MyVolGroup01/usrlv block which will have the MAX 340 PE's (eaCH PE is nearly 4KB).
    B. Max Space in GB
    - `#>lvcreate -L 1.3GB MyVolGroup01 -n usrlv`
this will create the device /dev/MyVolGroup01/usrlv block which will have the MAX 1.3 GB of Space. 
Note: Allocating More space than possible will naturally shoot an error. so you can check what is feasible. Refer to vgdisplay command to know what space is available within a Volume Group.
Now How can we Extend an existing Logical Volume.. ? remember, You can only extend a Logical Volume, if its Volume Group has Free PE's left. ie, say usrlv is a Logical Volume for MyVolGroup01, as laid in step4 and we need to resize the Logical volume for it. We can only do that provided there is 'free PE's left in the Volume Group 'MyVolGroup01'. IF there is no space left, you can create the Physical Volumes as said in step 1 and extend it to the Volume Group, and then do the extend.. as mentioned hereafter.
    - `#>lvextend -l 979 /dev/MyVolGroup01/usrlv (will extend the usrlv to 979 PE's which are free )`
or
    - `#>lvextend -L 2.5GN /dev/MyVolGroup01/usrlv`

5. resize the restructuring of the Logical Volume, if you have done an extending. Or create the file system on the Logical Volume if its a newly created LV.
  - `#>mkfs.ext3 /dev/MyVolGroup01/usrlv`
or resize, while the FS is mounted via 'ext2online'
  - `#>ext2online /dev/MyVolGroup01/usrlv`
or unmount the LV. and then resize it via
  - `#>resize2fs /dev/MyVolGroup01/usrlv`


Hope this helps.. also I will check back when I need to..

