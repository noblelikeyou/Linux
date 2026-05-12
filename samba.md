================================================================================
                            NFS & SAMBA STUDY NOTES
================================================================================

TABLE OF CONTENTS
  [1] Objectives
  [2] NFS - Network File System
      [2.1] What is NFS?
      [2.2] NFS Configuration File (/etc/exports)
      [2.3] NFS and User IDs
      [2.4] NFS Permission Options
      [2.5] Mounting NFS
  [3] Samba (SMB)
      [3.1] What is Samba?
      [3.2] Samba Setup
      [3.3] Samba Share Configuration (smb.conf)
      [3.4] SELinux and Samba
  [4] Quick Reference / Cheat Sheet


================================================================================
  [1] OBJECTIVES
================================================================================

  - Configure an NFS server
  - Configure a Samba (SMB) share
  - Mount network drives manually and at boot


================================================================================
  [2] NFS - NETWORK FILE SYSTEM
================================================================================

--------------------------------------------------------------------------------
  [2.1] WHAT IS NFS?
--------------------------------------------------------------------------------

  - Native Linux/UNIX file-sharing protocol
  - Allows remote directories to appear as local folders
  - Latest version: NFSv4
  - Debian package required:

      sudo apt install nfs-kernel-server


--------------------------------------------------------------------------------
  [2.2] NFS CONFIGURATION FILE (/etc/exports)
--------------------------------------------------------------------------------

  NFS shares are defined in: /etc/exports

  General Syntax:
      <directory>  <host or network>(options)

  Example:
      /shared 192.168.1.1(rw,sync,no_subtree_check,root_squash)

  NOTE: There must be NO space between the host/network and the options.

  After editing /etc/exports, reload with:
      sudo exportfs -ra


--------------------------------------------------------------------------------
  [2.3] NFS AND USER IDs
--------------------------------------------------------------------------------

  - Permissions depend on UID (User ID), NOT the username string
  - NFS checks numeric UID across systems
  - UIDs must match on both server and client for correct permissions

  To find a user's UID:
      id username
      # or
      cat /etc/passwd | grep username


--------------------------------------------------------------------------------
  [2.4] NFS PERMISSION OPTIONS
--------------------------------------------------------------------------------

  Common Options:
  ---------------
    ro                Read-only
    rw                Read/write
    sync              Safer — writes must complete before reply
    async             Faster — but risk of data loss on crash

  Security & Behavior Options:
  -----------------------------
    no_subtree_check  Faster, skips subtree verification checks
    root_squash       Remote root becomes anonymous user (RECOMMENDED)
    no_root_squash    Remote root stays root — DANGEROUS, avoid in production
    all_squash        All remote users become anonymous
    anonuid=<id>      Set the UID for anonymous (squashed) access
    anongid=<id>      Set the GID for anonymous (squashed) access

  Full example line:
      /shared 192.168.1.1(rw,sync,no_subtree_check,root_squash)

  !! IMPORTANT: No space between the network address and the opening parenthesis.
     WRONG:  /shared 192.168.1.0/24 (rw,sync)
     RIGHT:  /shared 192.168.1.0/24(rw,sync)


--------------------------------------------------------------------------------
  [2.5] MOUNTING NFS
--------------------------------------------------------------------------------

  Manual Mount (does NOT persist after reboot):
  ----------------------------------------------
      sudo mount -o rw 172.16.125.128:/shared /mount/shared

      Breakdown:
        Remote path    : 172.16.125.128:/shared
        Local mount pt : /mount/shared
        Options        : rw (read/write)

  Mounting NFS at Boot via /etc/fstab:
  --------------------------------------
      Syntax:
        <remote>  <local>  <type>  <options>  <dump>  <pass>

      Example:
        172.16.125.128:/shared  /mount/shared  nfs  rw,users,noauto  0  0

      Option breakdown:
        rw       Read/write
        users    Non-root users can mount/unmount
        noauto   Do NOT mount automatically at boot (mount manually when needed)
        0 0      No dump, no fsck pass

  Create the local mount point if it doesn't exist:
      sudo mkdir -p /mount/shared


================================================================================
  [3] SAMBA (SMB)
================================================================================

--------------------------------------------------------------------------------
  [3.1] WHAT IS SAMBA?
