# OpenDecks (working title) — Product Brief

> One-liner: the music app that doesn't care where your music lives. Connect your existing subscriptions, share playlists with anyone on any platform, and let a local AI DJ run the decks.

## Problem

- Music is siloed by platform. Sending a Spotify playlist to an Apple Music user is still a chore in 2026; existing transfer tools (SongShift, TuneMyMusic) are clunky one-off utilities, not a home for your listening.
- "Shuffle" on every platform is dumb random. Nobody's phone can read a room, build energy across a set, or recover when the vibe dips.
- Every AI music feature ships as a subscription upsell because inference runs on someone else's servers.

## Solution

An iOS app (paid, one-time) that:

1. **Connects the subscriptions you already pay for.** Sign in with Spotify and/or Apple Music and get one unified library, one search, one queue. No new music license, no new monthly fee.
2. **Shares playlists across platforms.** Send a playlist to anyone; recipients open it in whatever app they use. Track matching via ISRC/metadata, with graceful fallback when a track is missing on their service.
3. **Puts a real AI DJ on shuffle.** Not random — a set builder that reads energy, tempo, key, and your taste, and picks the next track like a DJ reading a room.

## Why this can win

- **Wedge:** cross-platform playlist sharing. Everyone has felt this pain; it's the feature people send to friends, which is free distribution.
- **Moat:** the DJ. Transfer tools are utilities; a DJ that learns your taste and runs locally is a product people stay for.
- **Economics:** the DJ brain runs on-device (see Architecture). Zero inference server bills means a one-time purchase is sustainable — no other AI music app can price this way.

## How it works (user flow)

1. Open app, connect Spotify and/or Apple Music (OAuth via each platform's SDK).
2. Unified home: your playlists, liked songs, and history from all connected services, deduplicated.
3. Hit **DJ mode**: pick a seed (playlist, artist, mood, or "surprise me"). The DJ builds a live set — you see the upcoming queue and can steer it (more energy, chill out, skip the vibe).
4. **Share** any playlist or live set as a link. Recipient picks their platform; tracks map over automatically.

## Architecture

### Playback: bring your own subscription
- **Apple Music** via MusicKit: full-track playback for subscribers, clean official API.
- **Spotify** via the iOS SDK: playback for Premium subscribers, subject to Spotify Platform Terms.
- **Tidal** later (API exists). **YouTube Music and Amazon Music are out of scope for v1** — no usable official playback APIs.
- The app never touches audio files or licenses anything. It is a playback client + intelligence layer.

### The DJ: System 1 on device, System 2 on device too
- **System 1 — the fast layer (Laya/Jev-style):** a small local decision model that picks the next track in milliseconds. Inputs: current track's energy/tempo/key, set arc so far, user steering, skip/replay feedback. This is DJ intuition — fast, reactive, no round trip.
- **System 2 — the slow layer:** taste profiling, playlist curation, and cross-platform track matching. Runs on-device or during idle/charging; precomputes candidate pools the fast layer chooses from.
- **Why local matters:** latency (a DJ can't wait 2s per decision), privacy (listening history never leaves the phone), and cost (no server inference = one-time pricing works).
- Audio features (tempo, key, energy) from platform APIs where available (Spotify audio features, Apple Music metadata), supplemented by on-device analysis for gaps.

### Sharing
- Playlists serialized as ordered track lists with ISRC + title/artist/album fingerprints.
- Per-platform resolution at open time; report match rate honestly ("47 of 50 tracks found on Apple Music").

## Business model

- **Free:** connect accounts, browse unified library, share playlists, limited DJ sets per day (e.g. 3).
- **Pro — one-time purchase (~$7.99):** unlimited DJ mode, advanced steering, priority matching. No ads anywhere, ever — audio ads would destroy the DJ illusion, which is the whole product.
- No subscription tier at launch. Revisit only if server-side features (social, cloud sync) justify ongoing cost.

## MVP scope (v1)

- iOS only. SwiftUI.
- Spotify + Apple Music connect, unified library + search + queue.
- DJ mode v1: energy-aware next-track picking from a seed, with basic steering (energy up/down) and skip learning.
- Playlist share links with cross-platform resolution.
- Free tier + one-time Pro unlock via App Store.

## Out of scope for v1

- YouTube Music / Amazon Music support, Android, offline downloads, social features, collaborative live sets, desktop.

## Risks

- **Platform terms:** Spotify's Platform Terms restrict commercial use patterns; Apple requires a MusicKit developer token and active subscriber. Legal review of both before building playback.
- **App Store review (4.2 minimum functionality):** a thin wrapper around Spotify/Apple Music gets rejected. The DJ + unified library + sharing must be demonstrably real functionality, not a reskin.
- **Catalog matching:** ISRC coverage is good but not perfect; obscure tracks and regional licensing will miss. UX must handle misses gracefully.
- **Scope creep:** "connect everything" is tempting; discipline is shipping two platforms well.

## Roadmap

- **Phase 0:** Legal/terms check on Spotify + Apple Music SDKs. Spike: on-device next-track picker prototype.
- **Phase 1 (MVP):** as scoped above. TestFlight with friends.
- **Phase 2:** Tidal, steering v2 (mood targets, "recover the room"), share-link web fallback.
- **Phase 3:** Android, collaborative sets, open-source the client with paid store builds.

## Open questions

- Working title — OpenDecks is a placeholder.
- Exact free-tier DJ limit (tune from TestFlight data).
- Whether the System-1 picker starts as heuristics + learning, or a trained small model from day one.
