See also: <?add topic='NFS'?> <?add topic='SSH'?>

### Basics

-   Resolve a name via nsswitch

        getent hosts <host name>

-   [CloudShark](http://www.cloudshark.org): Sharing network traces
-   DNS Lookup

        dig <domain>
        dig <domain> +noall +answer
        dig <domain> +short
        dig MX <domain>
        dig NS <domain>
        dig ANY <domain>

        dig -x <IP>
        dig -x <IP> +short

        dig @8.8.8.8 <domain>

        dig -f input.txt +noall +answer

-   netcat Commands

        nc -l -p <port>       # Listen on port
        nc -w3 <ip> <port>  # Listen for connection from IP on port

        # Search banners
        echo | nc -v -n -w1 <ip> <port min>-<port max>

        # Port scan
        nc –v –n –z –w1 <ip> <port min>-<port max>

-   [paketlife.net cheet
    sheets](http://packetlife.net/library/cheat-sheets/) for all network
    protocols (PDFs)

### DNS RR

-   warpsrv - CLI wrapper for DNS RR connections

        apt-get install -y wrapsrv netcat
        export eval $(wrapsrv <DNS name> "netcat -z %h %p && echo http_proxy=http://%h:%p")

### Configuration

-   ethtool - Usage

        ethtool eth0                       # Print general info on eth0
        ethtool -i eth0                    # Print kernel module info
        ethtool -S eth0                    # Print eth0 traffic statistics
        ethtool -a eth0                    # Print RX, TX and auto-negotiation settings
        ethtool -p eth0                    # Blink LED

        # Changing NIC settings...
        ethtool -s eth0 speed 100
        ethtool -s eth0 autoneg off
        ethtool -s eth0 duplex full
        ethtool -s eth0 wol g               # Turn on wake-on-LAN

    Do not forget to make changes permanent in e.g.
    /etc/network/interfaces.

-   ip - Usage

        ip link show
        ip link set eth0 up
        ip addr show
        ip neigh show

-   miitool - Show Link Infos

        # mii-tool -v
        eth0: negotiated 100baseTx-FD flow-control, link ok
          product info: vendor 00:07:32, model 17 rev 4
          basic mode:   autonegotiation enabled
          basic status: autonegotiation complete, link ok
          capabilities: 1000baseT-HD 1000baseT-FD 100baseTx-FD 100baseTx-HD 10baseT-FD 10baseT-HD
          advertising:  100baseTx-FD 100baseTx-HD 10baseT-FD 10baseT-HD flow-control
          link partner: 1000baseT-HD 1000baseT-FD 100baseTx-FD 100baseTx-HD 10baseT-FD 10baseT-HD flow-control

-   Enable Jumbo Frames

        ifconfig eth1 mtu 9000

-   [NFS - Tuning
    Secrets](https://speakerdeck.com/gnb/130-lca2008-nfs-tuning-secrets-d7):
    SGI Slides on NFS Performance

#### iptables

-   [ipsets vs. iptables
    Performance](http://daemonkeeper.net/781/mass-blocking-ip-addresses-with-ipset/)
-   [ipsets - Using IP sets for simpler iptables
    rules](http://utcc.utoronto.ca/~cks/space/blog/linux/IptablesIpsetNotes)

        ipset create smtpblocks hash:net counters
        ipset add smtpblocks 27.112.32.0/19
        ipset add smtpblocks 204.8.87.0/24
        iptables -A INPUT -p tcp --dport 25 -m set --match-set smtpblocks src -j DROP

-   iptables - Loopback Routing:

        iptables -t nat -A POSTROUTING -d <internal web server IP> -s <internal network address> -p tcp --dport 80 -j SNAT --to-source <external web server IP>

-   iptables - Show active rules:

        iptables -S
        iptables -L 
        iptables -L <table>

-   iptables - Full flush:

        iptables -F
        iptables -X
        iptables -t nat -F
        iptables -t nat -X
        iptables -t mangle -F
        iptables -t mangle -X
        iptables -P INPUT ACCEPT
        iptables -P FORWARD ACCEPT
        iptables -P OUTPUT ACCEPT

-   iptables - Allow established:

        iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

-   iptables - Log failed requests:

        iptables -I INPUT 5 -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

-   iptables - Persistency on Debian:

        apt-get install iptables-persistent

        # Set some rules and call
        invoke-rc.d iptables-persistent save

-   [iptables - Persistency on Ubuntu:
    UFW](https://wiki.ubuntu.com/UncomplicatedFirewall) (Uncomplicated
    FireWall)

        ufw enable
        ufw status
        ufw allow ssh/tcp
        ufw allow from <IP> proto tcp to any port <port>
        ufw delete allow from <IP> proto tcp to any port <port>

-   fail2ban CLI Commands

        fail2ban-client status
        fail2ban-client status <jail name>

### Troubleshooting

-   Black Hole Route: To block IPs create route on loopback

        route add -net 91.65.16.0/24 gw 127.0.0.1 lo   # for a subnet
        route add  91.65.16.4 gw 127.0.0.1 lo   # for a single IP

-   Quick Access Log IP Top List

        tail -100000 access.log | awk '{print $1}' | sort | uniq -c |sort -nr|head -25

-   Find out if IP is used before configuring it

        arping <IP>

-   Traceroute with AS and network name lookup

        lft -AN www.google.de

-   Manually lookup AS

     

-   [dailychanges.com](http://www.dailychanges.com/): Tracks DNS changes

### Measuring

-   vnstat - Short term measurement bytes/packets min/avg/max:

        vnstat -l      # Live listing until Ctrl-C and summary
        vnstat -tr     # 5s automatic traffic sample

-   vnstat - Long term statistics:

        vnstat -h      # last hours (including ASCII graph)
        vnstat -d      # last days
        vnstat -w      # last weeks
        vnstat -m     # last months

        vnstat -t       # top 10 days

-   curl - Time details on HTTP requests:

        curl -w "DNS: %{time_namelookup} Connect: %{time_connect} start:  %{time_starttransfer} total:  %{time_total}\n" -o /dev/null -s http://example.com

### Discovery

-   LLDP

        lldpctl
        lldpctl eth0

-   nmap commands

        # Network scan
        nmap -sP 192.168.0.0/24

        # Host scan
        nmap <ip>
        nmap -F <ip>      # fast
        nmap -O <ip>     # detect OS
        nmap -sV <ip>     # detect services and versions
        nmap -sU <ip>     # detect UDP services

        # Alternative host discovery
        nmap -PS <ip>     # TCP SYN scan
        nmap -PA <ip>     # TCP ACK scan
        nmap -PO <ip>     # IP ping
        nmap -PU <ip>     # UDP ping

        # Alternative service discovery
        nmap -sS <ip>      
        nmap -sT <ip>
        nmap -sA <ip>
        nmap -sW <ip>

        # Checking firewalls
        nmap -sN <ip>
        nmap -sF <ip>
        nmap -sX <ip>

### Debugging

-   [X-Trace - Multi-protocol tracing
    framework](http://x-trace.net/pubs/nsdi-html/xtrace.html)
-   iptraf - Real-time statistics in ncurses interfaces
-   mtr - Debug routing/package loss issues
-   netstat - The different modes

        # Typically used modes
        netstat -rn          # List routes
        netstat -tlnp       # List all open TCP connections
        netstat -tlnpc      # Continuously do the above
        netstat -tulpen    # Extended connection view
        netstat -a           # List all sockets

        # And more rarely used
        netstat -s            # List per protocol statistics
        netstat -su          # List UDP statistics
        netstat -M           # List masqueraded connections
        netstat -i            # List interfaces and counters
        netstat -o           # Watch time/wait handling

-   nttcp - TCP performance testing

        # On sending host
        nttcp -t -s

        # On receiving host
        nttcp -r -s

-   List Kernel Settings

        sysctl net

-   [SNMP - Dump all
    MIBs](http://net-snmp.sourceforge.net/wiki/index.php/TUT:snmpwalk):
    When you need to find the MIB for an object known only by name try

        snmpwalk -c public -v 1 -O s <myhost> .iso | grep <search string>

-   [Hurricane Electric - BGP Tools](http://bgp.he.net/): Statistics on
    all AS as well as links to their looking glasses.
-   tcpdump - Be verbose and print full package hex dumps:

         tcpdump -i eth0 -nN -vvv -xX -s 1500 port <some port>

-   tcpdump - Non-promiscuous mode to list only traffic that the network
    stack processes:

        tcpdump -e ...

-   [tcpdump -
    Tutorial](http://delicious.com/redirect?url=http%3A//dmiessler.com/study/tcpdump/):
    Many usage examples.

        # Filter port
        tcpdump port 80
        tcpdump src port 1025 
        tcpdump dst port 389
        tcpdump portrange 21-23

        # Filter source or destination IP
        tcpdump src 10.0.0.1
        tcpdump dest 10.0.0.2

        # Filter  everything on network 
        tcpdump net 1.2.3.0/24

        # Logically operators
        tcpdump src port 1025 and tcp 

        # Provide full hex dump of captured HTTP packages
        tcpdump -s0 -x port 80

        # Filter TCP flags (e.g. RST)
        tcpdump 'tcp[13] & 4!=0'

-   [darkstat](https://unix4lyfe.org/darkstat/) - libpcap monitoring
