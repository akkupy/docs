### ※ Raspberry Pi RAID NAS Server Setup ※ ###

__Hardware:__
- Raspberry Pi 4 (4GB RAM)
- Micro SD card 64GB
- 2x Integral USB 3.1 flash drives 128GB

__OS:__
- Raspbian  Bullseye

--------------------

#### FORMAT DRIVES

1. Insert drive and list existing partition tables:

   ```sh
   $ sudo fdisk -l
   ```

2. Unmount drive (if needed):

   ```sh
   $ sudo unmount /media/pi/<HARD-DRIVE-LABEL>
   ```

3. Partitioning:

   ```sh
   $ sudo fdisk /dev/sda
   ```  
   [m] for help  
   [o]  
   [n]  
   [p]  
   [1]  
   [Enter]  
   [Enter]  
   [w]  

4. Formatting:

   ```sh
   $ sudo mkfs.ext4 /dev/sda1
   ```  
   [y]  
   
   *** _DO NOT MOUNT THE DRIVE_ ***

5. Repeat steps 1-4 above for other drives.

--------------------

#### CREATE RAID ARRAY

1. Update system and install 'mdadm' RAID package:  

   ```sh
   $ sudo apt-get update
   $ sudo apt-get upgrade
   $ sudo apt-get install mdadm
   ```

2. Find out the mount points for each drive:

   ```sh
   $ blkid
   ```  
   -or-  
   ```sh
   $ lsblk
   ```

