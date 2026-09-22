# Porch

A private one-to-one video call in the browser. No accounts, no install, no server of your own. One person opens the page, sends the link, the other opens it, and the call connects.

## How it works

- **Media** — WebRTC, browser to browser. Audio and video never touch a server; they go directly between the two machines and are encrypted end to end by the protocol.
- **Signaling** — the free public [PeerJS](https://peerjs.com) broker relays the connection handshake (SDP offers, ICE candidates). It sees who is connecting, not what you say.
- **NAT traversal** — Google's public STUN servers only. For two homes on ordinary residential internet this connects directly the large majority of the time. There is no TURN relay configured, so a call from a corporate network or over a VPN may fail to connect (see below).

Whoever opens the room first claims the "host" ID on the broker. The second person's registration is rejected as a duplicate, which is the signal to flip to "guest" and dial the host. The guest re-dials every four seconds until answered, so it doesn't matter who arrives first.

## Deploy it

### GitHub + Cloudflare Pages

1. Create a new repo on GitHub (public or private, either works) and push these two files to it:

   ```bash
   git init
   git add index.html README.md
   git commit -m "Porch: two-person video call"
   git branch -M main
   git remote add origin git@github.com:YOUR-USERNAME/porch.git
   git push -u origin main
   ```

2. Go to **dash.cloudflare.com → Workers & Pages → Create → Pages → Connect to Git**, pick the repo.
3. Leave the build command empty and set the output directory to `/` (it's a static file, there's nothing to build).
4. Deploy. You get `https://porch-xyz.pages.dev`.

### Or GitHub Pages

Push the repo, then **Settings → Pages → Source: Deploy from a branch → main / (root)**. You get `https://YOUR-USERNAME.github.io/porch/`.

Either host gives you HTTPS automatically, which is mandatory — browsers refuse camera access on plain HTTP.

### Custom domain

Optional. Both hosts let you point a domain at the deployment from their dashboard; Cloudflare's is one dropdown if the domain is already on Cloudflare.

## Using it

1. Open the URL, click **Start a call**, allow camera and microphone.
2. Click **Copy link** and send it to the other person however you like.
3. They open it and click **Join call**. Connection takes a few seconds.

The room code lives in the URL fragment (`#abc12xyz`), which never leaves the browser — it isn't sent to the web server in the HTTP request. Anyone with the link can join, so treat the link as the password. Codes are 8 random characters from a 31-letter alphabet, about 40 bits, which is far beyond guessable for casual use.

## Known rough edges

**Refreshing as the host.** If the host reloads the page, the broker may still be holding the old host ID for a few seconds. The reloaded tab sees the ID as taken, becomes a guest, and dials a host that no longer exists. Wait about 10 seconds and reload again, or have both people reload.

**The public broker.** PeerJS's free cloud broker is a shared service with no uptime guarantee. If calls suddenly stop connecting and everything else looks fine, that's the first suspect. Running your own is about 40 lines of Node (`peer` npm package) on Fly.io or a small VPS, and you'd then pass `{ host, port, path, secure: true }` into the `new Peer()` options.

**No TURN.** By design. If you ever need it to work from an office, a hotel, or over a VPN, sign up for a free TURN account (Metered's Open Relay gives 20 GB/month, enough for roughly 20 hours of relayed calling) and add the servers to the `ICE` object at the top of the script in `index.html`. The commented-out line shows the shape.

**Safari.** Works, but it re-asks for camera permission more often than Chrome and is stricter about autoplay. The `playsinline` attributes and the `play()` nudge in the stream handler are there for it.

**Two people only.** The host/guest ID scheme is deliberately two-slot. A third person joining an occupied room is told the call is full.

## Changing things

Everything is in `index.html` — no build step, no dependencies beyond the PeerJS script tag. Edit, commit, push; the host redeploys on its own.
