# AI HUB — Accident Detection Web + TextBee SMS

This package is a **simple Flask front end + SMS adapter** for the AI Hub accident-detection project.

The web interface does **not** contain the accident-detection algorithm. It calls the team's existing Python detector through one clearly marked function. TextBee SMS is also isolated in its own file.

## What the finished flow does

```text
Select video
    ↓
Preview video
    ↓
RUN ACCIDENT DETECTION
    ↓
AI Hub Python detector
    ↓
ACCIDENT / NO ACCIDENT
    ↓
If accident:
  timestamp + confidence + snapshot (if supplied)
    ↓
TextBee SMS
```

---

# 1. Folder structure

```text
aihub_accident_detection_web_sms/
│
├── app.py                    # Flask web application
├── detector_adapter.py       # ONLY place to connect AI Hub detector
├── sms_adapter.py            # Complete TextBee REST API code
├── requirements.txt
├── .env.example              # Fill credentials/settings here
├── README.md
│
├── templates/
│   ├── index.html             # Upload + preview page
│   └── result.html            # Detection result page
│
├── static/
│   └── style.css
│
└── uploads/
    └── snapshots/             # Accident snapshots produced by detector
```

---

# 2. IMPORTANT: what the students need to fill/change

There are only **two project-specific areas**.

### A. TextBee settings

Copy `.env.example` to `.env` and fill:

```text
TEXTBEE_API_KEY=YOUR_REAL_TEXTBEE_API_KEY
TEXTBEE_DEVICE_ID=YOUR_DEVICE_ID_IF_REQUIRED
TEXTBEE_RECIPIENT=+91XXXXXXXXXX
```

`TEXTBEE_DEVICE_ID` can be left blank. TextBee can use the account's default/enabled device.

Do **not** put the real API key into Python source code or GitHub.

### B. AI Hub detector

Open:

```text
detector_adapter.py
```

Replace the example section inside `detect_accident()` with the team's existing detector call.

The web application expects this simple dictionary:

```python
{
    "accident": True,
    "confidence": 87.5,
    "timestamp": "00:00:07",
    "snapshot": "accident_00_00_07.jpg"
}
```

For no accident:

```python
{
    "accident": False,
    "confidence": 91.2,
    "timestamp": None,
    "snapshot": None
}
```

If the detector does not create a snapshot, simply return `None` for `snapshot`.

---

# 3. Install Python

Use Python 3.10+ if possible.

Check:

```bash
python --version
```

or on Windows:

```bash
py --version
```

---

# 4. Create a virtual environment

Windows:

```bash
py -m venv .venv
.venv\Scripts\activate
```

PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Linux/macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

---

# 5. Install the package requirements

```bash
pip install -r requirements.txt
```

The SMS code uses Python's `requests` library and `python-dotenv` for `.env` settings.

---

# 6. Configure TextBee

## Step 1 — TextBee account

Create/sign in to the TextBee account.

## Step 2 — Register the Android phone

The Android phone used as the SMS gateway must be registered with TextBee and have a working SIM.

## Step 3 — Get the API key

Generate an API key in the TextBee dashboard.

## Step 4 — Find the device ID

If the students want to force a particular registered phone, copy its device ID.

Otherwise leave:

```text
TEXTBEE_DEVICE_ID=
```

TextBee can select the default/enabled device.

## Step 5 — Choose the recipient

Use international/E.164 format.

Example:

```text
TEXTBEE_RECIPIENT=+919876543210
```

Do not use:

```text
9876543210
```

## Step 6 — Create `.env`

Copy:

```text
.env.example
```

to:

```text
.env
```

Then fill the real values.

Example:

```text
TEXTBEE_API_KEY=xxxxxxxxxxxxxxxx
TEXTBEE_DEVICE_ID=xxxxxxxxxxxxxxxx
TEXTBEE_RECIPIENT=+919876543210
DEMO_MODE=0
```

The `.env` file should not be committed to GitHub.

---

# 7. How the TextBee code works

The complete SMS request is already implemented in `sms_adapter.py`.