3. Create RAID volume/array:

   __Linear Mode__  
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=linear --raid-devices=2 /dev/sda1 /dev/sdb1
   ```

   __RAID-0 (Stripe Mode)__  
   ```sh 
   $ sudo mdadm --create --verbose /dev/md0 --level=stripe --raid-devices=2 /dev/sda1 /dev/sdb1
   ```  
   -or-  
   ```sh
   $ sudo mdadm -Cv /dev/md0 -l0 -n2 /dev/sd[ab]1
   ```

   __RAID-1 (Mirror Mode)__  
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=mirror --raid-devices=2 /dev/sda1 /dev/sdb1
   ```  
   -or-  
   ```sh
   $ sudo mdadm -Cv /dev/md0 -l1 -n2 /dev/sd[ab]1
   ```

   __RAID-4/5/6__  
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=4 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1
   ```   
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1 --spare-devices=1 /dev/sde1
   ```  
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=6 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
   ```

   __RAID-10__  
   ```sh
   $ sudo mdadm --create --verbose /dev/md0 --level=10 --raid-devices=4 /dev/sda1 /dev/sdb1 /dev/sdc1 /dev/sdd1
   ```

4. Confirm RAID array:

   ```sh
   $ cat /proc/mdstat
   ```

5. Save RAID array:

   ```sh
   $ sudo -i  
   $ mdadm --detail --scan >> /etc/mdadm/mdadm.conf  
   $ less /etc/mdadm/mdadm.conf  
   $ exit
   ```

6. Create file system:

   ```sh
   $ sudo mkfs.ext4 -v -m .1 -b 4096 -E stride=32,stripe-width=64 /dev/md0
   ```

   NOTE: To change advanced parameters after creation if needed:
   ```sh
   $ tune2fs -E stride=n,stripe-width=m /dev/md0
   ```

7. Create mount point and mount file system:

   ```sh
   $ sudo mkdir /mnt/raidx  
   $ sudo mount /dev/md0 /mnt/raidx
   ```

   NOTE: To change the owner of the mount point to pi:  
   ```sh
   $ sudo chown pi:pi /mnt/raidx
   ```

8. Check content of the mounted file system:

   ```sh
   $ ls -la /mnt/raidx
   ```

9. Confirm its capacity:

   ```sh
   $ df -h -x devtmpfs -x tmpfs
   ```

10. Update the initial file system (Raspberry Pi uses a RAM disk image when booting up and we want to include our array):
 
    ```sh   
    $ sudo update-initramfs -u
    ```

11. Check UUID of mounted file system:
 
    ```sh   
    $ blkid
    ```  
    -or-  
    ```sh
    $ ls -l /dev/disk/by-uuid
    ```

12. Add to fstab (make the drive permanent and auto mount drive at boot):

    ```sh
    $ sudo nano /etc/fstab
    ```  

    Enter new line before the bottom comments and add:  
   
    ```
    UUID=(my_uuid) /mnt/raidx ext4 defaults,noatime 0 0  
    ```

    Save and exit:  
    [Ctrl+O]  
    [Ctrl+X]

13. Retrieve drive parameters and test speeds (optional):

    ```sh
    $ sudo hdparm -I /dev/md0
    $ sudo hdparm -tT --direct /dev/md0
    ```
   
14. Reboot:
 
    ```sh
    $ sudo reboot
    ```

15. Useful commands/Manage Mode:

    `$ cat /proc/mdstat` : show status of all RAID devices  
    `$ mdadm --detail /dev/md0` : detailed information about RAID md0 (`mdadm -D`)  
    `$ mdadm --detail --brief /dev/md0` : for shorthened/brief details (`mdadm -Db`)  
    `$ mdadm --query /dev/md0` : quick human-readable summary of RAID md0 (`mdadm -Q`)  
    `$ mdadm --examine /dev/sdx`	: information about RAID component device sdx (`mdadm -E`)  
    `$ mdadm --stop /dev/md0`	: stop RAID device md0  
    `$ mdadm --assemble --scan`	: restart/assemble RAID device  

--------------------

#### SET UP NAS

1. Install SAMBA:

   ```sh
   $ sudo apt-get install samba samba-common-bin
   ```

2. Set up SAMBA password (for user pi):

   ```sh
   $ sudo smbpasswd -a pi
   ```

3. Edit SAMBA config file:

   ```sh
   $ sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak  
   $ sudo nano /etc/samba/smb.conf
   ```  

   Scroll all the way down to the bottom of the file and add the following code to create NAS file share:  
   
   ```
   # NAS Share Block  
   [NAS]  
   path = /mnt/raidx  
   comment = RPI4 RAID0 NAS Server  
   volume = NAS-Server  
   valid users = pi  
   read only = NO  
   guest ok = NO  
   public = NO  
   writable = YES  
   browsable = YES  
   ### -rwxr--r--  
   create mask = 0744  
   ### -rwxr-xr-x  
   directory mask = 0755  
   ### All hosts on the 192.168.142 subnet allowed:  
   hosts allow = 192.168.142.  
   ```

   Save and exit:  
   [Ctrl+O]  
   [Ctrl+X]  

4. Check configuration file for internal correctness:

   ```sh
   $ testparm
   ```

5. Restart SAMBA service:

   ```sh
   $ sudo /etc/init.d/samba restart
   ```  
   -or-  
   ```sh
   $ sudo service smbd restart
   ```

6. Reboot ?

   ```sh
   $ sudo reboot
   ```

--------------------

#### USAGE ON MAC

1. Open Finder

2. Menu "Go" ⟶ "Connect to Server..."  

   Address: [smb://rpi4/NAS]

3. [Connect]

4. Connect As:
   - [x] Registered User  

   Name: pi  
   Password: ●●●●●●●●

5. [Connect]

--------------------

#### APPENDIX  

##### Commands to check

`$ rsync options source destination` : remote/local file-copying tool  
`$ rsync -ahv /mnt/u1/ /mnt/u2/` : example of the above

##### RAID Levels

Number of Drives /  
  RAID Level Availability:

2 /  
  RAID-0 Stripe (Fastest, but no redundancy)  
  RAID-1 Mirror (Excellent redundancy, good speed)

3 /  
  RAID-0 Stripe (Fastest, but no redundancy)  
  RAID-4 Dedicated parity disk (Good speed & redundancy)  
  RAID-5 Block-level striping with distributed parity (Excellent speed & redundancy)  

4 /  
  RAID-6 Block-level striping with two parity blocks distributed across all member disks (Excellent speed & redundancy)  
  RAID 10 (nested RAID 1+0) (Excellent speed and redundancy)  

##### References

⇒ [mdadm(8) - Linux man page](https://linux.die.net/man/8/mdadm)  
⇒ [How To Manage RAID Arrays with mdadm](https://www.digitalocean.com/community/tutorials/how-to-manage-raid-arrays-with-mdadm-on-ubuntu-16-04)  
⇒ [Advantages and disadvantages of various RAID levels](https://datapacket.com/blog/advantages-disadvantages-various-raid-levels/)  
⇒ [RAID setup](https://raid.wiki.kernel.org/index.php/RAID_setup)  
⇒ [A guide to mdadm](https://raid.wiki.kernel.org/index.php/A_guide_to_mdadm)  
⇒ [Characteristics of Linux RAID levels](https://linux.die.net/EVMSUG/characraidlvls.html)  
⇒ [Build your own Raspberry Pi NAS](https://pimylifeup.com/raspberry-pi-nas-3/)  
⇒ [How to Setup a Raspberry Pi Samba Server](https://pimylifeup.com/raspberry-pi-samba/)  
⇒ [Build a Raspberry Pi RAID NAS Server – Complete DIY Guide](https://pchelp.ricmedia.com/build-raspberry-pi3-raid-nas-server/)  
⇒ [Partitioning, Formatting, and Mounting a Hard Drive in Linux](https://medium.com/@sh.tsang/partitioning-formatting-and-mounting-a-hard-drive-in-linux-ubuntu-18-04-324b7634d1e0)  