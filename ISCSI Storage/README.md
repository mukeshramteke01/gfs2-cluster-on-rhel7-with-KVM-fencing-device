### ISCSI Storage configuration and installation for cluster

#### Install the package

    [root@iscsi ~]# yum install targetcli

#### ADD and create partition /dev/sdb

    [root@iscsi ~]# fdisk -l

    Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x00006070

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     1050623      524288   83  Linux
    /dev/sda2         1050624    20971519     9960448   8e  Linux LVM

    Disk /dev/mapper/rhel-root: 7105 MB, 7105150976 bytes, 13877248 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 65536 bytes / 65536 bytes


    Disk /dev/mapper/rhel-swap: 1044 MB, 1044381696 bytes, 2039808 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 65536 bytes / 65536 bytes

#### Create Physical volume

    [root@iscsi ~]# pvcreate /dev/sdb
      Physical volume "/dev/sdb" successfully created.

#### Create Volume group 

    [root@iscsi ~]# vgcreate vg_iscsi /dev/sdb
      Volume group "vg_iscsi" successfully created

#### Verify  Volume group

    [root@iscsi ~]# vgs
      VG       #PV #LV #SN Attr   VSize   VFree  
      rhel       1   3   0 wz--n-  <9.50g  <1.90g
      vg_iscsi   1   0   0 wz--n- <20.00g <20.00g

#### Create Logical volume

    [root@iscsi ~]# lvcreate -L +5G -n lv_iscsi_1 vg_iscsi
      Logical volume "lv_iscsi_1" created.

#### Verify Logical volume

    [root@iscsi ~]# lvs
      LV         VG       Attr       LSize   Pool   Origin Data%  Meta%  Move Log Cpy%Sync Convert
      pool00     rhel     twi-aotz--  <7.59g               16.38  17.09                           
      root       rhel     Vwi-aotz--  <6.62g pool00        18.79                                  
      swap       rhel     Vwi-aotz-- 996.00m pool00        0.01                                   
      lv_iscsi_1 vg_iscsi -wi-a-----   5.00g  

#### Enter the admin console

    [root@iscsi ~]# targetcli 
    Warning: Could not load preferences file /root/.targetcli/prefs.bin.
    targetcli shell version 2.1.fb46
    Copyright 2011-2013 by Datera, Inc and others.
    For help on commands, type 'help'.

    /> cd backstores/block 

#### Created block storage object iscsi_server using /dev/vg_iscsi/lv_iscsi_1.

    /backstores/block> create iscsi_server /dev/vg_iscsi/lv_iscsi_1

    /backstores/block> cd /iscsi 

#### Create a target

    /iscsi> create iqn.2017-18.iscsi.server:tar1
    Created target iqn.2017-18.iscsi.server:tar1.
    Created TPG 1.
    Global pref auto_add_default_portal=true
    Created default portal listening on all IPs (0.0.0.0), port 3260.

#### Set ACL

    /iscsi> cd iqn.2017-18.iscsi.server:tar1/tpg1/luns 

    /iscsi/iqn.20...ar1/tpg1/luns> cd ../acls 
 
    /iscsi/iqn.20...ar1/tpg1/acls> create iqn.2017-18.iscsi.server:iscsi.linjb.com
    Created Node ACL for iqn.2017-18.iscsi.server:iscsi.linjb.com

#### Set LUN

    /iscsi/iqn.20...ar1/tpg1/acls> cd iqn.2017-18.iscsi.server:iscsi.linjb.com 
    /iscsi/iqn.20...csi.linjb.com>
    
    /iscsi/iqn.20...ar1/tpg1/luns> ls
    o- luns .................................................................................................................. [LUNs: 0]
    
    /iscsi/iqn.20...ar1/tpg1/luns> create /backstores/block/iscsi_server 
    Created LUN 0.
    Created LUN 0->0 mapping in node ACL iqn.2017-18.iscsi.server:iscsi.linjb.com

    /iscsi/iqn.20...ar1/tpg1/luns> ls
    o- luns .................................................................................................................. [LUNs: 1]
      o- lun0 ....................................................... [block/iscsi_server (/dev/vg_iscsi/lv_iscsi_1) (default_tg_pt_gp)]

    /iscsi/iqn.20...ar1/tpg1/luns> exit
    Global pref auto_save_on_exit=true
    Last 10 configs saved in /etc/target/backup.
    Configuration saved to /etc/target/saveconfig.json

#### Enable and Restart the target service

    [root@iscsi ~]# systemctl enable target.service
    Created symlink from /etc/systemd/system/multi-user.target.wants/target.service to /usr/lib/systemd/system/target.service.

    [root@iscsi ~]# systemctl restart target.service

#### Allow iSCSI Target service

    [root@iscsi ~]# firewall-cmd --permanent --add-port=3260/tcp
    success

    [root@iscsi ~]# firewall-cmd --reload
    success
 