The code sends:

```text
POST https://api.textbee.dev/api/v1/gateway/send-sms
```

with:

```text
x-api-key: YOUR_API_KEY
Content-Type: application/json
```

and a JSON body containing:

```json
{
  "recipients": ["+919876543210"],
  "message": "ACCIDENT DETECTED\nVideo: highway.mp4\nTime: 00:00:07\nConfidence: 93.4%"
}
```

If `TEXTBEE_DEVICE_ID` is filled, the request additionally contains:

```json
"deviceId": "YOUR_DEVICE_ID"
```

The current TextBee account-level endpoint and authentication method are documented by TextBee. The device-specific route is not required here.

---

# 8. Accident information sent in the SMS

The SMS is automatically created from the detector result.

Current format:

```text
ACCIDENT DETECTED
Video: highway.mp4
Time: 00:00:07
Confidence: 93.4%
```

The students do **not** need to manually type the timestamp or confidence into the SMS code.

The detector supplies:

```python
result["timestamp"]
result["confidence"]
```

and `sms_adapter.py` inserts them into the message.

If the team later wants to add another detector field, for example an accident class, the only place that needs changing is `build_accident_message()` in `sms_adapter.py`.

---

# 9. Connect the AI Hub detector

Open:

```text
detector_adapter.py
```

Suppose the team's existing detector has:

```python
from accident_pipeline import run_detection
```

and returns:

```python
raw = run_detection(video_path)
```

Then the adapter should become approximately:

```python
from accident_pipeline import run_detection


def detect_accident(video_path: str) -> dict:
    raw = run_detection(video_path)

    return {
        "accident": raw["accident"],
        "confidence": raw.get("confidence"),
        "timestamp": raw.get("timestamp"),
        "snapshot": raw.get("snapshot"),
    }
```

**Do not move the detector into `app.py`.**

Keeping the adapter separate makes the web interface independent of YOLO, CLIP+SVM, or another future detector.

---

# 10. Snapshot integration

If the detector creates an accident image, save it in:

```text
uploads/snapshots/
```

For example:

```text
uploads/snapshots/accident_00_00_07.jpg
```

Return only the filename:

```python
"snapshot": "accident_00_00_07.jpg"
```

The web page will display it automatically.

If there is no snapshot:

```python
"snapshot": None
```

---

# 11. Timestamp integration

The timestamp should represent the first detected accident event according to the team's detector.

Example:

```python
"timestamp": "00:00:07"
```

The result page has a **VIEW EVENT** button. It converts the timestamp to seconds, moves the video player to that position, and starts playback.

Therefore the detector only needs to provide the timestamp; the web interface handles the player jump.

---

# 12. Run the web application

Activate the virtual environment first.

Then:

```bash
python app.py
```

Open the address shown by Flask, normally:

```text
http://127.0.0.1:5000
```

---

# 13. First test WITHOUT the real detector

This is useful before connecting the students' algorithm.

In `.env` temporarily set:

```text
DEMO_MODE=1
```

Run:

```bash
python app.py
```

Upload any video.

The package will simulate:

```text
ACCIDENT DETECTED
Confidence: 93.4%
Timestamp: 00:00:07
```

If TextBee credentials are also filled, the app will attempt the real SMS send. **For a UI-only test, keep TextBee credentials blank and the page will show the SMS error rather than sending a message.**

After UI testing, set:

```text
DEMO_MODE=0
```

and connect the real detector.

---

# 14. Recommended integration order

Do this in exactly this order.

### Test 1 — Web interface only

1. Set `DEMO_MODE=1`.
2. Leave TextBee unconfigured if you do not want to send SMS.
3. Start Flask.
4. Upload a video.
5. Confirm video preview works.
6. Click `RUN ACCIDENT DETECTION`.
7. Confirm the result page appears.
8. Confirm timestamp and confidence are displayed.

### Test 2 — TextBee only

1. Fill the real API key.
2. Fill recipient phone number in E.164 format.
3. Optionally fill device ID.
4. Keep `DEMO_MODE=1`.
5. Upload a short test video.
6. Confirm the phone receives the SMS.
7. Check the returned TextBee response/batch ID if needed.

