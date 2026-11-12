# Sonos-WebSocket-Automate-Flow

Zero-Interruption Audio Switching (Android TV video sources + Sonos music)


The UI Problem

On Android TV, switching music while watching a live video stream normally breaks the viewing experience. Let's instead command Spotify and Sonos while hidden in the background.

The Architecture: Overlay UI + Local WebSocket (websocat)

Instead of switching foreground apps, this workflow keeps video uninterrupted by separating the selection UI from the playback transport:
1. Non-Intrusive Overlay: The sideloaded LlamaLab Automate app overlays a dialog briefly without pausing live video playback.
2. Dynamic Recent History: Automate queries the Sonos speaker's local WebSocket API for recent Spotify items.
3. Direct Speaker Handshake: Once an item is selected, Automate goes behind-the-scenes to pipe a lightweight JSON payload directly to the speaker's local WebSocket endpoint.

Why websocat Over Full CLI Wrappers

• Self-Contained on Android TV: A single static binary eliminates the need for an external Linux host, Python environment, or companion server.
• Protocol Precision: Talking directly to Sonos's local S2 WebSocket API bypasses legacy UPnP container restrictions like Error 804.
