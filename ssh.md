================================================================================
  SSH, SFTP & FTP — Study Notes
  Source: SSH & SFTP slides, FTP slides, MIT450 Lesson 9
================================================================================

TABLE OF CONTENTS
-----------------
  1. SSH — Secure Shell
  2. SSH Key Authentication
  3. SSH Server Config (sshd_config)
  4. SCP — Secure Copy
  5. SFTP — Secure File Transfer Protocol
  6. Chroot Jails (SSH/SFTP)
  7. SSH Tunnels
  8. FTP — File Transfer Protocol
  9. vsftpd — Configuration
  10. FTPS — FTP over SSL
  11. Quick Reference / Exam Tips

================================================================================
  1. SSH — SECURE SHELL
================================================================================

WHAT IS SSH?
  - Protocol for secure remote access to Linux/Unix systems
  - Encrypts all traffic (commands, passwords, file transfers)
  - Default port: 22
  - Preinstalled on most Linux server distributions
  - Supports: remote shell, SFTP/SCP file transfer, SSH tunneling

HOW SSH WORKS (connection steps):
  1. TCP handshake establishes the connection
  2. Server sends accepted encryption protocols + its public key
  3. Client checks server key against ~/.ssh/known_hosts
       - First connect: prompted to accept and store the key
       - Key mismatch on reconnect: WARNING — possible MITM attack
  4. Asymmetric encryption used to establish a secure session
  5. Symmetric encryption used for all further session data
       NOTE: Asymmetric is slow (used only to exchange the symmetric key)
             Symmetric is fast (used for all actual data transfer)

BASIC SSH COMMANDS:
  ssh hostname                        Connect as current local user
  ssh user@hostname                   Connect as specific user
  ssh root@php.scweb.ca               Connect as root
  ssh -C root@php.scweb.ca            Connect with compression enabled
  ssh -p 50022 server.com             Connect on non-default port 50022
  ssh user@host "command"             Run a single command without full shell

  WARNING: Never run 'shutdown' or 'reboot' over SSH unless you have
           another way back into the machine!

================================================================================
  2. SSH KEY AUTHENTICATION
================================================================================

WHY USE KEY AUTH?
  - More secure than passwords (no password travels over the network)
  - Resistant to brute-force attacks
  - Enables passwordless automated logins (scripts, cron jobs)

KEY PAIR CONCEPTS:
  - Private key: stays on YOUR machine, never share it
  - Public key:  placed on the SERVER in ~/.ssh/authorized_keys
  - When connecting, server challenges client using public key math
    Only the holder of the matching private key can respond correctly

GENERATING KEYS:
  ssh-keygen                                     Interactive (prompts for options)
  ssh-keygen -t rsa -b 4096 -C "email@x.com"    4096-bit RSA key with label
  ssh-keygen -t ed25519 -C "email@x.com"        Modern alternative (recommended)

  Key types:
    rsa     — widely supported, use 4096-bit minimum
    ed25519 — faster, more secure, shorter keys (preferred)
    ecdsa   — elliptic curve, good alternative
    dsa     — outdated, avoid

  Generated files (default location ~/.ssh/):
    id_rsa          private key  (protect this!)
    id_rsa.pub      public key   (safe to share/copy to servers)

COPYING PUBLIC KEY TO SERVER:
  ssh-copy-id user@server               Easiest method — appends key automatically
  ssh-copy-id -p 50022 user@server      With non-default port

  Manual method:
    cat ~/.ssh/id_rsa.pub               Print your public key
    Then paste its contents into this file ON THE SERVER:
      ~/.ssh/authorized_keys

SECURITY HARDENING (after key auth is working):
  In /etc/ssh/sshd_config:
    PasswordAuthentication no           Disables password login entirely
    This eliminates brute-force attacks completely

================================================================================
  3. SSH SERVER CONFIG — /etc/ssh/sshd_config
================================================================================

KEY FILE INFO:
  Config file:   /etc/ssh/sshd_config
  Service name:  sshd
  Man page:      man sshd_config
  Mozilla guide: https://infosec.mozilla.org/guidelines/openssh

IMPORTANT SETTINGS:
  PermitRootLogin no                    Block root login entirely (recommended)
  PermitRootLogin prohibit-password     Allow root only with key (no password)
  PasswordAuthentication no             Disable password auth (use keys only)

MATCH BLOCKS (per user/group/IP rules):
  Match User franco
      AllowTcpForwarding no             Disable SSH tunneling for user franco

  Match Group students
      AllowTcpForwarding no             Disable tunneling for all in group

  Match Address 172.22.100.0/24,172.22.5.0/24,127.0.0.1
      PermitRootLogin without-password  Allow root login from trusted networks
      PasswordAuthentication yes

