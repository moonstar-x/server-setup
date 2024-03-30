services:
  pihole:
    image: pihole/pihole:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - 53:53
      - 53:53/udp
      - 9500:80
    volumes:
      - ./config:/etc/pihole
      - ./dnsmasq:/etc/dnsmasq.d
    environment:
      TZ: America/Guayaquil
      WEBPASSWORD: '031688'
      FTLCONF_LOCAL_IPV4: 192.168.100.4


Settings > DNS > Permit all origins
Enable Google and Cloudflare ipv4 and ipv6 DNS servers
Add local DNS as `home.arpa` to point to local ipv4
Go into router settings and change DNS to be your server IP, in my case, `192.168.100.4` and `192.168.100.1`
If you have IPv6, make sure to do the same, in my case `device:ip:address:v6` and `fe80::1`

Now all devices can enjoy PiHole.


---

Need to check how to do TLS in local entrypoint for traefik
