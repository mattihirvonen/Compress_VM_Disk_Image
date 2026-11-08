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

## QEMU (QNAP Virtualization Station) VM Images
Look like current status in there is no way to deflate / shrink VM image size (Virtualization Station v3.x and v4.x).
- https://community.qnap.com/t/virtualization-station-shrink-img-file-when-deleting-file-from-inside-the-vm/5764 (posted 2026-04-16)
- https://www.xeve.de/wp/2022/12/shrinking-a-qnap-virtualization-station-disk-image/
- https://qemu-project.gitlab.io/qemu/tools/qemu-img.html


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
