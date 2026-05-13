================================================================================
        BIND9 DNS SERVER - COMPLETE STUDY NOTES (MIT409)
================================================================================

YOUR SERVER INFO:
  IP Address      : 10.0.2.15
  Network         : 10.0.2.0/24
  Domain          : kasparov.com
  DNS Server Name : server1.kasparov.com.
  Reverse Zone    : 2.0.10.in-addr.arpa

================================================================================
PART 1 - INSTALLATION
================================================================================

-- Install BIND9 and utilities:
   sudo apt install bind9 bind9-utils bind9-dnsutils -y

-- Install and configure firewall:
   sudo apt install firewalld -y
   sudo systemctl enable --now firewalld
   sudo firewall-cmd --add-port=53/udp --permanent
   sudo firewall-cmd --add-port=53/tcp --permanent
   sudo firewall-cmd --reload

-- Enable and start BIND9 (service is called 'named'):
   sudo systemctl enable named
   sudo systemctl start named
   sudo systemctl status named

-- View the named service file:
   sudo systemctl cat named

   INTERESTING THINGS IN named.service:
   - After=network.target        : waits for network before starting
   - Wants/Before=nss-lookup     : integrates with system name resolution
   - Restart=on-failure          : auto restarts if it crashes
   - Alias=bind9.service         : both 'named' and 'bind9' work as service names
   - ExecReload=rndc reload      : uses rndc tool to reload (not full restart)

================================================================================
PART 2 - CONFIGURING BIND SERVER
================================================================================

-- Open the main options config file:
   sudo vim /etc/bind/named.conf.options

-- Contents of /etc/bind/named.conf.options:
   options {
           directory "/var/cache/bind";

           listen-on port 53 { 127.0.0.1; 10.0.2.15; };

           allow-recursion { any; };

           allow-query { localhost; 10.0.2.0/24; };

           dnssec-validation auto;

           listen-on-v6 { any; };
   };

   EXPLANATION:
   - listen-on     : tells BIND which IP:port to listen on (IP+port = socket)
   - allow-recursion: allows recursive DNS lookups
   - allow-query   : who is allowed to send DNS queries to this server
   - directory     : where zone files are stored (/var/cache/bind)

-- Verify config (no output = no errors):
   sudo named-checkconf /etc/bind/named.conf

================================================================================
PART 3 - CREATING A FORWARD ZONE (yourlastname.com)
================================================================================

STEP 1: Create named.conf.zones file:
   sudo vim /etc/bind/named.conf.zones

   Contents:
   zone "kasparov.com" {
           type master;
           file "db.kasparov.com";
           allow-update{none;};
   };

STEP 2: Include zones file in main named.conf:
   sudo vim /etc/bind/named.conf
   -- Add this line at the bottom:
   include "/etc/bind/named.conf.zones";

STEP 3: Set permissions on zones file:
   sudo chown root:bind /etc/bind/named.conf.zones
   sudo chmod 750 /etc/bind/named.conf.zones

STEP 4: Create the zone file in /var/cache/bind/:
   sudo vim /var/cache/bind/db.kasparov.com

   Contents:
   $TTL 86400
   @   IN  SOA server1.kasparov.com.  garry.kasparov.com. (
                   2026041901  ;Serial (YYYYMMDD01)
                   15          ;Refresh (day of birth month)
                   1800        ;Retry after 30 minutes
                   1814400     ;Expire after 3 weeks
                   10800 )     ;Negative cache TTL 3 hours

   ; DNS SERVER
   @               IN  NS      server1.kasparov.com.

   ; A RECORDS
   server1.kasparov.com.   IN  A   10.0.2.15
   localhost.kasparov.com. IN  A   127.0.0.2

   ; MX RECORD
   @               IN  MX  10  mail.kasparov.com.

   ; WILDCARD WITH CUSTOM SHORT TTL
   *               300 IN  A   127.0.0.3

   ; CNAME RECORD
   search.kasparov.com.    IN  CNAME   www.google.com.

   RECORD TYPE EXPLANATIONS:
   - $TTL    : default time to live for all records
   - SOA     : Start of Authority - defines the zone and its properties
   - @       : represents the domain itself (kasparov.com)
   - IN      : stands for Internet (record class)
   - NS      : Name Server record
   - A       : IPv4 address record
   - MX      : Mail Exchanger record (needs priority number e.g. 10)
   - CNAME   : Canonical Name - alias pointing to another domain
   - *       : wildcard - matches any subdomain not explicitly defined
   - PTR     : Pointer record - used in reverse zones (IP -> hostname)

   SOA FIELD EXPLANATIONS:
   - Serial  : version number, increment when changes are made (YYYYMMDD01)
   - Refresh : how often secondary DNS checks for updates
   - Retry   : how long to wait before retrying a failed refresh
   - Expire  : how long secondary keeps zone data if primary unreachable
   - Negative TTL: how long to cache "record not found" responses

