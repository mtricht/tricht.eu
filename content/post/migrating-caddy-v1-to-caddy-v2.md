+++ 
draft = false
date = 2021-04-03T13:24:32+02:00
title = "Migrating Caddy v1 to v2 using Caddyfile"
slug = "migrating-caddy-v1-to-v2" 
+++

I like using [Caddy](https://caddyserver.com/) as a HTTPS server with automatic SSL certificates using [Let's Encrypt](https://letsencrypt.org/).
With a few configuration lines you can get up and running in no time.

Recently I switched VPS providers and ended up having to migrate Caddy from v1 to v2. The documentation for Caddy is wonderful and chockful with emojis. The [getting started](https://caddyserver.com/docs/getting-started#your-first-caddyfile) tutorial starts by telling you how a Caddyfile looks like in v2, where my confusion started. As I mostly need reverse proxies, the very first example there looks like this:

```Caddyfile
localhost

reverse_proxy 127.0.0.1:9000
```

which quite frankly looks quite different than what it looked like in v1. No curly brackets. How do I define multiple domains? Well apparently, the above is the equivalent to 

```Caddyfile
localhost {
  reverse_proxy 127.0.0.1:9000
}
```

which is sorta what you're used to! They do mention this somewhere [else](https://caddyserver.com/docs/caddyfile-tutorial#multiple-sites) but it is not mentioned in the [v2 upgrade guide](https://caddyserver.com/docs/v2-upgrade).

So to help you, here's my v1 Caddyfile:

```Caddyfile
subdomain1.tricht.eu {
  proxy / localhost:7777
}
subdomain2.tricht.eu {
  proxy /notifications/hub/negotiate localhost:8081 {
    transparent
  }
  proxy /notifications/hub localhost:3012 {
    websocket
  }
  proxy / localhost:8081 {
    transparent
  }
}
subdomain3.tricht.eu {
  proxy / localhost:8080 {
    transparent
  }
  limits {
    header 64kb
    body 1gb
  }
  header / {
    Strict-Transport-Security "max-age=31536000; includeSubdomains; preload"
    X-XSS-Protection "1; mode=block;"
    X-Content-Type-Options "nosniff"
    X-Frame-Options "SAMEORIGIN"
  }
  gzip {
    ext *
    level 4
  }
}
```

and what it looks like in v2:

```
subdomain1.tricht.eu {
  reverse_proxy localhost:7777
}
subdomain2.tricht.eu {
  reverse_proxy /notifications/hub/negotiate localhost:8081
  reverse_proxy /notifications/hub localhost:3012
  reverse_proxy localhost:8081
}
subdomain3.tricht.eu {
  reverse_proxy localhost:8080
  header {
    Strict-Transport-Security "max-age=31536000; includeSubdomains; preload"
    X-XSS-Protection "1; mode=block;"
    X-Content-Type-Options "nosniff"
    X-Frame-Options "SAMEORIGIN"
  }
  encode gzip
}
```

So not much has changed, except a missing `limits` directive and confusing documentation.
