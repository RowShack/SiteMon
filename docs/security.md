# Running SiteMon safely

Back to the [README](../README.md).

SiteMon holds the card keys of the apps it reads and shows the state of every site, so treat its web page like an admin
console. This page says how to run it so that only the people, and the apps, you mean to can reach it.

## Best: keep it off the public internet

Reach SiteMon over a VPN or a tailnet (WireGuard, Tailscale **without** Funnel, or your firewall's own VPN). Leave
`bind` on `127.0.0.1` (the default) or bind to the VPN address only, and keep the port closed to the internet. Clients
then arrive with their own addresses, so sign-in lockout and the audit log work as intended. Apps that send their card
(push) can use the same VPN or tailnet.

## If you publish it anyway

1. **Put an identity-aware gate in front** (a reverse proxy or access service that signs people in before a request
   reaches SiteMon), ideally on a different machine. Apps that push their card can't sign in to such a gate, so let only
   `POST /api/card/v1` through it without sign-in (SiteMon checks the app's key itself, before it reads anything).
2. **Tell SiteMon which proxy to believe.** If the proxy runs on the same server (nginx, Caddy, a tunnel agent), add its
   address to `web.trusted_proxies` in `config.json` (or `trusted_proxies` on `POST /api/admin/web`) and make the proxy send
   `X-Forwarded-For` and `X-Forwarded-Proto`. Without this, every visitor looks like the proxy: a few wrong passwords from
   anyone can lock that one address out for 15 minutes. Forwarded headers from anyone not listed are ignored.
3. **Two-step sign-in for everyone.** Settings > Security > two-step sign-in required, and enrol every account.
   A server set up from another computer or listening on the network starts with it required for admins.
   Use long random passwords: an admin can download the apps' card keys in a backup (the backup is protected by a
   password you choose, and asks for your password again).
4. **Finish the first-run setup yourself** before the port is reachable, and check Settings > Audit log for
   `login.failed` and `login.blocked`.

## Keys between SiteMon and the apps

- **Reading an app (pull).** The key is a card key made in that app; it can read the app's card and nothing else. SiteMon
  stores it encrypted, never shows it again and sends it only to the address you saved: change the address and SiteMon
  asks for the key again. For an https app SiteMon remembers the app's certificate when you test the connection and
  stops, without sending the key, if another certificate turns up. Plain http has to be ticked; avoid it outside a VPN.
- **An app that sends its card (push).** SiteMon makes one incoming key per app and shows it once. The key can only send
  that one app's card: it can't sign in, read the dashboard or speak for another app. Replace or revoke it under
  Settings > Apps; the old key stops at once. A key is limited to one card every 5 seconds, 1 MB and 2000 sites.
- **Admins choose the addresses.** An app's address can be anything the server can reach, because apps are usually on
  your own network. Give the admin role only to people you would let reach those addresses from the server.

## What changed in 0.2.0

These were found by a security review of FortiMon, whose web server and sign-in SiteMon shares, and were fixed for SiteMon
in 0.2.0 (see the [changelog](https://github.com/RowShack/SiteMon/releases)):

- Nothing beyond 4 KB is read from someone who isn't signed in, and requests with `Transfer-Encoding` or two
  `Content-Length` headers are refused. A card without a valid key is refused before it is read.
- The first-run setup code is longer and limited to 5 wrong tries per address; setup without the code works only from the
  server itself.
- Sign-in lockout counts per address, per address and user name, and per user name, and the real owner can still get in
  from an address that signed them in recently.
- At most 256 connections (64 per address) and a 30-second request timeout; Linux service limit `LimitNOFILE=4096`.
- `Strict-Transport-Security` is sent over HTTPS.
- Backup with the card keys, restoring users, making an admin, resetting an admin's password or two-step sign-in, and
  uploading a certificate ask for the password again (and an authenticator code if you use one).
- A card with more than 2000 sites is refused, so one app's key can't fill SiteMon's history.

## What was checked, and what was not

The security review itself was of FortiMon. For SiteMon the shared parts were re-tested on a running SiteMon test
server on localhost (setup through a proxy with and without `trusted_proxies`, lockout, body and header limits, request
framing, connection caps, the password prompts), the app and key routes were checked to be admin only, and the incoming
card endpoint was checked: no key or a wrong key is refused before the body is read, a key speaks for its own app only,
and a revoked key stops at once. Automated tests cover each fix.

It did **not** cover: behaviour behind a real reverse proxy or tunnel (only simulated headers), exhaustion at large scale,
real apps side by side on a network (stub apps were used), the Windows service end to end, or any automated code-scanning
tool. Wrong card keys are not rate limited per address (the keys are too long to guess). The Linux service has no memory
limit or syscall filter, and the `LimitNOFILE` setting reaches an existing Linux install only when you run the `.run`
installer again. Even with these fixes, exposing the port to everyone is not recommended.

Found a problem? Please report it privately: see [SECURITY.md](../SECURITY.md).
