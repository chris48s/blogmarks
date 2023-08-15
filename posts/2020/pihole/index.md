<!--
.. title: Pi-Hole
.. slug: pihole
.. date: 2020-09-29 00:00:00
.. tags: raspberry-pi
.. category: 
.. link: 
.. description: 
.. type: text
-->

I've been running [Pi-Hole](https://pi-hole.net/) for years and I'm a big fan of it. It provides network-level blocking of ads and trackers. Set up Pi-Hole, configure your network to use it as a DNS server and it won't resolve DNS requests for known ad/tracking domains. This is harder to detect than in-browser content blockers and also blocks ads on mobile platforms where ad blockers may not be available because the blocking is applied at the network level, rather than the device.

Pi-Hole can run on a Raspberry Pi (hence the name) - and that's how I run mine - but can also be installed on other hardware (including via Docker). It can also be [configured as a DNS-Over-HTTPS proxy](https://docs.pi-hole.net/guides/dns-over-https/) using [Argo Tunnel](https://github.com/cloudflare/cloudflared) so you can use DOH and have DNS-level ad-blocking.