VALIDATING CONFIG (CRITICAL STEP):
  sshd -t                               Test config for syntax errors

  ALWAYS run 'sshd -t' before restarting sshd when connected remotely.
  A broken config + service restart = locked out of your server!

  Workflow:
    1. Edit /etc/ssh/sshd_config
    2. Run: sshd -t
    3. If no errors: systemctl restart sshd
    4. Test connection in a NEW terminal before closing current session

================================================================================
  4. SCP — SECURE COPY
================================================================================

SYNTAX:
  scp [options] [[user@host1:]]source [[user@host2:]]destination

  Remote path format: user@hostname:/path/to/file
  Use ~ for home directory: user@hostname:~/filename

EXAMPLES:
  scp root@php.scweb.ca:~/hello.txt /tmp/
    Download hello.txt from root's home on remote → local /tmp/

  scp /etc/hosts root@php.scweb.ca:/etc/hosts
    Upload local /etc/hosts → overwrite remote /etc/hosts

  scp -r /local/folder user@server:/remote/path/
    Copy entire directory recursively

  scp -P 50022 file.txt user@server:/dest/
    Use non-default port (capital -P for scp, lowercase -p for ssh)

USE CASE: Good for quick one-off transfers in scripts or command line.
          Not ideal for interactive browsing of remote files.

================================================================================
  5. SFTP — SECURE FILE TRANSFER PROTOCOL
================================================================================

WHAT IS SFTP?
  - Secure File Transfer Protocol — runs over SSH (port 22)
  - Interactive session (like FTP but encrypted)
  - NOT the same as FTPS (FTP+TLS)
  - Better than SCP for browsing and managing remote files interactively
  - Supported by GUI clients: FileZilla, Cyberduck, WinSCP

CONNECTING:
  sftp user@hostname
  sftp root@php.scweb.ca
  sftp -P 50022 user@server         Non-default port

SFTP COMMANDS (inside the sftp> prompt):
  FILE TRANSFER:
    put filename          Upload local file to current remote directory
    get filename          Download remote file to current local directory
    put -r folder/        Upload entire folder recursively
    get -r folder/        Download entire folder recursively

  NAVIGATION:
    ls                    List files in remote directory
    lls                   List files in LOCAL directory
    cd /remote/path       Change remote directory
    lcd /local/path       Change LOCAL directory
    pwd                   Show current remote directory
    lpwd                  Show current LOCAL directory

  DIRECTORY MANAGEMENT:
    mkdir dirname         Create directory on remote server
    lmkdir dirname        Create directory LOCALLY
    rmdir dirname         Remove remote directory

  SESSION:
    exit  (or quit, bye)  Close SFTP session
    help                  List all available commands

KEY CONCEPT — Two "worlds" in SFTP:
  Commands without 'l' prefix = operate on REMOTE server
  Commands WITH 'l' prefix    = operate LOCALLY
  Example: ls = remote listing,  lls = local listing
           cd = change remote,   lcd = change local

================================================================================
  6. CHROOT JAILS (SSH / SFTP)
================================================================================

WHAT IS A CHROOT JAIL?
  - Changes the "root directory" a user sees when they log in
  - User cannot navigate above their assigned directory
  - /home/user1 becomes / from that user's perspective
  - Prevents access to sensitive files: /etc/passwd, /etc/shadow, etc.
  - Common use case: SFTP-only users who should only see their own files

EXAMPLE:
  Normal filesystem view:  / → /etc, /home, /var, ...
  Jailed user's view:      /  (which is actually /home/user1)
  They cannot 'cd ..' above their jail — the system won't allow it

CRITICAL PERMISSION REQUIREMENT:
  The chroot directory MUST NOT be writable by the jailed user.
  OpenSSH enforces this as a security requirement.

  Correct setup:
    chown root:user1 /home/user1      Owner=root, Group=user's primary group
    chmod 550 /home/user1             Permissions: r-xr-x---

  Resulting ls -l output:
    dr-xr-x--- 5 root user1 /home/user1

  Since user can't write to their home dir, create a subdir for their files:
    mkdir /home/user1/files
    chown user1:user1 /home/user1/files    User owns this subfolder

CONFIGURING SFTP CHROOT IN /etc/ssh/sshd_config:
  Match Group sftpusers
      ForceCommand internal-sftp      Forces SFTP only — no shell access
      ChrootDirectory %h              %h = user's home directory

  Tokens available for ChrootDirectory:
    %h    User's home directory
    %u    Username

  NOTE: For SFTP chroot — no extra setup needed beyond the config above.
        For interactive SSH chroot — also need: shell binary + /dev bindings.

