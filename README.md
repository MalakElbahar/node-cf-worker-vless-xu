# Node Cloudflare Worker for VLESS — Secure Fast Tunnel Proxy

[![Releases](https://img.shields.io/badge/Releases-via%20GitHub-blue?logo=github)](https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases) ⚡️🔒☁️

<img alt="Cloudflare Workers + Node + VLESS" src="https://www.cloudflare.com/img/learning/cloudflare-fundamentals/cloudflare-workers/Cloudflare-Workers-Hero.svg" width="80%">

Table of contents
- About
- Features
- Architecture diagram
- Quick links
- Prerequisites
- Install and run
- Configuration
- Deploy to Cloudflare Workers
- Run locally
- Example clients
- Security and best practices
- Troubleshooting
- FAQ
- Contributing
- License

About
- node-cf-worker-vless-xu provides a Node-based template and worker code to expose a VLESS-compatible tunnel via Cloudflare Workers.
- It aims to deliver a low-latency, privacy-aware proxy that uses Cloudflare Workers as the edge layer and a VLESS backend for encryption and multiplexing.
- Use the GitHub Releases page to get the packaged worker and helper scripts: https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases

Features
- VLESS transport handling for modern clients
- Light Node tooling to generate worker code and config
- Wrangler-ready worker script for Cloudflare Workers
- Simple mapping between VLESS connections and Cloudflare worker fetch handlers
- TLS-on-edge via Cloudflare
- Small footprint for low overhead and cost control

Architecture diagram
![Architecture diagram](https://upload.wikimedia.org/wikipedia/commons/3/33/Cloudflare_logo.svg)

- Client (VLESS) <--> Cloudflare Worker (edge) <--> Backend node/vless proxy server
- The worker accepts websocket or fetch requests, decrypts, and forwards to the backend.
- The backend runs VLESS server logic and serves outbound traffic.

Quick links
- Releases (download and run asset): https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases
- Use the Releases button above or the link to download the packaged worker file. The release contains the worker script and helper runner. Download the asset and execute the provided script to generate keys and deploy.

Prerequisites
- Node.js 16+ (for local tools and build)
- npm or yarn
- wrangler CLI (for Cloudflare Workers)
- A Cloudflare account with Workers enabled
- A VLESS-capable server or a backend that accepts proxied traffic

Install and run (developer flow)
1. Clone this repo
   git clone https://github.com/MalakElbahar/node-cf-worker-vless-xu.git
   cd node-cf-worker-vless-xu

2. Install dependencies
   npm install

3. Build the worker
   npm run build

4. Deploy (see Deploy to Cloudflare Workers section)

Releases and packaged files
- Visit the Releases page to get the packaged worker and helper scripts:
  https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases
- The Releases page includes a compressed archive such as node-cf-worker-vless-xu-v1.0.0.zip or a direct worker.js file.
- Download the asset from the release and then run the supplied script. For example:
  - Extract the archive.
  - Run node setup.js to generate configuration and keys.
  - Run node worker.js to test locally.
- The bundled script contains a ready-to-publish worker and a small Node helper for local testing.

Configuration
- The project uses a JSON config file named config.json.
- Sample config.json
  {
    "vless": {
      "id": "uuid-here",
      "flow": "xtls-rprx-direct",
      "address": "127.0.0.1",
      "port": 12345
    },
    "worker": {
      "route": "/vless",
      "timeoutMs": 30000,
      "maxConns": 1000
    }
  }
- Set a strong UUID for vless.id. Use a UUID generator or the script included in Releases.
- worker.route is the worker path the client will hit. Match it with the client config.

Deploy to Cloudflare Workers
- Use wrangler to publish the worker. Example:
  npx wrangler login
  npx wrangler init --site --name node-cf-worker-vless-xu
  npx wrangler publish dist/worker.js --name vless-proxy-worker

- The release file includes a worker script optimized for Workers. Download it via the Releases page and deploy it with wrangler:
  - Download the worker asset from https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases
  - Place worker.js in your project root.
  - Run npx wrangler publish worker.js

Run locally
- The repo provides a small local server that simulates the worker environment. Use it to test flows before publishing.
- Example:
  node local-server.js --config config.json
- The local server mirrors the Cloudflare fetch interface and logs connections and metadata.

Example client setups
- VLESS clients need to point to your Cloudflare Worker domain and the route path.
- Example V2Ray client JSON snippet:
  {
    "protocol": "vless",
    "settings": {
      "vnext": [
        {
          "address": "your-worker.example.workers.dev",
          "port": 443,
          "users": [
            {
              "id": "uuid-here",
              "flow": "xtls-rprx-direct"
            }
          ]
        }
      ]
    },
    "streamSettings": {
      "network": "ws",
      "wsSettings": {
        "path": "/vless"
      },
      "security": "tls"
    }
  }
- Replace uuid-here with the id in config.json.

Security and best practices
- Keep vless IDs secret. Treat them as credentials.
- Rotate keys if you suspect compromise.
- Use Cloudflare account access controls and audit logs.
- Limit worker route exposure by using Cloudflare Access or firewall rules if needed.
- Monitor worker usage to stay within free tier limits or to control costs.

Performance tips
- Keep worker logic minimal. Offload heavy work to backend servers.
- Use proper caching and connection reuse if you proxy HTTP flows.
- Limit concurrent connections in config.worker.maxConns to match backend capacity.

Troubleshooting
- Worker fails on publish
  - Confirm wrangler version and Node version.
  - Check dist/worker.js exists and is valid JS.
- Connections drop
  - Increase timeoutMs in config.json.
  - Check backend reachability and firewall rules.
- Client fails to authenticate
  - Ensure UUID in client config matches config.json.
  - Check the route path and TLS settings.

FAQ
- Q: Do I need to run the Node server in cloud?
  - A: The worker runs at the edge. You need a backend VLESS server reachable by the worker. That server can run anywhere with stable network access.
- Q: Can I use this without Cloudflare?
  - A: The worker code targets Cloudflare Workers. You can adapt the core logic to other edge platforms, but you must handle their API differences.
- Q: Which clients work?
  - A: Any client that supports VLESS and websocket/tls transport should work when configured to hit the worker route.

Contributing
- Open an issue for bugs or feature requests.
- Fork the repo, make a branch, and submit a pull request.
- Keep changes small and focused.
- Run tests before opening PRs.

Testing
- Unit tests live in tests/.
- Run tests:
  npm test

Changelog
- See Releases for packaged builds and changelogs:
  https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases
- Each release contains release notes and the packaged worker file. Download the version that matches your target environment and follow the included README in the release asset.

Examples and snippets
- Minimal worker handler (excerpt)
  addEventListener('fetch', event => {
    event.respondWith(handle(event.request))
  })
  async function handle(req) {
    // parse vless frame or websocket handshake
    // forward to backend
    return new Response('OK')
  }

- Local test script
  node local-test.js --target http://127.0.0.1:12345 --path /vless

Images and badges
- Badges use Shields for simple status and links.
  [![Releases](https://img.shields.io/badge/Releases-via%20GitHub-blue?logo=github)](https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases)

- Logo assets
  - Cloudflare: https://www.cloudflare.com/img/learning/cloudflare-fundamentals/cloudflare-workers/Cloudflare-Workers-Hero.svg
  - Node: https://raw.githubusercontent.com/github/explore/main/topics/nodejs/nodejs.png

License
- This project uses the MIT license. See LICENSE for details.

Contact
- Open issues on GitHub for bugs, questions, or support.
- Use the Releases page to fetch the packaged files and run them as instructed:
  https://github.com/MalakElbahar/node-cf-worker-vless-xu/releases

Thanks for checking the repo.