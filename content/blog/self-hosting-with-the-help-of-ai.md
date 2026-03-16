---
title: "Self Hosting With the Help of Ai"
date: 2026-01-26T21:05:41+02:00
draft: true
---
Ages ago I bought a small HP computer with 16G of RAM, 120G of disk space with the goal of hosting home assistant there. But never found time. Living in suburbs also
meant that the only way to connect to the Internet was using 4G modem which meant I had unreliable connection. There was also a lot of hassle setting up dynamic DNS
so I could easily access my machine remotely. But things changed. I learned about Cloudflare tunnels, Tailscale, and I know have fiber at home.

Some time ago I stumbled upon the [blog post](https://www.contraption.co/how-to-host-web-apps-on-a-mac-mini/) on how to turn your Mac Mini into a self-hosting server.
For ages I used shared hosting, but deploying some custom code there, e.g. MQTT server, was out of the question. Consuming more than 256M of RAM, and your app is killed. Of course
one could sping up an EC2 server on AWS, or migrate to a bigger plan, but since I don't host that many projects and only rarely need something custom, it felt like a
massive overkill.

From the Contraption blog I learned about Cloudflare tunnels. What these do for you, is you set up a secure tunnel between your server and Cloudflare, and then all of
the traffic is routed via Cloudflare network to your home server without ever exposing IP address. End users see Cloudflare's IP address. This completely eliminates the
need for any dynamic DNS service which constantly listens to your public IP changes and then updates DNS settings.

## Setting up Cloudflare tunnel

There are two ways one can configure tunnels: locally using config file or via Clouflare web UI. I chose the latter since this is much simpler and faster.

You should start by installing the `cloudflared` on your machine
