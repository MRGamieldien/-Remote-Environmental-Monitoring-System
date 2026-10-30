# SAEON Real-Time Weather Station Monitoring System

A web-based weather monitoring prototype that collects environmental measurements using an **Arduino UNO R4 WiFi**, stores them in **Firebase Realtime Database**, and displays them on a **React dashboard**.

The dashboard is configured for **Dwarsberg Weather Station, Jonkershoek**, with South African time (`Africa/Johannesburg`). It supports real sensor measurements as well as a clearly labelled demonstration mode.

## Project overview

The system is designed to make station measurements accessible through a live, browser-based dashboard. Data travels through the following components:

```text
DHT11 + soil moisture sensor + TR-525M rain gauge
                      |
               Arduino UNO R4 WiFi
                      |
                  Wi-Fi / HTTPS
                      |
             Firebase Realtime Database
                      |
               React + Vite dashboard
```

The board periodically collects readings and sends them to Firebase. The dashboard subscribes to the database so that incoming readings can appear without manually refreshing the page.

## Main features

- **Live monitoring:** Displays the latest available temperature, humidity, soil moisture and rainfall measurements.
- **Rainfall tracking:** Supports rainfall per reporting interval and cumulative rainfall, with counter state stored on the board.
- **Historical graphs:** Displays measurements over selectable date ranges, with summaries and statistics.
- **Station overview:** Includes metric cards, a station image, daily summaries and weather visualisations.
- **Connection feedback:** Shows loading, connection and missing-data states rather than presenting fabricated readings as live measurements.
- **Demo mode:** Offers simulated observations for previewing the interface without hardware or Firebase.
- **Future expansion:** Dashboard fields exist for wind speed, wind direction, net radiation and atmospheric pressure; the matching physical sensors and complete dedicated views are not yet implemented.

> **Status:** This is a prototype. The supplied code and documentation do not establish that the complete system has been verified with physical sensors in the field.

## Technologies used

| Layer | Technology |
|---|---|
| Microcontroller | Arduino UNO R4 WiFi |
| Firmware | Arduino C++ with WiFiS3 and HTTPS |
| Temperature and humidity | DHT11 |
| Soil moisture | Analog soil moisture sensor |
| Rainfall | TR-525M tipping-bucket rain gauge |
| Backend | Firebase Realtime Database |
| Device authentication | Firebase Authentication, email/password |
| Frontend | React 19, Vite 7 |
| Graphs | Recharts |
| Deployment options | Firebase Hosting or Vercel |

## Project structure

```text
weather-station/
├── arduino/
│   └── WeatherStation/
│       ├── WeatherStation.ino       # Arduino firmware
│       ├── FirebaseLogin.h          # Firebase sign-in and token renewal
│       ├── RainState.h              # Rain counter persistence
│       └── arduino_secrets.h.example
├── dashboard/
│   ├── public/                      # Images and graphics
│   ├── src/
│   │   ├── components/              # Dashboard UI and charts
│   │   ├── hooks/useStation.js      # Live Firebase listeners
│   │   ├── lib/                     # Weather data processing and demo data
│   │   ├── config.js                # Station and app settings
│   │   └── App.jsx                  # Main dashboard
│   ├── scripts/                     # Test data and simulation utilities
│   ├── tests/                       # Data-processing tests
│   ├── .env.example
│   └── package.json
├── firebase/
│   ├── database.auth.example.rules.json
│   └── database.prototype.rules.json
├── docs/TESTING.md
├── firebase.json
└── START_HERE.md
```

## Requirements

Install or prepare the following:

