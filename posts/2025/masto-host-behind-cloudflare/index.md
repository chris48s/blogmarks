<!--
.. title: Running masto.host behind CloudFlare
.. slug: masto-host-behind-cloudflare
.. date: 2025-08-17 00:00:00
.. tags: mastodon
.. category: 
.. link: 
.. description: 
.. type: text
-->

My mastodon instance is managed by [masto.host](https://masto.host/) and I use CloudFlare for DNS. If you use this combination of services and run "Check DNS" in the masto.host dashboard, it will tell you

> NOTICE: CloudFlare Proxy needs to be disabled.
> The Proxy Status for that record should read 'DNS Only'.
> If you see the 'orange cloud' next to the record then Proxy is enabled and you need to disable it.

This is a working configuration, but running in DNS Only mode means there is no cache/CDN in front of your instance. It also means a lot of useful CloudFlare features are not available to you as they only work when CloudFlare is acting as a proxy.

Although masto.host tells you to set CloudFlare to DNS Only, they do this to reduce support overhead for CloudFlare users. That is understandable, but it is perfectly possible to run CloudFlare in proxy mode. In my case, I needed to do two things:

1. Ensure the TLS mode is set to [Full (strict)](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/). I want this anyway because my origin has its own TLS cert, but Flexible mode will cause a [redirect loop](https://developers.cloudflare.com/ssl/troubleshooting/too-many-redirects/) because the upstream server redirects http:// requests to https://.
2. Turn off [Always Use HTTPS](https://developers.cloudflare.com/ssl/edge-certificates/additional-options/always-use-https/). This is needed so that the origin's TLS certificate can be [renewed by Let's Encrypt Bot](https://community.cloudflare.com/t/let-lets-encrypt-bot-bypass-always-use-https/200274).
