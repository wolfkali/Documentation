

```bash
sudo dnf install vim dhcp-server bind bind-utils chrony radvd
```

LX-DNS-DHCP-01

```bash
sudo nmcli connection modify ens18 ipv4.addresses 192.168.1.2/24
sudo nmcli connection modify ens18 ipv4.gateway 192.168.1.1
sudo nmcli connection modify ens18 ipv4.dns "127.0.0.1 192.168.1.3"
sudo nmcli connection modify ens18 ipv4.method manual
sudo nmcli connection down ens18 && sudo nmcli connection up ens18
```

LX-DNS-DHCP-02

```bash
sudo nmcli connection modify ens18 ipv4.addresses 192.168.1.3/24
sudo nmcli connection modify ens18 ipv4.gateway 192.168.1.1
sudo nmcli connection modify ens18 ipv4.dns "172.0.0.1 192.168.1.2"
sudo nmcli connection modify ens18 ipv4.method manual
sudo nmcli connection down ens18 && sudo nmcli connection up ens18
```

LX-DNS-DHCP-01

```bash
sudo nmcli connection modify ens18 ipv6.addresses 2a01:cb14:14a6:5600:be24:11ff:fe68:53eb/64
sudo nmcli connection modify ens18 ipv6.gateway fe80::7ab2:13ff:fe14:a5a6
sudo nmcli connection modify ens18 ipv6.dns "2a01:cb14:14a6:5600:be24:11ff:fe68:53eb 2a01:cb14:14a6:5600:be24:11ff:fe8e:a0ff"
sudo nmcli connection modify ens18 ipv6.method manual
sudo nmcli connection down ens18 && sudo nmcli connection up ens18
```

LX-DNS-DHCP-02

```bash
sudo nmcli connection modify ens18 ipv6.addresses 2a01:cb14:14a6:5600:be24:11ff:fe8e:a0ff/64 fe80::be24:11ff:fe8e:a0ff/64
sudo nmcli connection modify ens18 ipv6.gateway fe80::7ab2:13ff:fe14:a5a6
sudo nmcli connection modify ens18 ipv6.dns "2a01:cb14:14a6:5600:be24:11ff:fe68:53eb 2a01:cb14:14a6:5600:be24:11ff:fe8e:a0ff"
sudo nmcli connection modify ens18 ipv6.method manual
sudo nmcli connection down ens18 && sudo nmcli connection up ens18
```

firewalld

```bash
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --zone=public --add-service=dns --permanent
sudo firewall-cmd --zone=public --add-service=dhcp --permanent
sudo firewall-cmd --zone=public --remove-service=cockpit --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --zone=public --list-all
```

## DHCP - dhcp

```bash
sudo vim /etc/dhcp/dhcpd.conf
```

```bash
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
default-lease-time 600;
max-lease-time 7200;
authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.2 192.168.1.254;
  option routers 192.168.1.1;
  option subnet-mask 255.255.255.0;
  option domain-name "valdeyron.net";
  option domain-name-servers 192.168.1.2, 192.168.1.3;
  option dhcp6.name-servers 2a01:cb14:14a6:5600:be24:11ff:fe68:53eb,2a01:cb14:14a6:5600:be24:11ff:fe8e:a0ff ;
  option ntp-servers 192.168.1.2, 192.168.1.3;
}

host poste1 {
    hardware ethernet 00:11:22:33:44:55;  # MAC du client
    fixed-address 192.168.1.100;           # IP réservée
    #option host-name "poste1";
}

ddns-updates on;
ddns-domainname "valdeyron.net.";
ddns-rev-domainname "in-addr.arpa.";

key "dhcp-key" {
	algorithm hmac-sha256;
	secret "73afQIywBvgljFNH9wTcUA9VHNA5y3eGPXTA97o7myA=";
};

```

```bash
sudo systemctl status dhcpd
sudo systemctl enable dhcpd
sudo systemctl start dhcpd
```

## DNS - Bind

```bash
sudo vim /etc/named.conf
```

