# Changelog

## 2026-10-29 — DD2 movie playback fixed in DWProton + FFmpeg 8.1.3

### Fixed

- Dragon's Dogma 2 no longer crashes during the opening movie with:

  ```text
  Failed to create SourceReader with registered byte stream handler.
  error(c00d36bb)
  ```

- Root cause: DWProton's FFmpeg build used `--enable-libvpx`, so `libavcodec.so.62` required `libvpx.so.9`. The Proton runtime only ships `libvpx.so.6`. The mismatch prevented the media stack from loading the codec library, which surfaced as a Media Foundation SourceReader failure.
- Removed `--enable-libvpx` from the FFmpeg configure arguments and rebuilt FFmpeg 8.1.3. Native VP8/VP9 decoding remains available.

### Added

- Backported the GE-Proton 11-6 media rework onto a Valve Wine 11.0 tree:
  - Full `winedmo` media backend with demuxer, transform, decoder, and muxer support
  - 26 `winedmo` exports, matching the working GE-Proton 11-6 runtime
  - SourceReader / media session / Quartz / qasf / wmvcore fixes from the GE patch series
- Added Python 3.9 compatibility for the Proton runtime scripts:
  - `from __future__ import annotations` in `proton`, `utilities.py`, `vulkan.py`, and `protonfixes/upscalers.py`
  - Fallback for `typing.Self`, which is unavailable in Python 3.9
- Added a launcher for the custom build (`scripts/steamit-dd2-custom`).
- Added the combined Wine media-rework patch and the DWProton FFmpeg patch.

### Validation

- Clean `redist` build completed successfully.
- Tested with the custom-built Wine, `winedmo`, and FFmpeg 8.1.3 — no GE binaries swapped in.
- Dragon's Dogma 2 reached shader compilation, played the H.264 3840x2160 60 fps opening movie, and continued into the intro without the SourceReader crash.
- Verified the final `libavcodec.so.62.28.103` has no `libvpx.so.9` dependency.

### Known limitations

- The Wine source is a hybrid of Valve Wine 11.0 and GE-Proton 11-6's media rework; it is not the stock DWProton Wine tree.
- The unrelated RSX3D compatibility shim from the GE patch series was skipped.
- Minimal DualSense-haptic helper stubs were added to make the GE audio patches compile; haptics are not implemented by those stubs.
- The tested build uses `-O3 -march=native`, so it is specific to the build machine's CPU.
- REFramework 01422, `intro_skip.lua`, and `OnlineFix.ini` with `EmulateTicket=true` were part of the tested game setup.
- No proprietary game or OnlineFix files are included in this repository.

### Build artifacts used during testing

- DWProton build name: `dwproton-ffmpeg8`
- Proton version string: `dwproton-11.0-15`
- FFmpeg: `n8.1.3` (`libavcodec 62.28.103`)
- Wine base: `9358696fe9a2261329f4a83aa6a65fd436106154`
- GE-Proton media patch series: GE-Proton 11-6 `patches/ge-video-rework`