- [Node.js](https://nodejs.org/) **22.12 or newer**, including npm
- [Arduino IDE](https://www.arduino.cc/en/software)
- **Arduino UNO R4 Boards** package in Arduino IDE
- Arduino libraries: **DHT sensor library** and **Adafruit Unified Sensor**; also install **ArduinoJson** and **ArduinoHttpClient** used by the Firebase authentication helper
- A Firebase project with **Realtime Database** and **Authentication** enabled
- Arduino UNO R4 WiFi, supported sensors, suitable cables, USB cable and **2.4 GHz Wi-Fi**

## Quick start: Preview the dashboard

If you want to see the interface before connecting hardware:

1. Extract the project ZIP.
2. Open the `weather-station` folder in VS Code.
3. Open **Terminal → New Terminal**.
4. Run:

```bash
cd dashboard
npm ci
npm run demo
```

5. Open the URL shown in the terminal, usually **http://localhost:5173**.

**Demo mode uses artificial readings.** These values are for demonstration only and must not be confused with real measurements.

## Firebase setup for live monitoring

1. Go to [Firebase Console](https://console.firebase.google.com/) and create or select your project.
2. Under **Build → Realtime Database**, create a database and copy its exact **database root URL**.
3. Under **Authentication → Sign-in method**, enable **Email/Password**. Create a dedicated user for the station under **Users**.
4. Open Realtime Database **Rules**. Use `firebase/database.auth.example.rules.json` as the starting point and **replace its example UID with the station user's actual UID**. Review and publish the rules. The included prototype rules permit public writes and should not be used for a public deployment.
5. In **Project settings → General**, register a **Web app** and copy its configuration values.
6. In `dashboard`, make a copy of `.env.example` named `.env`, and fill in:

```dotenv
VITE_FIREBASE_API_KEY=your_web_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_DATABASE_URL=https://your-database-root-url
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
VITE_DATA_MODE=firebase
```

7. Start the real dashboard:

```bash
cd dashboard
npm ci
npm run dev
```

The page should initially show missing/waiting measurements until your device successfully uploads real observations.

### Configure the Arduino credentials

Copy:

`arduino/WeatherStation/arduino_secrets.h.example`

and save it as:

`arduino/WeatherStation/arduino_secrets.h`

The **current firmware** requires the following definitions in this private file:

```cpp
#pragma once
#define SECRET_WIFI_SSID "YOUR_WIFI_NAME"
#define SECRET_WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
#define FIREBASE_DATABASE_URL "https://YOUR_DATABASE_ROOT_URL"
#define FIREBASE_API_KEY "YOUR_FIREBASE_WEB_API_KEY"
#define FIREBASE_USER_EMAIL "YOUR_STATION_USER_EMAIL"
#define FIREBASE_USER_PASSWORD "YOUR_STATION_USER_PASSWORD"
#define FIREBASE_AUTH_TOKEN ""
```

**Important:** The packaged `.example` header does not yet list the three `FIREBASE_API_KEY`, `FIREBASE_USER_EMAIL` and `FIREBASE_USER_PASSWORD` values, but the current `FirebaseLogin.h` depends on them. Add them as shown above. The firmware signs in and refreshes its authentication token. Never commit the actual credentials.

## Sensor wiring

**Switch off or disconnect the board before wiring.** Check each module's supply voltage and pin labels.

| Sensor | Sensor connection | Arduino UNO R4 WiFi |
|---|---|---|
| DHT11 | DATA | **D4** |
| DHT11 | VCC and GND | Compatible power supply and GND |
| Soil moisture | Analog output (AO) | **A0** |
| Soil moisture | VCC and GND | Rated supply and GND |
| TR-525M rain gauge | One dry-contact wire | **D2** |
| TR-525M rain gauge | Other dry-contact wire | **GND** |

- The rain gauge uses the board's internal pull-up; do not connect its dry-contact input to 5 V.
- If using a bare DHT11 sensor, a suitable data pull-up resistor may be needed.
- Ensure all powered sensor modules share the Arduino's ground.
- Check that analog output voltages remain within the UNO R4's supported input range.

### Calibration

The firmware currently includes these example values in `WeatherStation.ino`:

```cpp
constexpr int SOIL_DRY_RAW = 800;
constexpr int SOIL_WET_RAW = 350;
constexpr float MM_PER_TIP = 0.5f;
```

Replace the soil dry/wet values using measurements from **your actual sensor**. Verify the rain-gauge millimetres-per-tip value against the specific TR-525M unit and its calibration. These values should not be assumed universally correct.

## Upload the Arduino program

1. Connect the Arduino UNO R4 WiFi to your computer with a USB data cable.
2. In Arduino IDE, install the **Arduino UNO R4 Boards** package in Boards Manager.
3. Install the required libraries in Library Manager.
4. Open `arduino/WeatherStation/WeatherStation.ino`.
5. Select **Tools → Board → Arduino UNO R4 WiFi** and the correct port.
6. Confirm that `arduino_secrets.h` contains your own credentials and Firebase URL.
7. Click **Upload**.
8. Open **Serial Monitor**, set **115200 baud**, and check the Wi-Fi, authentication and upload messages.

The sketch sends readings on an approximately **five-minute interval**, with retries for some errors. It also maintains rain-tip state on the board across restarts using flash-backed EEPROM storage.

## Firebase data structure

The dashboard reads two main database paths:

```text
station/
├── latest/                    # Most recent reading
└── readings/                  # Timestamped history entries
    ├── -FirebasePushId1/
    └── -FirebasePushId2/
```

A representative reading looks like this:

```json
{
  "ts": 1793354400000,
  "temperature": 23.4,
  "humidity": 55.0,
  "soil_moisture": 41.0,
  "rain_interval_mm": 0.0,
  "rain_total_mm": 12.5,
  "reading_id": "uno-r4-01-25"
}
```

This example is illustrative, **not a current real measurement**. The `ts` field is a Unix timestamp in milliseconds. Failed or unavailable sensor fields may be omitted rather than replaced with invented readings.

## How live updates work

1. The Arduino reads the connected sensors.
2. It connects to Firebase over Wi-Fi and HTTPS using its dedicated Firebase identity.
3. It writes a new history record to `/station/readings` and updates `/station/latest`.
4. The React application listens to those database paths using Firebase's `onValue()` functionality.
5. New measurements appear on the overview and relevant charts without a manual page refresh.

**Note:** Realtime Database listeners deliver changes as they happen, but the firmware's normal sampling/upload interval is around five minutes. This is a live-updating dashboard, not continuous per-second sampling.

## Testing

From the `dashboard` folder:

```bash
npm test
npm run build
```

Additional scripts are provided:

```bash
npm run mock       # Generate synthetic sample data locally
npm run seed       # Seed Firebase with synthetic data; check configuration first
npm run simulate   # Simulate periodic Firebase measurements
```

**Do not seed your production measurement database with fake data.** Use a dedicated test project. See `docs/TESTING.md` for more extensive verification steps.

## Deployment

To build the dashboard:

```bash
cd dashboard
npm ci
npm run build
```

The output is generated in `dashboard/dist`. The included `firebase.json` supports Firebase Hosting, and `dashboard/vercel.json` supports Vercel. On Vercel, set the project root directory to `dashboard` and supply the required `VITE_FIREBASE_*` environment variables.

**Before publicly deploying:** verify Firebase Security Rules, restrict write access to the station identity, check database costs and retention, and make sure no actual passwords or private tokens are included in the repository or deployment files.

## Troubleshooting

| Issue | What to check |
|---|---|
| Dashboard shows demo data | Use `npm run dev` instead of `npm run demo` and set `VITE_DATA_MODE=firebase` |
| Dashboard shows no readings | Confirm `/station/latest` exists in Realtime Database and Firebase web configuration is correct |
| Permission denied | Confirm database rules and Firebase Authentication setup |
| Arduino cannot connect | Check 2.4 GHz Wi-Fi, board selection, credentials, TLS connectivity and Firebase root URL |
| Authentication error | Check Email/Password sign-in, web API key, station user credentials and authorized UID |
| DHT11 has no reading | Check D4, power, ground and DHT11 library |
| Soil moisture stuck at 0%/100% | Check A0, analog output and wet/dry calibration |
| Rain gauge not counting | Check the dry contact between D2 and GND and inspect Serial Monitor |
| Dashboard is out of date | Check board connectivity and the time of the latest Firebase reading |


## Further documentation

- [`START_HERE.md`](START_HERE.md) — short setup guide
- [`docs/TESTING.md`](docs/TESTING.md) — test checklist and acceptance guidance
- [Firebase Realtime Database documentation](https://firebase.google.com/docs/database)
- [Arduino UNO R4 WiFi documentation](https://docs.arduino.cc/hardware/uno-r4-wifi/)

---

**SAEON Weather Station Monitoring Prototype** — Arduino sensors, Firebase and a React real-time dashboard.
