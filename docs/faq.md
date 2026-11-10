# SiteMon FAQ

Back to the [README](../README.md).

## Does SiteMon talk to my network gear?

No. It only reads the cards that FortiMon, UniMon and MerakiMon serve (or push), so it needs no vendor key.

## Pull or push?

Pull when SiteMon can reach the app's address: you add the app and its card key in SiteMon. Push when it cannot: SiteMon makes
an incoming key for that app, and the app's **Send to SiteMon** setting sends its card there. See the [setup guide](setup.md).

## What can a card key do?

Read the card and nothing else: it cannot sign in, change settings or read any other page. Keys are shown once and stored only
as a hash in the app that issues them, and can be revoked.

## An app shows "key refused"

The app rejected the key. It was revoked or mistyped: make a new card key in the app (**Settings > API keys**) and use
**Re-key** on the app in SiteMon.

## An app shows "offline"

For a pulled app, SiteMon cannot reach its address, or the app's certificate changed from the one you trusted (SiteMon then does
not send the key). For a pushed app, nothing arrived within 3 of its intervals. Use **Test connection** on the app. Its last data
stays on the wallboard, dimmed, with "last seen".

## An app shows "needs SiteMon update"

The app sends a card with a newer major version than this SiteMon understands. Update SiteMon (**Settings > Updates**).

## A pushing app says "CERTIFICATE_VERIFY_FAILED"

SiteMon has a self-signed certificate. In the app, **Settings > API keys > Send to SiteMon > Trust its certificate...** shows
the fingerprint to compare with SiteMon's own. From then on the card and key are sent only to a server that shows exactly that
certificate, and stop if it changes. This needs the vendor app at 1.11.3 (FortiMon), 0.1.3 (MerakiMon) or 0.1.2 (UniMon) or
newer.

## Can I put the wallboard on a TV?

Yes: open `/wall`, optionally with filters in the address (`#status=critical`), or make a wallboard key under
**Settings > API keys** for a screen nobody signs in to.

## Has it been tried with real apps side by side?

Not yet. It is verified against simulated apps and tests. Real use with all three apps, a push app going offline live, email
and Teams destinations and a rollback with a broken release are still to be checked.

## I forgot the admin password

On the server itself, run `--reset-password admin` (see [configuration](configuration.md#users-and-roles)).

## How do I update?

**Settings > Updates** checks once a day, and a service install can use **Update now**. Otherwise run the newer installer over
the old one: data and settings are kept.
