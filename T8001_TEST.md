# Testing the T8001/T8002 video patch

This branch is an unofficial test build of Eufy Mega Security 0.1.276 with live
video for T8111 cameras behind a T8001 or T8002. It has not been tested on a real
HomeBase. Report problems with it here, not to the upstream project.

The fix alone is on the `t8001-legacy-video` branch. This branch only adds packaging.

## Install

1. In Home Assistant's app store, open **Repositories** and add this, including the
   branch after `#`:

   ```text
   https://github.com/chrishultin/eufy-mega-security#t8001-test
   ```

2. In the original Eufy Mega Security app, note your configuration, turn off
   **Start on boot** and **Watchdog**, then **Stop** it. Both apps use port 3218,
   so they cannot run together.
3. In the app store choose **Check for updates**, then install
   **Eufy Mega Security (T8001 test)** from the T8001 test fork. It builds on your machine,
   which takes a few minutes.
4. Enter the same configuration and start it. It has its own data, so Eufy may ask
   for a verification code or CAPTCHA in the app's web UI.
5. The existing integration should reconnect by itself through discovery. If it
   does not, restart Home Assistant.

## Test

1. Close Eufy live views in Homebridge, HomeKit and the Eufy app. For the cleanest
   result, stop Homebridge during the test: it talks to the same HomeBase.
2. After start, wait about three minutes for the four startup snapshots.
3. Open one camera for 30 seconds, close it, then do the same for the other three,
   one at a time.
4. Only then try two cameras at once. An original HomeBase may serve only one live
   stream at a time.
5. Keep the `camera_stream_closed` lines from the log. They contain no keys.

## Read the result

In a `camera_stream_closed` line:

| Field | Meaning |
|---|---|
| `original_gateway_sign=0` | The unencrypted handshake reply was accepted |
| `original_key=rsa_unwrapped` | The RSA session key worked |
| `original_key=no_rsa_key`, `unwrap_failed`, `lookup_failed`, `key_invalid` or `gateway_short` | The serial-derived key was used, as Homebridge v4 does |
| `original_key=empty_key_field` or `key_size` | Commands were sent unencrypted, as Homebridge v4 does |
| `media_start_protocols=original-...` | Which start command was sent |
| `media_start_acked=1` | The HomeBase received it |
| `media_start_result` | The HomeBase's answer. `0` is success and `503` is busy |
| `video_output_frames` above 0 | Video came out |
| no `original_...` fields at all | The patched path did not run, or the original app is running |

If it fails, stop the test app until the next attempt: each failed startup snapshot
can wake a battery camera.

## Go back

Stop and uninstall the test app, start the original, and turn **Start on boot** and
**Watchdog** back on. The original republishes its connection details when it starts.
Remove the test repository from the app store if you are done.

## Known risks

- Everything was tested against a simulated HomeBase built from Homebridge v4's
  behavior. A real T8001 may differ.
- Cameras are not queued. If the HomeBase allows only one stream, a second view
  retries for about ten seconds while it reports busy, or may take over the first.
- On a T8002, this path replaces the newer ECC method for T8111 cameras.
- If cameras on one channel work and others stay blank, send the log line:
  `foreign_video_frames` will show whether video is arriving on another channel.

## What this does not add

Audio, HomeBase entities, guard mode and camera controls through a T8001 or T8002.
Only live video and the fresh snapshots taken from it.