SETUP CHECKLIST FOR SFTP CHROOT:
  [ ] Create user (assign shell /sbin/nologin if FTP-only)
  [ ] Add user to sftpusers group
  [ ] Set chroot dir ownership: chown root:username /home/username
  [ ] Set chroot dir permissions: chmod 550 /home/username
  [ ] Create writable subdir: mkdir /home/username/files
  [ ] Add Match block to /etc/ssh/sshd_config
  [ ] Validate config: sshd -t
  [ ] Restart sshd: systemctl restart sshd

================================================================================
  7. SSH TUNNELS
================================================================================

WHAT ARE SSH TUNNELS?
  - Route arbitrary TCP traffic through an encrypted SSH connection
  - Access internal services from outside a network
  - Bypass network restrictions
  - Create SOCKS proxies for routing all browser traffic

NOTE: You can disable tunneling per user/group in sshd_config:
  AllowTcpForwarding no

--- LOCAL PORT FORWARDING (-L) ---
  "Forward a local port to somewhere the SSH server can reach"

  ssh -L localport:remotehost:remoteport user@sshserver

  Example:
    ssh -L 54000:10.13.37.2:3389 root@server.com
    Opens port 54000 locally → traffic goes through server.com → 10.13.37.2:3389
    Use case: Access an internal RDP server from outside the network

--- REMOTE PORT FORWARDING (-R) ---
  "Open a port on the remote server that connects back to your local machine"

  ssh -R remoteport:localhost:localport user@sshserver

  Example:
    ssh -R 54000:172.16.2.4:3389 root@server.com
    Opens port 54000 on server.com → routes back to 172.16.2.4:3389 locally
    Use case: Expose a local service to the remote server

--- DYNAMIC / SOCKS PROXY (-D) ---
  "Route ALL traffic through SSH server"

  ssh -D localport user@sshserver

  Example:
    ssh -D 2001 root@server.com
    Creates SOCKS5 proxy on localhost:2001 — point browser here
    All traffic appears to come from server.com's network location

GATEWAYPORTS:
  By default, tunnel ports bind only to localhost (127.0.0.1)
  To allow OTHER machines on your network to use your tunnel:

  In /etc/ssh/sshd_config on the server:
    GatewayPorts yes

TUNNEL SUMMARY TABLE:
  Flag    Direction         Typical Use
  -L      local → remote    Access internal services from outside
  -R      remote → local    Expose local service to remote network
  -D      dynamic (SOCKS)   Route all browser/app traffic through server

================================================================================
  8. FTP — FILE TRANSFER PROTOCOL
================================================================================

WHAT IS FTP?
  - Protocol for transferring files between hosts
  - Default port: 21 (command channel)
  - Unencrypted by default — passwords sent in cleartext!
  - Uses TWO separate channels:
      Command channel (port 21): sends instructions
      Data channel: transfers actual file data