```bash
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
	listen-on port 53 { 127.0.0.1; 192.168.1.2; };
	listen-on-v6 port 53 { ::1; 2a01:cb14::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost; 192.168.1.0/24; 2a01:cb14::/64;};

    	// Serveurs DNS de forwarding
    	forwarders {
        	1.1.1.1;  // DNS Cloudflare
        	8.8.8.8;  // DNS Google
            2606:4700:4700::1111; // DNS Cloudflare
            2001:4860:4860::8888; // DNS Google
    	};
	forward only;

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


zone "valdeyron.net" IN {
    type master;
    file "/etc/binfmt.d/db.valdeyron.net";
    allow-update { key dhcp-key; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/etc/binfmt.d/1.168.192.rev";
    allow-update { key dhcp-key; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

```bash
sudo vim /etc/binfmt.d/db.valdeyron.net
```

```bash
$TTL    604800
@       IN      SOA     dns1.valdeyron.net. admin.valdeyron.net. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

@       IN      NS      ns1.valdeyron.net.
dns1     IN      A       192.168.1.2
dns2     IN      A       192.168.1.3
```

```bash
sudo vim /etc/binfmt.d/1.168.192.rev
```

```bash
$TTL 86400
@   IN  SOA     ns1.valdeyron.net. admin.valdeyron.net. (
            2         ; Serial
            3600      ; Refresh
            1800      ; Retry
            604800    ; Expire
            86400 )   ; Minimum TTL

    IN  NS  ns1.valdeyron.net.

110 IN PTR poste1.valdeyron.net.

```

```bash
sudo systemctl status named
```
## NTP - Chrony

```bash
sudo vim /etc/chrony.conf
```

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
pool 2.almalinux.pool.ntp.org iburst

server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst

# Use NTP servers from DHCP.
sourcedir /run/chrony-dhcp

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
allow 192.168.1.0/24

# Serve time even if not synchronized to a time source.
local stratum 10

# Require authentication (nts or key option) for all NTP sources.
#authselectmode require

# Specify file containing keys for NTP authentication.
keyfile /etc/chrony.keys

# Save NTS keys and cookies.
ntsdumpdir /var/lib/chrony

# Insert/delete leap seconds by slewing instead of stepping.
#leapsecmode slew

# Get TAI-UTC offset and leap seconds from the system tz database.
leapsectz right/UTC

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking

```

```bash
sudo systemctl restart chronyd
chronyc sources
```


tsig-keygen dhcp-key


```bash
sudo vim /etc/radvd.conf
```

```bash
interface ens18 {
    AdvSendAdvert on;
    MaxRtrAdvInterval 30;
    prefix 2a01:cb14:14a6:5600::/64 {
        AdvOnLink on;
        AdvAutonomous on;
    };

    RDNSS 2a01:cb14:14a6:5600:be24:11ff:fe68:53eb {
        AdvRDNSSLifetime 3600;
    };

    route ::/0 {
        AdvRoutePreference medium;
        AdvRouteLifetime 1800;
    };

};
```

host impriment {
    hardware ethernet 74:BF:C0:74:04:13;  # MAC du client
    fixed-address 192.168.1.69;           # IP réservée
    #option host-name "poste1";
}

host nas {
    hardware ethernet BC:5F:F4:1B:4A:D0;  # MAC du client
    fixed-address 192.168.1.29;           # IP réservée
    #option host-name "poste1";
}

host firewalld {
    hardware ethernet BC:24:11:06:3A:AB;  # MAC du client
    fixed-address 192.168.1.23;           # IP réservée
    #option host-name "poste1";
}

host cpl-haut {
    hardware ethernet CC:32:E5:28:C5:A0;  # MAC du client
    fixed-address 192.168.1.32;           # IP réservée
    #option host-name "poste1";
}

host cpl-bas {
    hardware ethernet CC:32:E5:28:D2:40;  # MAC du client
    fixed-address 192.168.1.48;           # IP réservée
    #option host-name "poste1";
}

host cpl-salon {
    hardware ethernet F0:86:20:93:9A:DC;  # MAC du client
    fixed-address 192.168.1.100;           # IP réservée
    #option host-name "poste1";
}

host vm {
    hardware ethernet 38:f7:cd:c7:c1:9a;  # MAC du client
    fixed-address 192.168.1.19;           # IP réservée
    #option host-name "poste1";
}

@ IN NS dns1.valdeyron.net.

dns1    	IN      A       192.168.1.2
dns2    	IN      A       192.168.1.3

imprimant      	IN      A      	192.168.1.69
nas 		IN  	A 	192.168.1.29
firewalld  	IN  	A 	192.168.1.23
cpl-haut 	IN 	A 	192.168.1.32
cpl-bas 	IN 	A 	192.168.1.48
cpl-salon 	IN 	A 	192.168.1.100
vm 		IN 	A 	192.168.1.19

device-283.home IN CNAME dns1.valdeyron.net