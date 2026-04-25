# VideoPlayer-Polyphase-Demo

A small Polyphase Engine project that demonstrates the `com.polyphase.formats.video`
native addon. The addon itself lives in `Packages/com.polyphase.formats.video/` as a
**git submodule** so it can be developed and versioned independently of this demo.

# Cloning

The submodule is required — without it the engine starts up but discovers zero native
addons and the demo scene falls back to a `Node3D` placeholder for the `VideoPlayer`
node. Always clone with `--recurse-submodules`:

```bash
git clone --recurse-submodules <demo-repo-url> VideoPlayer-Polyphase-Demo
```

If you already cloned without `--recurse-submodules`:

```bash
cd VideoPlayer-Polyphase-Demo
git submodule update --init --recursive
```

## Updating the submodule

Pull the latest pinned submodule SHA recorded by this repo:

```bash
git pull
git submodule update --init --recursive
```

Move the submodule forward to the latest commit on its `main` branch and record the
new pin in this repo:

```bash
git submodule update --remote Packages/com.polyphase.formats.video
git add Packages/com.polyphase.formats.video
git commit -m "Bump video addon submodule"
```

## Fixing a broken submodule

If the engine logs `Discovered 0 native addons` or `Failed to construct node 'VideoPlayer'`,
the submodule's working tree is almost always the cause. Diagnose from this repo's root:

```bash
git submodule status
```

Read the leading character of each line:

| Prefix | Meaning | Fix |
|---|---|---|
| `-` | Submodule not initialized (folder empty / only `.git`) | `git submodule update --init --recursive` |
| `+` | Checked-out SHA differs from the SHA pinned in this repo | `git submodule update --recursive` (resets to pinned SHA) |
| `U` | Merge conflict inside the submodule | resolve inside the submodule, then `git add` from this repo |
| (space) | Healthy — submodule matches the pin | nothing to do |

If `Packages/com.polyphase.formats.video/` contains only a `.git` entry (no source
files), or if `git status` inside that folder shows every file staged for deletion,
restore the working tree:

```bash
cd Packages/com.polyphase.formats.video
git fetch origin
git reset --hard <desired-sha>     # e.g. origin/main, or a specific SHA
```

If the `<desired-sha>` is not on `origin` (`git cat-file -t <sha>` reports "bad
object" after a fetch), it was never pushed from whichever machine produced it —
push it from there first; do not guess a different SHA.

# Pre-requisites

## Windows: FFmpeg setup (required for video playback)

The addon's `External/ffmpeg/` folder is intentionally **not** version-controlled
(see `Packages/com.polyphase.formats.video/External/.gitignore`) — FFmpeg
binaries are user-supplied so each project ships with its own LGPL-clean build.

Without FFmpeg in place, the addon DLL fails to link and the engine has no
`VideoPlayer` node type to register. Symptoms in the editor log:

```
Build failed: ... cannot open file 'avformat.lib'
Failed to construct node 'VideoPlayer' (type=2428470202, unknown type?), using Node3D placeholder.
```

Follow the **"Setting up FFmpeg (Windows)"** section in
`Packages/com.polyphase.formats.video/README.md`. In short:

1. Download `ffmpeg-release-full-shared.7z` from
   <https://www.gyan.dev/ffmpeg/builds/>.
2. Extract it. Copy the `include/`, `lib/`, and `bin/` directories into
   `Packages/com.polyphase.formats.video/External/ffmpeg/` so the layout matches:

   ```
   Packages/com.polyphase.formats.video/
     External/
       ffmpeg/
         include/
         lib/
         bin/
   ```
3. Rebuild the addon (open the project; the editor's addon manager auto-builds
   on first load, or use the Addons window's **Build** button to re-trigger).
4. Confirm `Intermediate/Plugins/com.polyphase.formats.video/<hash>/com.polyphase.formats.video.dll`
   is produced and the FFmpeg DLLs (`avcodec-62.dll`, `avformat-62.dll`, etc.)
   land next to it via the addon's `copyBinaries` step.

Use the LGPL "shared" build only — don't use FFmpeg builds configured with
`--enable-gpl` or `--enable-nonfree`.

## Linux

The addon picks up FFmpeg via `pkg-config`, so the manual copy step is not
needed. Install the system packages:

```bash
sudo apt install libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libswresample-dev
```

Then open the project; the addon builds automatically on first engine load.
