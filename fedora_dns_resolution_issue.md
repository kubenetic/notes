# Fedora used the wrong DNS resolver on the first place

So lately I installed a _piHole_ server one of my Raspberry Pis and setup my pfSense router to use this as DNS resolver.
But then some strange happend. I use Fedora Linux on my laptop and workstation too. On my laptop the DNS resolution of
the local domain addresses worked seamlessly but my workstation got hard time with it. 

I've checked the status of the `systemd-resolved` service with the command below.

```bash
resolvectl status
```

It returned the following output.

```bash
Global
  Protocols: LLMNR=resolve -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (eno1)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
         Protocols: +DefaultRoute LLMNR=resolve -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 1.1.1.1
       DNS Servers: 192.168.1.100 1.1.1.1 8.8.8.8
        DNS Domain: home.arpa
```

## With CLI

So my system got the list of the nameservers from the DHCP server (pfSense), but I didn't know why it started to use the
DNS server of Cloudflare. 

So I listed the connections and set the appropriate DNS server by hand on the my wired connection. After that I 
restarted the network connection. I used the commands below.

```bash
nmcli connection show
nmcli connection modify <name of the connection> ipv4.dns '192.168.1.100'
nmcli connection down <name of the connection>
nmcli connection up <name of the connection>
```

## With config file

I put the settings above into a config file on path `/etc/systemd/resolved.conf.d/dns_servers.conf`.

```text
[Resolve]
DNS=10.22.8.253
FallbackDNS=1.1.1.1
```

After this only a restart was needed for the `systemd-resolved` service.

And voila. The DNS resolution issue flied away :-)
