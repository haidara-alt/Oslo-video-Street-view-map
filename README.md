Oslo Street Archive
A personal Street-View clone: a map of Oslo with your filmed streets drawn in
blue. Click anywhere on a blue line and the matching YouTube video opens,
already scrubbed to the right second.
How the sync works
Your video filename (`VID_20260722_133255_00_076`) encodes local Oslo time:
13:32:55. Your GPX file's first trackpoint is timestamped `11:32:55Z` (UTC).
13:32:55 CEST = 11:32:55 UTC — they're the same instant. That means the
first GPX point ≈ second 0 of the video, and every later point's offset from
that first timestamp is exactly the second to seek to in the video.
The app never even needs to parse the filename — it just uses each GPX
track's own first timestamp as t=0. The filename only matters as the shared
name that ties a `.gpx` to its recording, which is why the code expects
`gpx/VID_..._076.gpx` to match `"filename": "VID_..._076"` in the manifest.
If playback and GPS ever drift apart a little (e.g. GPS took a second or two
to lock at the start), nudge `offsetAdjustSeconds` per video in
`manifest.json` — positive to skip ahead, negative to start earlier.
Files
```
index.html        the whole app (map + player)
manifest.json      list of your videos: name, YouTube ID, path to its GPX
gpx/*.gpx           one GPX file per video, same basename
```
Adding a new filmed street
Upload the new Insta360 video to YouTube (as a 360 video — YouTube will
auto-detect it from the metadata Insta360 Studio embeds, and the embedded
player will give viewers the drag-to-look-around 360 controls for free).
Copy its video ID — the part after `watch?v=` in the URL.
Drop the matching `.gpx` file into `gpx/`.
Add one entry to `manifest.json`:
```json
{
  "name": "Whatever street this is",
  "filename": "VID_20260722_140310_00_077",
  "youtubeId": "dQw4w9WgXcQ",
  "gpx": "gpx/VID_20260722_140310_00_077.gpx",
  "offsetAdjustSeconds": 0
}
```
No code changes needed — the map picks up every entry in the manifest.
Hosting it on GitHub Pages
Create a repo (can be public or private with GitHub Pro/org for Pages).
Push these three items (`index.html`, `manifest.json`, `gpx/`) to the repo root.
Repo Settings → Pages → Deploy from branch → `main` / root.
Your map is live at `https://<username>.github.io/<repo>/`.
Important: don't just double-click `index.html` locally — browsers block
`fetch()` of local files for security, so the manifest/GPX won't load. Either
test with a tiny local server (`python3 -m http.server` in the folder, then
open `http://localhost:8000`) or just push to GitHub Pages and test live.
Notes / things you can extend later
Right now every filmed street is the same blue. If you want to color-code
status (e.g. filmed vs. planned vs. needs reshoot), add a `"status"` field
per manifest entry and branch the polyline color on it.
The nearest-point search is a simple linear scan — completely fine at
personal-project scale (hundreds of points per street), would need a
spatial index if you ever film an entire city.
The nearest-point match is straight-line distance to trackpoints, so very
sparse GPX tracks (big gaps between points, like the ~25s gap at the very
start of your sample file) will feel slightly less precise right where the
gap is. More frequent GPS logging = smoother matching.
