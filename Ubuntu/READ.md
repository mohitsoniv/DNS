# Domain Name System (DNS) by Mohit Soni (mohitsoniv)
## - Apache setup with Coustom Domain on Ubuntu
### Step.1 Installing and enable Apach2 on Ubuntu.
```
apt-get updat
apt-get install apache2 -y
systemctl enable apach2
systemctl start apach2
systemctl status apach2
```
### Step.2 Installing Firewall.
```
apt-get install firewalld -y
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
apt install bind9 bind9utils bind9-doc dnsutils -y
sudo systemctl enable bind9
sudo systemctl start bind9
sudo systemctl status bind9
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
#### Create Zones directory at /etc/bind/zones
```
mkdir zones
```
#### path: /etc/bind/zones/mypage.com.fzone
```
nano /etc/bind/zones/filename
```
```
$TTL 604800
@   IN  SOA ns1.mypage.com. admin.mypage.com. (
        2025051901 ; Serial
        604800     ; Refresh
        86400      ; Retry
        2419200    ; Expire
        604800 )   ; Negative Cache TTL

; Name servers
    IN  NS  ns1.mypage.com.

; A records
@       IN  A   public ip   
ns1     IN  A   public ip     
www     IN  A   public ip    
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
nano /etc/bind/named.conf.local
```
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "mypage.com" {
    type master;
    file "/etc/bind/zones/mypage.com.fzone";
};

```
### Step.7 Validate your DNS zone file (mypage.com.fzone) for the domain mypage.com.
```
 named-checkzone mypage.com mypage.com.fzone
```
### Step.8 Restart Bind 
```
systemctl restart bind9
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
## Note:
### Here’s how to configure your DNS settings on different operating systems:


#### - Windows: Go to Control Panel > Network and Internet > Network and Sharing Center > Change adapter settings. Right-click your network connection, select Properties, then select Internet Protocol Version 4 (TCP/IPv4) or Version 6 (TCP/IPv6) and click Properties. Here, you can set your preferred DNS server.


#### - MacOS: Go to System Preferences > Network, select your network interface, click Advanced, and go to the DNS tab. You can add your DNS server here.


#### - Linux: This depends on your distribution and network manager, but typically you can edit /etc/resolv.conf directly or configure through network management tools (like NetworkManager) to add your DNS server.

