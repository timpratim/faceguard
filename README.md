# FaceTouchGuard

A local macOS desktop utility that detects when your hand touches acne-prone regions of your face and shouts funny voice alerts at you to break the habit.

Everything runs locally on-device using MediaPipe landmark-based geometry — no cloud APIs, no VLMs, no custom models. Voice clips are pre-generated using [Gradium TTS]([https://gradium.ai](https://shorturl.at/viNOn)) and played offline.

https://github.com/user-attachments/assets/40af6142-482c-4847-9482-934cd7ace240

## How it works

1. Captures frames from the built-in camera via OpenCV
2. Runs **MediaPipe Face Landmarker** to detect 478 face landmarks
3. Runs **MediaPipe Gesture Recognizer** to detect hands + classify hand pose
4. Builds convex-hull polygons for acne-prone face zones (cheeks, chin, mouth, forehead)
5. Filters out false positives using:
   - **Gesture classification** — rejects `Closed_Fist` (holding a cup), `Open_Palm` (hovering), etc.
   - **Finger extension detection** — only extended fingertips count
   - **Hand proximity check** — hand must be within the face bounding box
   - **Z-depth filter** — rejects hands held in front of the face
6. Requires consecutive positive frames to reduce noise
7. Plays a random funny voice clip (with queuing so clips always finish)
8. Tracks session touch count

### 1. Grant camera permission

The first time you run the app, macOS will prompt you to grant camera access to your terminal app. You can also pre-authorize it in:

**System Settings → Privacy & Security → Camera** → enable your terminal app (Terminal, iTerm2, VS Code, etc.)

### Keyboard controls (when preview window is focused)

| Key | Action |
|-----|--------|
| `q` / `ESC` | Quit |
| `d` | Toggle debug overlay |
| `p` | Toggle preview window |
| `f` | Toggle forehead zone |
| `1` / `2` / `3` | Set sensitivity: low / medium / high |
| `SPACE` | Pause / resume monitoring |

### GUI mode


A small Tkinter control panel with Start/Stop buttons, sensitivity dropdown, and live touch count.

## Face zones

| Zone | Description |
|------|-------------|
| Left cheek | Eye-cheek boundary to jawline (left side) |
| Right cheek | Mirror on right side |
| Chin / jawline | Lower jaw contour from ear to ear |
| Mouth perimeter | Outer lip region |
| Forehead (optional) | Eyebrows to hairline, disabled by default |

Zone landmark indices are documented in `face_zones.py` and can be adjusted.

## False-positive filtering

The app uses a layered filtering approach to avoid beeping when you're not actually touching your face:

| Filter | What it catches |
|--------|----------------|
| Gesture classification | Holding a cup (`Closed_Fist`), waving (`Open_Palm`), V-sign (`Victory`) |
| Finger extension | Hand gripping an object (curled fingers) |
| Hand proximity (bbox) | Hand beside the face, not on it |
| Z-depth | Objects held in front of the face |
| Consecutive frames | Momentary noise, hand passing by quickly |
| Forehead y-guard | Hands above the head (hair adjustment) |

## Sensitivity presets

| Preset | Distance threshold | Consecutive frames | Cooldown | Z-threshold |
|--------|-------------------|--------------------|----------|-------------|
| Low    | 5 px              | 4                  | 2.0 s    | 0.08        |
| Medium | 10 px             | 3                  | 1.2 s    | 0.08        |
| High   | 20 px             | 2                  | 0.8 s    | 0.12        |

## Voice clip behavior

- On face touch, a random clip plays
- Clips cycle through all available files before repeating (shuffled deck)
- If you touch your face while a clip is playing, the next clip is **queued**
- The current clip always finishes before the next one starts
- Only one clip is queued at a time — rapid touches don't stack up indefinitely

## Session logging

On exit, a session summary is appended to `session_log.json`:

```json
{
  "date": "2026-03-29T11:52:34",
  "touch_count": 22,
  "duration_seconds": 120.5,
  "sensitivity": "medium"
}
```

## Known limitations

- macOS only (uses `afplay` for audio playback)
- Camera preview window must run on the main thread on macOS
- Single-face only — ignores additional faces in frame
- Very fast swipe-touches may be missed if they don't persist for enough consecutive frames
- Forehead zone has a slightly higher false-positive rate due to proximity to hair
- No menu-bar / tray integration yet

## Future ideas

- Convert to a native macOS menu-bar app using `rumps`
- Bundle as a `.app` with `py2app`
- Add daily/weekly touch statistics dashboard
- Hotkey to pause/resume monitoring
- Spoken real-time alerts via streaming TTS
- Support for Linux/Windows (replace `afplay`)

## License

MIT
