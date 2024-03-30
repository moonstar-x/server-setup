```yaml
services:
  dns:
    image: pihole/pihole:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    networks:
      default:
      proxy_external:
        aliases:
          - pihole
    ports:
      - 53:53
      - 53:53/udp
    volumes:
      - ./config:/etc/pihole
      - ./dnsmasq:/etc/dnsmasq.d
    environment:
      TZ: America/Guayaquil
      WEBPASSWORD: '031688'
      FTLCONF_LOCAL_IPV4: 192.168.100.4
      VIRTUAL_HOST: dns.home.arpa
    labels:
      traefik.enable: true
      traefik.docker.network: proxy_external
      traefik.http.routers.pihole.rule: Host(`dns.home.arpa`)
      traefik.http.routers.pihole.entrypoints: local
      traefik.http.routers.pihole.service: pihole@docker
      traefik.http.services.pihole.loadbalancer.server.port: 80

networks:
  proxy_external:
    external: true
```


Settings > DNS > Permit all origins
Enable Google and Cloudflare ipv4 and ipv6 DNS servers
Add local DNS as `home.arpa` to point to local ipv4
Go into router settings and change DNS to be your server IP, in my case, `192.168.100.4` and `192.168.100.1`
If you have IPv6, make sure to do the same, in my case `device:ip:address:v6` and `fe80::1`

Now all devices can enjoy PiHole.


---

Need to check how to do TLS in local entrypoint for traefik
