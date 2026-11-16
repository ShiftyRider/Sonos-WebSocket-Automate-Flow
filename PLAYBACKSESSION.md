## Why `soco-cli` / UPnP Fails on Spotify (`Error 800` & `Error 804`)

Legacy Sonos CLI tools like `soco-cli` communicate over **UPnP/SOAP (`http://<speaker-ip>:1400`)**. While port `1400` still works for basic transport controls (`play`, `pause`, `volume`) and direct audio stream URLs, Spotify deprecated legacy UPnP queue injection on Sonos in favor of Sonos's modern **Cloud Queue / `playbackSession`** architecture over **WebSockets (`wss://<speaker-ip>:1443`)**.

When attempting to queue Spotify URIs or Spotify-backed Sonos Favorites via UPnP, the speaker rejects the request with `80x` UPnP fault codes:

| UPnP Error | Meaning | Why It Happens With Spotify |
| :--- | :--- | :--- |
| **`Error 804`** | *Incompatible URI / Invalid Metadata* | Legacy `AddURIToQueue` expects traditional track URIs (`x-sonos-spotify:...`) accompanied by valid DIDL-Lite XML metadata. Modern Spotify playlists, Daily Mixes, and radio stations use cloud-backed container IDs (`spotify:playlist:...`, `spotify:station:...`) that the legacy UPnP stack cannot resolve. |
| **`Error 800`** | *Not Supported / General Failure* | Spotify streams on Sonos S2 must be initialized as a `playbackSession` (`loadContainer`) rather than pushed into the legacy UPnP queue. When `soco-cli` attempts to extract a raw URI from a Sonos Favorite (`add_favorite_to_queue`) and force-queue it via UPnP, the speaker refuses the operation. |

### The Takeaway: Port `1400` (UPnP) vs. Port `1443` (`wss://`)

* **Port `1400` (`soco-cli` / `curl`):** Best for simple transport commands, grouping, and non-cloud-queue streams.
* **Port `1443` (`websocat` / `x2rock`):** Required to read modern S2 **Recents (`playbackHistory`)** and load Spotify containers (`playbackSession` / `loadContainer`) natively without relying on a Raspberry Pi middleman or cloud OAuth login.
