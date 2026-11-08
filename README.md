# How to Compress VM Disk Images

This is README project how to compress (deflate / shrink) virtual machine "thin provisioned" disk image file.

NOTE(s):
- Virtual machine disk image shall not contain snapshots (which prewent image file compression) !!!
- Delelete / combine snapshots to one single image file before compress process.

## VirtualBox VDI Images
- https://superuser.com/questions/529149/how-to-compact-virtualboxs-vdi-file-size
- https://www.softperfect.com/articles/how-to-compact-vdi-file/

To deflate (compact) a VirtualBox VDI image, you must first fill the unused space inside the guest operating system with zeros, shut down the virtual machine, and then run the compaction command on the host.

### 1. Zero out free space inside the guest OS

-   **For Windows Guests:**
    1.  Download **SDelete** from Microsoft Sysinternals.
    2.  Open a Command Prompt as Administrator inside the VM.
    3.  Run the following command (replace `C:` with your target drive letter): 

        *cmd*
        
        ```
        sdelete.exe C: -z
        
        ```

-   **For Linux Guests:**
    -   **Option A (Fill free space with zeros and delete):**
        
        *bash*
        ```
        sudo dd if=/dev/zero of=/var/tmp/bigemptyfile bs=4096k
        sudo rm /var/tmp/bigemptyfile
        ```

    -   **Option B (Using `zerofree` for ext2/ext3/ext4):**
        -   Boot the Linux guest into recovery mode or unmount the partition, then run:
            
            *bash*
            ```
            zerofree -v /dev/sda1
            ```

----------

### 2. Shut down the virtual machine

-   Power off the guest OS completely (do not leave it in a saved/suspended state).

----------

### 3. Compact the VDI file on the host

Open a terminal or command prompt on your **host machine** and run the VirtualBox management command: 
-   **On Windows Host:**  *cmd*
    ```
    "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifymedium --compact "C:\path\to\yourdisk.vdi" 
    ```
-  **On Linux or macOS Host:**   *bash*
    ```
    vboxmanage modifymedium --compact "/path/to/yourdisk.vdi"
    ```

## QNAP Virtualization Station VM Images (QEMU)
Look like current status is there is no practical way to deflate / shrink afterwards older existing VM image size (Virtualization Station v3.x and v4.x).
- https://community.qnap.com/t/virtualization-station-shrink-img-file-when-deleting-file-from-inside-the-vm/5764 (posted 2026-04-16)
- https://www.xeve.de/wp/2022/12/shrinking-a-qnap-virtualization-station-disk-image/
- https://qemu-project.gitlab.io/qemu/tools/qemu-img.html

There is Virtualization Station update (see snapshot from release note), which allow new virtual machines to take advantage of disk space reclamation. Feature use require
- Quest OS support TRIM/UNMAP (most of modern quest OS support this feature: Ubuntu, Debian, Windows 11, ...)
- "*Enable Space Reclaim*" in Virtual Station VM configuration's "*Storage*" tab

Virtualization Station 4.2.0.281
2026/08/28
  
[New Features]  
- *Added virtual disk space reclamation to recover unused space and improve storage efficiency*.
- Added Instant Import, which starts a virtual machine directly from its original disk files with no format conversion, speeding up deployment.  
- Added machine type selection when creating or editing a virtual machine. New virtual machines use Q35 by default.  
- Added a VNC console mode for virtual machines with GPU passthrough, so you can keep using the web console when no external monitor is connected.


## PROXMOX Images (QEMU/KVM) 
Under Construction......

- https://www.virtualizationhowto.com/2026/05/why-proxmox-wasnt-reclaiming-storage-until-i-fixed-this-setting/
- https://www.reddit.com/r/Proxmox/comments/1f59doo/clean_up_your_server_reclaim_disk_space/
- https://forum.proxmox.com/threads/reclaim-disk-space-in-lxc-raw-images.24801/
- https://dannyda.com/2020/05/26/how-to-shrink-reclaim-free-virtual-disk-space-from-virtual-machines-on-proxmox-ve-pve-windows-linux-debian-ubuntu-kali-linux-rhel-centos-fedora-etc/

Reclaiming unused disk space from a Proxmox virtual machine (VM) image requires a chain of configuration steps from inside the guest operating system all the way down to the hypervisor and thin-provisioned storage pool. 

----------
#### Step 1: Enable Discard and SSD Emulation in Proxmox

For the hypervisor to know when space is freed inside the VM, the virtual disk must pass TRIM/UNMAP signals.
-   Select your VM in the Proxmox web interface.
-   Go to **Hardware** and double-click your main Hard Disk (do not do this for EFI disks).
-   Check the boxes for **Discard** and **SSD emulation**.
-   Ensure your VM's hard disk controller is set to **Virtio-SCSI** (IDE, SATA, and standard Virtio-Block generally do not support passdown as cleanly). Restart the VM if you made controller or hardware changes.

----------
#### Step 2: Trim Free Space Inside the Guest OS

You must tell the guest operating system to send discard commands for deleted files.

For Linux Guests

-   **Manual trim:** Open the VM terminal and run:
    
    *bash*
    ```
    sudo fstrim -av
    ```
-   **Automatic trim:** Enable the systemd timer so it runs periodically:
    
    *bash*
    ```
    sudo systemctl enable --now fstrim.timer
    ```
----------
#### Step 3: Verify and Compact Storage Layers

Depending on your storage backend in Proxmox, space reclamation behaves differently:

-   **Thin LVM / ZFS / Ceph:** Once `fstrim` or Windows optimization runs, the free blocks are automatically returned to the pool (note that ZFS or thin-provisioned LVM blocks free up the allocation map, though raw file sizes on non-sparse/fixed setups might require specific handling or a backup/restore cycle if blocks were written sequentially and never unmapped).

-   **QCOW2 / File-based storage:** If you are using raw/qcow2 files on a standard directory and the image file itself won't shrink automatically, the standard method to fully compact a VM image is to:
    1.  Shut down the VM.
    2.  Backup the VM.
    3.  Destroy and restore the VM from the backup, or use `qemu-img convert` to strip out unallocated/zeroed blocks.
 
 
## VMware VMDK Images
My old VMware ESXi server is nowdays retired and current primary server is QEMU/KVM based PROXMOX (on Debian). 
- https://en.wikipedia.org/wiki/Proxmox_Virtual_Environment
- https://pve.proxmox.com/wiki/Main_Page

So return to VMDK topic later.
- https://simplyblock.io/glossary/what-is-thin-provisioning/
- https://www.cloudbolt.io/vmware-administration/vmware-thin-provisioning/


## Some Markdown File Editors
> How to use [StackEdit](https://stackedit.io/) with browser
> > - From MENU select **Import/Export || Import Markdown** text file from local disk to server
> > - WYSIVYG edit file in browser's window
> > - Read syntax examples from "***Welcome file***"
> > - From MENU select "**Import/Export || Export as Markdown**" (== download new edited file back to local disk)

> How to use [DILLINGER](https://dillinger.io/) with browser
> > - From window's upper right select "**Import**" text file from local disk to server
> > - WYSIVYG edit file in browser's window
> > - Read syntax examples from "***Untitled Document.md***"
> > - From window's upper right select "**Export as... / Markdown**" (== download new edited file back to local disk)

> Freeware offline editor [Nimbalyst](https://nimbalyst.com/blog/the-complete-guide-to-markdown-editors/)
