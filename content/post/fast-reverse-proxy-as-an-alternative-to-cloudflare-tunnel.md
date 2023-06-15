+++
date = "2023-06-15T20:27:30+02:00"
title = "Fast Reverse Proxy as an alternative to Cloudflare Tunnel"
draft = false

+++

[Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) is a wonderful (free) product that provides you with a secure way to connect your private (home) server(s) to the internet. This allows you to expose your homelab server to the internet without the need to open any ports on your home network or reveal your home IP address.

While Cloudflare Tunnel works great giving you automatic SSL and all other goodies Cloudflare comes with, it sadly also operates as a man-in-the-middle, routing all traffic through Cloudflare's servers. This setup raises potential concerns about privacy and data security, as the unencrypted traffic could theoretically be accessed by Cloudflare.

For those seeking an alternative solution, [Fast reverse proxy](https://github.com/fatedier/frp) (FRP) is an open-source project that can be self-hosted to replace Cloudflare Tunnel. If I had known about FRP earlier, I would have chosen it from the beginning. FRP offers a different approach by eliminating the need for an intermediary like Cloudflare, allowing you to have direct control over the traffic flow. This grants you the ability to ensure end-to-end encryption, thus mitigating potential security concerns.

If you decide to self-host FRP and replace a Cloudflare Tunnel-like product, you'll need a [publicly available server](https://m.do.co/c/eb6358832805) running FRP as a starting point. The server's configuration might resemble the following:
```ini
[common]
bind_port = 7000
authentication_method = token
token = <generatedToken>
```
This configuration sets FRP to listen on port 7000, enabling communication between the private server and the FRP instance on the public server.

On the client side, the configuration would look like this:
```ini
[common]
server_addr = <ipOfPublicServer>
server_port = 7000
token = <generatedToken>

[serviceName]
type = tcp
local_ip = 172.17.0.1
local_port = 8080
remote_port = 80
use_encryption = true
```
With this setup, the client will connect to the publicly available server, exposing the service running on the local port `8080` of the client server to the public server on port `80`. In simpler terms, it can be described as a glorified version of an automatic SSH tunnel.

Another option to consider instead of FRP is [rathole](https://github.com/rapiz1/rathole), which positions itself as a faster alternative. Although rathole is relatively new, it presents an intriguing solution worth exploring.

After configuring FRP, the next step is to add a reverse proxy such as [Caddy](https://caddyserver.com/) to handle automatic SSL certifications. While FRP takes care of the traffic routing and encryption between the public and private server, a reverse proxy like Caddy can provide an additional layer of security by automatically obtaining and managing SSL certificates for your services.