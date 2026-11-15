<div align="center">

🚌 UniLoop
Your campus commute, connected.
Smarter shuttles for university students in Dubai.







Student app · Edge counter · Smart dispatch

</div>

About
Getting to class in Dubai's academic hubs costs students time and money. Taxis and ride-hailing eat into allowances, metro and bus routes don't reach every campus gate, and without live tracking students stand at the curb with no idea whether the shuttle is 2 minutes or 20 minutes away.

Operators have the opposite problem: they can't see the queue. At 8:15 a.m. queues double within minutes and full shuttles leave students behind, while mid-day runs burn fuel on near-empty vehicles.

UniLoop gives both sides one live view.

Students see the live shuttle location, its distance from the stop and an estimated arrival time.

Operators see the live queue at each stop and get a suggestion to send an extra shuttle. The dispatcher always makes the final call.

Features
Module	Status	What it does
📷 Phone-camera stop counter	✅ Working	A phone mounted at the stop counts people in the waiting zone with YOLOv8 running in the browser
📍 Driver GPS sharing	✅ Working	The driver taps "Start sharing" and the shuttle's location streams to Firebase Realtime Database
🗺️ Rider shuttle map	✅ Working	Live shuttle on Google Maps with distance and ETA. Shows a demo bus when no live feed is active
🏫 Campus fleet integration	Next step	University deployment and fleet coordinator onboarding
🧠 Predictive dispatch engine	Next step	Suggestions using incoming seats and arrival times
Verified on Chrome mobile and desktop browsers.

How it works
Edge vision at the stop
UniLoop counts the queue, not identities.

Detect people on-device. The stop phone's camera feeds a lightweight YOLOv8n model running in the browser through ONNX Runtime Web (WebGL backend).

Filter by waiting polygon. Staff draw the waiting boundary once on screen. Only people whose feet fall inside the polygon are counted.

Smooth over 20 seconds. Passersby are filtered out so the count reflects real demand.

Capacity logic
The live curb count is compared against the shuttle's seat capacity to produce a dispatch state:

State	Meaning
🟢 Calm	The queue fits comfortably in the next shuttle
🟡 Busy	The queue is close to seat capacity
🔴 Send an extra shuttle	The queue exceeds what the next shuttle can carry
Adjustable thresholds: set seat capacity and trigger points for your fleet (for example a 14-seater or a 30-seater)

30-second cadence: headcount snapshots are saved every 30 seconds, giving clean time-series data without heavy database writes

CSV export: download history to spot recurring day-of-week spikes and adjust base timetables

Rider experience
The driver taps Start sharing and location updates go to Firebase Realtime Database.

Students open the map and see the shuttle on Google Maps, its distance from the stop and its ETA.

Architecture
text
 ┌───────────── Stop phone ─────────────┐        ┌──── Driver phone ────┐
 │ Camera → YOLOv8n (ONNX Runtime Web)  │        │ "Start sharing"      │
 │ → waiting-polygon filter             │        │ Geolocation API      │
 │ → 20 s smoothing → headcount (int)   │        └──────────┬───────────┘
 └──────────────────┬───────────────────┘                   │ lat / lng
                    │ count only, no video                   │
                    ▼                                        ▼
            ┌─────────────────── Firebase Realtime Database ───────────────────┐
            │  stop queue counts (every 30 s)      ·      live shuttle location │
            └───────────────┬───────────────────────────────────┬──────────────┘
                            ▼                                   ▼
                ┌──── Operator view ────┐            ┌──── Rider map ────┐
                │ Live queue + Calm /   │            │ Google Maps,      │
                │ Busy / Send extra     │            │ distance + ETA    │
                │ CSV export            │            └───────────────────┘
                └───────────────────────┘
Tech stack
Detection: YOLOv8n (Ultralytics), COCO person class

Inference: ONNX Runtime Web with the WebGL backend

Frontend: HTML, CSS and vanilla JavaScript

Realtime data: Firebase Realtime Database

Maps: Google Maps JavaScript API

Getting started
Prerequisites
A recent Chrome browser (desktop or Android)

A Firebase project with Realtime Database enabled

A Google Maps JavaScript API key

Python 3.8+ if you need to export the model to ONNX

1. Clone the repository
bash
git clone https://github.com/theyfwrishic/GDG-HWUD.git
cd GDG-HWUD
2. Prepare the model
The repo ships yolov8n.pt (PyTorch weights, about 6.5 MB). Browsers need the ONNX version, so export it once:

bash
pip install ultralytics
yolo export model=yolov8n.pt format=onnx imgsz=640 opset=12
This creates yolov8n.onnx. Put it where the counter page loads the model from.

3. Add your keys
Replace the placeholders with your own Firebase config and Maps key:

js
const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.firebaseio.com",
  projectId: "your-project",
  appId: "YOUR_APP_ID"
};
xml
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_MAPS_API_KEY"></script>
Restrict your Maps key to your domain in the Google Cloud console, and set Firebase security rules before going public.

4. Run locally
Camera and GPS access need HTTPS or localhost:

bash
python3 -m http.server 8000
# or
npx serve .
Open http://localhost:8000. To test on a phone, deploy to Firebase Hosting or GitHub Pages, or use an HTTPS tunnel.

5. Try the three pages
Stop counter: open it on the stop phone, allow camera access, draw the waiting polygon, and set the seat capacity.

Driver: open it on the driver's phone, allow location access, and tap Start sharing.

Rider map: open it on any device to watch the shuttle approach.

Project structure
text
GDG-HWUD/
├── yolov8n.pt          # YOLOv8 nano weights (export to ONNX for the browser)
├── yolov8n.onnx        # Browser model (generated in step 2)
├── docs/
│   └── UniLoop-your-campus-commute-connected.pptx   # Ideathon pitch deck
└── README.md
Add the counter, driver and rider page files here so the layout matches your repo.

Privacy by design
🚫 No facial recognition. The model only detects that a person is present.

📵 No video retention. Raw frames never leave the stop phone and are never stored.

🔢 Count only. The only thing synced to Firebase is an anonymous whole-number headcount.

📍 Opt-in location. Driver location is shared only while the driver chooses to share it.

Roadmap
Phone-camera stop counter with configurable waiting zone

Driver GPS sharing

Rider live shuttle map with ETA

Phase 1: Campus pilot. Partner with one Dubai university on one high-traffic route to calibrate the counters

Phase 2: Smarter dispatch. Use incoming seats, arrival times, historical peaks and seasonal traffic in suggestions

Phase 3: Carpool marketplace. Once UAE carpooling rules are clear, add verified peer carpooling and a private transport and micro-transit marketplace

Intended impact
Cheaper, more predictable and greener commutes, through fuller shuttles and fewer wasted trips.

Hackathon
Built for IDEAVERSE Ideathon 2026 (GDG Hackathon 2026), hosted by GDGoC BPDC, GDG HWUD and IEI BITS Pilani Dubai.

Team Stack Underflow

Rishi Chowdhury

Aaryan Sukumaran

Contact: as4327@hw.ac.uk

License and credits
YOLOv8 and the yolov8n.pt weights are by Ultralytics and licensed under AGPL-3.0. If you distribute UniLoop with these weights, follow the AGPL-3.0 terms or get an Ultralytics Enterprise License.

Choose a license for your own UniLoop code and add a LICENSE file.