--------------------------------------------------------------------------------

  SMB = Server Message Block — Windows' native file-sharing protocol
  Samba = Linux implementation of SMB

  - Allows Linux servers to share folders with Windows clients
  - Windows machines can connect natively (no extra client software)

  Two main services:
    smb.service   File sharing (core)
    nmb.service   NetBIOS name service (used by SMB v1 / legacy systems)

  NOTE: SMBv1 is insecure — still exists in legacy environments.

  Enable and start services:
      sudo systemctl enable --now smb
      sudo systemctl enable --now nmb


--------------------------------------------------------------------------------
  [3.2] SAMBA SETUP
--------------------------------------------------------------------------------

  Step 1 — Install Samba packages:
      sudo apt install samba smbclient samba-common

  Step 2 — Add a Samba user (Samba has its own password database):
      sudo smbpasswd -a username

      NOTE: The user must already exist as a Linux system user.
            smbpasswd -a adds them to Samba and sets their Samba password.

  Step 3 — Edit the Samba configuration file:
      sudo vim /etc/samba/smb.conf

      The file contains:
        - [global] section   : server-wide settings
        - [share] sections   : individual share definitions

  Step 4 — Validate config (always do this before restarting):
      testparm

  Step 5 — Restart Samba:
      sudo systemctl restart smb nmb


--------------------------------------------------------------------------------
  [3.3] SAMBA SHARE CONFIGURATION (smb.conf)
--------------------------------------------------------------------------------

  --- Example 1: Public/Guest Share ---

      [shared]
          comment     = Shared
          path        = /smbshare
          read only   = no
          browsable   = yes
          guest ok    = yes
          force user  = nobody

      Explanation:
        read only = no    Writable by clients
        guest ok  = yes   No password needed (anonymous access)
        force user = nobody   All files owned by 'nobody'
        path = /smbshare  Directory being shared


  --- Example 2: Restricted Group Share ---

      [projects]
          comment        = Project files
          path           = /srv/projects
          read only      = no
          guest ok       = no
          valid users    = @projectgroup
          force group    = projectgroup
          create mask    = 0660
          directory mask = 0770

      Explanation:
        guest ok = no           Only authenticated users
        valid users = @group    Only members of 'projectgroup' can access
        force group             All files created belong to this group
        create mask = 0660      New files: owner rw, group rw, others none
        directory mask = 0770   New dirs: owner rwx, group rwx, others none


  Accessing the share from Linux:
      smbclient //192.168.1.10/shared -U username

  Mounting from Linux (persistent via /etc/fstab):
      //192.168.1.10/shared  /mnt/win_share  cifs  username=alice,password=pass  0  0

  Accessing from Windows (File Explorer address bar):
      \\192.168.1.10\shared


--------------------------------------------------------------------------------
  [3.4] SELINUX AND SAMBA
--------------------------------------------------------------------------------

  If SELinux is enabled, the shared directory needs the correct context.

  Set Samba SELinux context on a directory:
      sudo chcon -t samba_share_t /smbshare

  Reset/restore SELinux context to default:
      sudo restorecon -v /smbshare


================================================================================
  [4] QUICK REFERENCE / CHEAT SHEET
================================================================================

  NFS
  ---
    Install server        : sudo apt install nfs-kernel-server
    Config file           : /etc/exports
    Reload exports        : sudo exportfs -ra
    Manual mount          : sudo mount -o rw <server>:<path> <local>
    Persistent mount      : /etc/fstab entry with type nfs
    Check user UID        : id <username>

  Samba
  -----
    Install               : sudo apt install samba smbclient samba-common
    Config file           : /etc/samba/smb.conf
    Add Samba user        : sudo smbpasswd -a <username>
    Validate config       : testparm
    Restart services      : sudo systemctl restart smb nmb
    Enable services       : sudo systemctl enable --now smb nmb
    SELinux context       : sudo chcon -t samba_share_t <path>
    Restore SELinux       : sudo restorecon -v <path>
    Connect (Linux)       : smbclient //<ip>/<share> -U <user>
    Connect (Windows)     : \\<ip>\<share>

  NFS vs Samba
  ------------
    Protocol              : NFS uses NFS | Samba uses SMB/CIFS
    Best for              : NFS = Linux-to-Linux | Samba = Linux-to-Windows
    Auth model            : NFS = UID-based | Samba = username/password
    Windows native        : NFS = No | Samba = Yes
    Printer sharing       : NFS = No | Samba = Yes
    Config file           : NFS = /etc/exports | Samba = /etc/samba/smb.conf

================================================================================
                                  END OF NOTES
================================================================================
