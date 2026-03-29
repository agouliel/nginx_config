# Instructions
1. `brew install nginx` (or `apt install nginx`)
2. `cd /opt/homebrew/etc/nginx` (or `cd /etc/nginx`)
3. `git init`
4. `git remote add origin git@github.com:agouliel/nginx_config.git`
5. `git fetch`
6. `git checkout -f main`

# Description
Two distinct architectural approaches for remote access. The first uses a traditional **Reverse Proxy with Port Forwarding**, while the second uses a modern **Zero Trust Tunnel**.

In the first setup, the router acts as a gatekeeper. When a request hits the public IP (managed dynamically by `no-ip.com`) on port 443, the router "forwards" it to the Mac mini. The `nginx.conf` then looks at the `Host` header to decide which `server` block to use, and then it uses the `location` (path) to decide which `proxy_pass` to trigger. This means no third-party interception and no content restrictions, but it also means security risks and configuration overhead (Let's Encrypt renewals).

In the second solution, the `cloudflared` utility on the Mac mini establishes an outbound connection to Cloudflare. Users connect to Cloudflare’s global network, and Cloudflare sends that traffic down the established "tunnel" to the machine.

# Servers
1. `agouliel.ddns.net:80`: redirect any requests to the same URL but on `https`
2. `agouliel.ddns.net:443` (HTTPS)
3. `www.goulielmos.org:80`: in this case, we are sending unencrypted traffic locally inside the Mac mini (the tunnel itself is already encrypted)
4. `goulielmos.org:443`: this does not exist in `nginx.conf`, only in Cloudflare dashboard. It forwards to `https://localhost` and requires the `"Origin Server Name"` setting to be `"agouliel.ddns.net"`. This setting tells the tunnel: "Even though the user typed `goulielmos.org` in their browser, when you connect to the Mac mini, tell Nginx that you are looking for `agouliel.ddns.net`." This "tricks" Nginx into matching the request to the existing HTTPS server block. Nginx is configured with `ssl_certificate fullchain.pem` which is tied to the `agouliel.ddns.net` domain.