TWO FTP MODES:

  ACTIVE MODE:
    - Client opens command channel to server on port 21
    - Server initiates data connection FROM port 20 → client's specified port
    - Problems:
        Firewalls block unexpected inbound connections to client
        NAT breaks active mode (server can't reach private IP)
    - Use case: Rare, mostly legacy environments

  PASSIVE MODE (recommended):
    - Client opens command channel to server on port 21
    - Server tells client which port to connect to (from pasv range)
    - Client initiates the data connection to that server port
    - Why it's better:
        Client initiates all connections → firewall-friendly
        Works behind NAT
        Easier for clients — server owns the complexity
    - Always configure your FTP server in passive mode

================================================================================
  9. vsftpd — CONFIGURATION
================================================================================

VSFTPD BASICS:
  Full name:    vsftpd = Very Secure FTP Daemon
  Config file:  /etc/vsftpd.conf
  Docs:         https://linux.die.net/man/5/vsftpd.conf

  vsftpd checks that each user's shell is listed in /etc/shells
  For FTP-only users, set shell to /sbin/nologin (disables SSH/terminal)
  Make sure /sbin/nologin is in /etc/shells or vsftpd rejects the user

USER ACCESS CONTROL:

  Layer 1 — /etc/ftpusers (hardcoded block list, always enforced):
    Lists users who can NEVER log in via FTP
    root is always here by default
    Checked first, before anything else

  Layer 2 — userlist (flexible allow/deny):
    userlist_enable=YES                   Enable the user list feature
    userlist_deny=YES                     List acts as DENY list (block listed users)
    userlist_deny=NO                      List acts as ALLOW list (only listed users allowed)
    userlist_file=/etc/vsftpd.user_list   Path to the list file (file must exist)

  userlist_deny logic summary:
    YES  →  deny list   — everyone except listed users can log in
    NO   →  allow list  — ONLY listed users can log in (whitelist mode)

CHROOT CONFIGURATION:
  chroot_local_user=YES                   Jail ALL local users by default
  chroot_local_user=NO                    Do NOT jail users by default

  chroot_list_enable=YES                  Enable the exceptions/override list
  chroot_list_file=/etc/vsftpd.chroot_list

  Combined behavior:
    chroot_local_user=NO  + list enabled  →  listed users ARE jailed
    chroot_local_user=YES + list enabled  →  listed users are EXEMPT from jail

PASSIVE MODE:
  pasv_enable=YES                         Enable passive mode
  pasv_min_port=10090                     Minimum port for data connections
  pasv_max_port=10100                     Maximum port for data connections

  Port range = number of simultaneous passive connections supported
  (10090-10100 = 10 ports = 10 simultaneous clients)
  Open this port range in your firewall as well!

================================================================================
  10. FTPS — FTP OVER SSL
================================================================================

WHAT IS FTPS?
  - FTP wrapped in SSL/TLS encryption
  - NOT the same as SFTP (completely different protocol)
  - Encrypts passwords and file data in transit
  - Still uses port 21 + passive port range

GENERATE A SELF-SIGNED CERTIFICATE:
  openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout vsftpd.key -out vsftpd.crt

SSL CONFIGURATION IN /etc/vsftpd.conf:
  rsa_cert_file=/etc/ssl/private/vsftpd.crt     Public certificate
  rsa_private_key_file=/etc/ssl/private/vsftpd.key   Private key

  ssl_enable=YES                        Enable SSL
  allow_anon_ssl=NO                     Don't allow anonymous SSL users
  force_local_logins_ssl=YES            Force SSL on command channel (passwords)
  force_local_data_ssl=YES              Force SSL on data channel (file contents)

  ssl_tlsv1=YES                         Allow TLS v1 (keep enabled)
  ssl_sslv2=NO                          Disable SSLv2 (broken/insecure)
  ssl_sslv3=NO                          Disable SSLv3 (POODLE vulnerability)

FTPS vs SFTP COMPARISON:
  Feature           FTPS                    SFTP
  Protocol base     FTP + TLS               SSH
  Port              21 + passive range      22 only
  Encryption        Optional (configurable) Always on
  Firewall          Harder (two channels)   Easy (single port)
  Setup             More complex            Simpler
  Preferred?        Legacy enterprise use   Modern standard

================================================================================
  11. QUICK REFERENCE / EXAM TIPS
================================================================================

SSH HANDSHAKE ORDER (memorize this):
  TCP handshake → server key exchange → client verifies (known_hosts)
  → asymmetric encryption → symmetric encryption

WHY ASYMMETRIC THEN SYMMETRIC?
  Asymmetric (RSA etc.) is slow — only used to safely exchange a key
  Symmetric (AES etc.) is fast — used for all actual session data
  Same pattern used by HTTPS/TLS

KEY FILES TO KNOW:
  /etc/ssh/sshd_config              SSH server configuration
  ~/.ssh/authorized_keys            Public keys allowed to authenticate
  ~/.ssh/known_hosts                Remembered server public keys
  /etc/vsftpd.conf                  FTP server configuration
  /etc/ftpusers                     FTP hardcoded deny list
  /etc/shells                       Valid shells (vsftpd checks this)

IMPORTANT COMMANDS:
  sshd -t                           Validate SSH config (ALWAYS before restart)
  ssh-keygen -t ed25519             Generate modern SSH key pair
  ssh-copy-id user@server           Copy public key to server
  systemctl restart sshd            Restart SSH server after config changes

CHROOT RULES (common exam topic):
  - Chroot dir must be owned by root
  - Chroot dir must NOT be writable by the jailed user
  - Permissions: 550 (dr-xr-x---), owner: root, group: user's group
  - Create a subdirectory inside the jail for user's actual files

USERLIST LOGIC (easy to mix up):
  userlist_deny=YES   →   deny list   (listed users are BLOCKED)
  userlist_deny=NO    →   allow list  (ONLY listed users allowed in)

CHROOT_LOCAL_USER LOGIC (easy to mix up):
  chroot_local_user=YES + list  →  listed users are EXEMPT (not jailed)
  chroot_local_user=NO  + list  →  listed users ARE jailed

ACTIVE vs PASSIVE FTP:
  Active  = server calls back to client (firewall problems, avoid)
  Passive = client initiates both connections (preferred, firewall-friendly)

SFTP ≠ FTPS:
  SFTP  = SSH-based, port 22, always encrypted
  FTPS  = FTP+TLS, port 21, encrypted only if configured

%h TOKEN in sshd_config:
  ChrootDirectory %h   →   %h expands to the user's home directory

VALIDATE SSH CONFIG BEFORE ANYTHING ELSE:
  sshd -t    ← run this. always. no exceptions.

================================================================================
  END OF NOTES — ssh.txt
================================================================================