STEP 5: Set permissions on zone file:
   sudo chown root:bind /var/cache/bind/db.kasparov.com
   sudo chmod 750 /var/cache/bind/db.kasparov.com

STEP 6: Verify zone file syntax:
   sudo named-checkzone kasparov.com /var/cache/bind/db.kasparov.com
   -- Should say OK at the end

STEP 7: Restart named:
   sudo systemctl restart named

================================================================================
PART 4 - CREATING A REVERSE ZONE (IP -> hostname)
================================================================================

  HOW REVERSE ZONES WORK:
  - Your subnet is 10.0.2.0/24
  - Write subnet BACKWARDS: 2.0.10
  - Add .in-addr.arpa at the end: 2.0.10.in-addr.arpa
  - PTR records map the LAST OCTET of IP to a hostname
    e.g. 15 IN PTR server1.kasparov.com. (15 = last octet of 10.0.2.15)

STEP 1: Add reverse zone to named.conf.zones:
   sudo vim /etc/bind/named.conf.zones

   Add below existing zone:
   zone "2.0.10.in-addr.arpa" {
           type master;
           file "db.10.0.2";
           allow-update{none;};
   };

STEP 2: Create the reverse zone file:
   sudo vim /var/cache/bind/db.10.0.2

   Contents:
   $TTL 86400
   @   IN  SOA server1.kasparov.com.  garry.kasparov.com. (
                   2026041901  ;Serial
                   15          ;Refresh
                   1800        ;Retry
                   1814400     ;Expire
                   10800 )     ;Negative cache TTL

   @       IN  NS      server1.kasparov.com.
   128     IN  PTR     server1.kasparov.com.
   15      IN  PTR     localhost.kasparov.com.

STEP 3: Set permissions:
   sudo chown root:bind /var/cache/bind/db.10.0.2
   sudo chmod 750 /var/cache/bind/db.10.0.2

STEP 4: Verify reverse zone:
   sudo named-checkzone 2.0.10.in-addr.arpa /var/cache/bind/db.10.0.2

STEP 5: Restart named:
   sudo systemctl restart named

================================================================================
PART 5 - USING DIG TO VERIFY DNS
================================================================================

  dig COMMAND SYNTAX:
  dig [record type] {domain} @{DNS server}

-- Verify wildcard (2 different subdomains):
   dig anything.kasparov.com @localhost
   dig hello.kasparov.com @localhost
   (both should return 127.0.0.3)

-- Verify NS record:
   dig NS kasparov.com @localhost
   (should return server1.kasparov.com.)

-- Verify extra A record:
   dig localhost.kasparov.com @localhost
   (should return 127.0.0.2)

-- Verify MX record:
   dig MX kasparov.com @localhost
   (should return mail.kasparov.com.)

-- Verify CNAME record:
   dig search.kasparov.com @localhost
   (should return www.google.com.)

-- Verify reverse zone (PTR):
   dig -x 10.0.2.15 @localhost
   (should return localhost.kasparov.com.)

================================================================================
PART 6 - EXTRA RECORDS (MX, CUSTOM TTL, CNAME)
================================================================================

-- MX Record (Mail Exchanger):
   @ IN MX 10 mail.kasparov.com.
   - Requires a PRIORITY NUMBER (e.g. 10) - lower number = higher priority
   - Points to the mail server for the domain

-- Custom TTL on wildcard:
   * 300 IN A 127.0.0.3
   - 300 seconds = 5 minutes (short TTL)
   - Overrides the default $TTL for this specific record

-- CNAME Record (alias):
   search.kasparov.com. IN CNAME www.google.com.
   - Points search.kasparov.com to www.google.com
   - CNAME cannot coexist with other records for same name

================================================================================
IMPORTANT FILE LOCATIONS
================================================================================

   /etc/bind/named.conf              : main BIND config file
   /etc/bind/named.conf.options      : BIND options (listen-on, allow-query etc)
   /etc/bind/named.conf.zones        : zone definitions (created by us)
   /var/cache/bind/                  : zone files directory
   /var/cache/bind/db.kasparov.com   : forward zone file
   /var/cache/bind/db.10.0.2         : reverse zone file

================================================================================
IMPORTANT COMMANDS SUMMARY
================================================================================

   sudo named-checkconf /etc/bind/named.conf              : verify main config
   sudo named-checkzone {zone} {zonefile}                 : verify zone file
   sudo systemctl restart named                           : restart BIND
   sudo systemctl status named                            : check BIND status
   sudo systemctl enable named                            : enable at boot
   dig {domain} @localhost                                : test DNS lookup
   dig -x {IP} @localhost                                 : test reverse lookup
   dig MX {domain} @localhost                             : test MX record
   dig NS {domain} @localhost                             : test NS record

================================================================================
PERMISSIONS REMINDER
================================================================================

   Files in /etc/bind/ and /var/cache/bind/ should be:
   - Group owner: bind
   - Permissions: 750
   Command: sudo chown root:bind {file} && sudo chmod 750 {file}

================================================================================
END OF NOTES
================================================================================
