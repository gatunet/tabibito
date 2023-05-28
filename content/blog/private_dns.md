+++
title = "Private DNS"
date = "2023-05-06"
+++

Maybe the point of less privacy for people in the internet is the fact that DNS isn't encrypted ny default, so your ISP can see what you search for, even if you don't use their DNS service(you shoudn't), so how do we fix that? well, if you use Firefox and doesn't want to mess with complicated setups, you can just enable DoH(DNS over HTTPS) in the configs, you'll be limited to only a couple, but I would say that **Cloudflare** is pretty ok.

Now if you're here for the complicated(but **A LOT** better setup) then here we go


## DNS Provider

The first thing you need to do is chose a provider, here are a couple of good ones:

- [9.9.9.9 - Quad9](https://www.quad9.net)
- [1.1.1.1 - Cloudflare](https://www.cloudflare.com/en-gb/learning/dns/what-is-1.1.1.1/)

For me, I'm going with **Quad9**, because not only they're a solid option, but I think they're really serious about privacy, if you want to know more you can go to their page or [this video](https://www.youtube.com/watch?v=a2RjbvMES-0).


## PiHole / DoH (Dns Over Https)

Now, this is "kind of" unecessary, but is a very nice to have in the setup, so I'm going to use it, if you don't want it, you can change the cloudflared service a bit to have it doing all the work.

For **PiHole** we're going to use the [pihole docker](https://hub.docker.com/r/pihole/pihole), with a *docker-compose* like the one bellow, and for the DoH part we'll use a container made by cloudflare that gives us a tool for using a couple of their services, one of them is to redirect our dns queries over DoH, for all of that we'll need a **docker-compose** like this:

Also as you can see, the tools made by cloudflare can accept a lot of upstream services, not only those made by cloudflare, so if you want you can even use googles DNS there.
```yaml
version: "3.8"

services:

  pihole:
    container_name: pihole
    image: pihole/pihole:latest

    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"

    environment:
      TZ: 'America/Chicago'
      WEBPASSWORD: 'admin' # CHANGE LATER !!!
      PIHOLE_DNS: cloudflared # or whathever the name of your other container is

    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'

    cap_add:
      - NET_ADMIN # Required if you are using Pi-hole as your DHCP server, else not needed

    restart: unless-stopped

  cloudflared:
    container_name: cloudflared
    image: cloudflare/cloudflared:latest
    command: proxy-dns --address 0.0.0.0 --port 53 --metrics 127.0.0.1:8080 --upstream https://9.9.9.9/dns-query --upstream https://149.112.112.112/dns-query --max-upstream-conns 0

    restart: unless-stopped
```

About this **command** entry on our *yaml* file, the entrypoint for this docker image is `ENTRYPOINT ["cloudflared", "--no-autoupdate"]`, and with this we just append all those flags into this command, I tried abstracting somethings to environment variables but had no luck with it, so I'm doind the "hard" way, but feel free to try a "better" way, and if you do find, please open an issue on the repo for this blog with your solution.

You should have a single **docker-compose** with both services for the "docker dns" to work and resolve your cloudflared service name to an ip for the pihole service so they can work together.


And after this you should have an ip to your pihole, and you can just either change your router setting to point there for DNS, or you can change in every machine.


## References

- [Quad9 - DNS](https://www.quad9.net)
- [Cloudflare - DNS](https://www.cloudflare.com/en-gb/learning/dns/what-is-1.1.1.1/)
- [Youtube - DNS Privacy](https://www.youtube.com/watch?v=a2RjbvMES-0)
- [DockerHub - PiHole container](https://hub.docker.com/r/pihole/pihole)