### Test 3 — Real detector

1. Set `DEMO_MODE=0`.
2. Connect the existing detector in `detector_adapter.py`.
3. Run one known accident video.
4. Confirm the detector returns accident=True.
5. Confirm timestamp is correct.
6. Confirm confidence is shown.
7. Confirm snapshot appears if produced.
8. Confirm SMS contains the same timestamp/confidence.

### Test 4 — Known normal video

Run a known non-accident video.

Expected:

```text
NO ACCIDENT DETECTED
```

No SMS should be sent.

### Test 5 — Final batch testing

After the integration works on individual videos, run the team's normal automated evaluation pipeline. Do not use the web interface as the batch-evaluation system.

---

# 15. What happens when an accident is detected

`app.py` does this:

```text
video upload
     ↓
detect_accident(video_path)
     ↓
result["accident"] == True?
     ↓
YES
     ↓
send_accident_sms(...)
     ↓
show result page
```

SMS is therefore **not sent for normal videos**.

---

# 16. What happens if TextBee fails?

The detection result is still displayed.

The page will show an SMS status such as:

```text
SMS error: TextBee HTTP 401: ...
```

This prevents a TextBee/API problem from hiding the detector result.

Typical causes include:

- invalid/revoked API key
- missing recipient
- incorrect phone number format
- no enabled TextBee device
- phone/device offline
- account limit reached

---

# 17. Common integration mistakes

### Mistake 1 — Putting the detector directly in `app.py`

Don't. Use `detector_adapter.py`.

### Mistake 2 — Sending SMS for every video

Don't. `send_accident_sms()` is called only when `result["accident"]` is true.

### Mistake 3 — Sending a local phone number format

Use E.164, for example:

```text
+919876543210
```

### Mistake 4 — Hardcoding the API key

Don't write:

```python
api_key = "real-key-here"
```

Use `.env`.

### Mistake 5 — Returning a different result structure

Keep the adapter output consistent:

```python
{
    "accident": True/False,
    "confidence": number or None,
    "timestamp": string or None,
    "snapshot": filename or None
}
```

---

# 18. Files students should normally edit

| File | Edit? | Purpose |
|---|---|---|
| `.env` | **YES** | TextBee credentials + recipient |
| `detector_adapter.py` | **YES** | Connect existing accident detector |
| `sms_adapter.py` | Usually no | TextBee communication + SMS message format |
| `app.py` | No | Flask workflow |
| `templates/index.html` | No | Upload/preview UI |
| `templates/result.html` | No | Result UI |
| `static/style.css` | Optional | Visual changes only |

---

# 19. Security note

Never upload `.env` to GitHub.

Add this to `.gitignore`:

```text
.env
.venv/
__pycache__/
uploads/
```

The API key is a credential and should be treated like a password.

---

# 20. TextBee references

The implementation in `sms_adapter.py` follows the current TextBee REST API documentation:

- Send SMS: `POST /api/v1/gateway/send-sms`
- Authentication: `x-api-key`
- Recipients: E.164 phone numbers
- Optional `deviceId`
- The response may include an `smsBatchId`

Official documentation:

https://textbee.dev/docs/sending-sms/sending-sms
https://textbee.dev/docs/api-reference

---

# Final integration checklist

- [ ] Python environment created
- [ ] `pip install -r requirements.txt` completed
- [ ] `.env` created from `.env.example`
- [ ] TextBee API key entered
- [ ] Recipient entered in E.164 format
- [ ] Device ID entered if a specific device is required
- [ ] `DEMO_MODE=0` for actual project
- [ ] Existing detector connected in `detector_adapter.py`
- [ ] Detector returns accident/confidence/timestamp/snapshot
- [ ] Accident video tested
- [ ] Normal video tested
- [ ] SMS received for accident
- [ ] No SMS sent for normal video
- [ ] Timestamp jump verified
- [ ] Snapshot verified if detector produces one
- [ ] `.env` excluded from GitHub

