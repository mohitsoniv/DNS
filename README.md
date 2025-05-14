# Domain Name System (DNS) by Mohit Soni (mohitsoniv)
## - Apache setup with Coustom Domain
### Step.1 Installing and enable httpd on linux machine.
```
yum update
yum install httpd -y
systemctl enable httpd.service
systemctl start httpd.service
systemctl status httpd.service
```
### Step.2 Installing Firewall.
```
yum install firewalld -y
systemctl enable firewalld
systemctl start firewalld
systemctl status firewalld
```
#### Configure http.
```
firewall-cmd --add-service=http --permanent
firewall-cmd --reload
```
### Step.3 Install Bind.
BIND, standing for Berkeley Internet Name Domain, is a widely used software suite for managing the Domain Name System (DNS). It's essentially the DNS server software that translates human-readable domain names (like google.com) into the numerical IP addresses that computers use to communicate.
```
yum install bind -y
systemctl enable named
systemctl start named
systemctl status named
```
### Step.4 Configure DNS with firewall.
```
firewall-cmd --add-service=dns --permanent
firewall-cmd --reload
```
### Step.5 Create forward zone file to define DNS records for the domain name.
#### refrence:  https://bind9.readthedocs.io/en/v9.20.8/chapter3.html
#### Public IP on linux
```
curl ifconfig.me 
```
#### path: /var/named/filename
```
nano /var/named/mypage.com.fzone
```
```
$TTL 2d    ; default TTL for zone

@         IN      SOA   ns1.example.com. hostmaster.example.com. (
                                800 ; serial number
                                12h        ; refresh
                                15m        ; update retry
                                4d         ; expiry
                                2h         ; minimum
                                )
; name server RR for the domain
           IN      NS      ns1.example.com.
www        IN      A       your public ip
```
### Step.6  Configure private ip and domain name in named.conf file.
#### Private IP on linux
```
ifconfig
```
or
```
ip addr show
```
```
nano /./etc/named.conf
```
```
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; add private ip; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; };

        /* 
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable 
           recursion. 
         - If your recursive DNS server has a public IP address, you MUST enable access 
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification 
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface 
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

zone "mypage.com" IN {
        type master;
        file "mypage.com.fzone";
        allow-query {any;};

};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```
### Step.7 Validate your DNS zone file (mypage.com.fzone) for the domain mypage.com.
```
 named-checkzone mypage.com mypage.com.fzone
```
### Step.8 Restart Bind 
```
systemctl restart named
```
### Step.9 To resolve the domain www.mypage.com to its IP address using the system’s current DNS setting.
```
 nslookup www.mypage.com
```
#### if output will be like this 
#### Output
```
Server:         172.31.0.2
Address:        172.31.0.2#53

Non-authoritative answer:
www.mypage.com  canonical name = mypage.com.
Name:   mypage.com
Address: 15.197.148.33
Name:   mypage.com
Address: 3.33.130.190
```
#### then
### Step.10 Open main configuration file for DNS name resolution.
#### path /etc/resolv.conf
```
nano /etc/resolv.conf
```
```
nameserver 127.0.0.1  # loopback address or localhost
```
```
nslookup www.mypage.com
```
#### Output
```
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   www.mypage.com
Address: 35.171.8.231
```
