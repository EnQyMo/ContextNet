# ContextNet — Air Quality Monitoring Node

## Description

This project uses the ContextNet middleware to run an air quality monitoring
system. Sensors report air pollutant readings. The system groups the sensors,
checks the risk level, and sends alerts to mobile devices. A web interface shows
the alerts in real time on a simulated phone screen.

The system has four parts:

- Group Definer (GD) — assigns each beacon to a group.
- Processing Node (PN) — reads sensor alerts, finds the sensor group, and sends a groupcast message.
- Mobile Node (MN) — receives the groupcast message and sends it to the web interface through WebSocket.
- Mobile Node web interface — a React application that shows each alert on a simulated phone screen.

## Code Information

Repository: `git@github.com:EnQyMo/ContextNet.git`

### Modules

| Module | Path | Language | Build | Purpose |
|---|---|---|---|---|
| utils | `utils/` | Java | Maven | Shared JSON parser (`JsonParser`, `JsonParseException`). |
| group-definer | `group-definer/` | Java | Maven | Assigns beacons to groups from `beacons.json`. |
| processing-node | `processing-node/` | Java | Maven | Reads sensor alerts, maps sensors to groups, sends groupcast messages through Kafka. |
| mobile-node | `mobile-node/` | Java | Maven | Receives groupcast messages. Runs a WebSocket server on port 8080. |
| mobile-node-ui | `mobile-node-ui/` | TypeScript | Vite and npm | React web interface. Connects to the WebSocket server and shows alerts. |

### Backend frameworks (mobile-node)

- Spark Java — light web framework.
- Jetty WebSocket — WebSocket server.
- Jackson — JSON processing.

The mobile-node backend code uses Java 8 APIs.

### Frontend stack (mobile-node-ui)

- React 18 — user interface (UI) framework.
- TypeScript — type safety.
- Vite — build tool.
- CSS3 — animations and gradients.
- WebSocket API — real-time communication.

### Web interface file layout

```
mobile-node-ui/
├── src/
│   ├── components/
│   │   ├── MobilePhone.tsx       # phone component
│   │   ├── MobilePhone.css
│   │   ├── AlertCard.tsx         # single alert card
│   │   └── AlertCard.css
│   ├── App.tsx                   # main component
│   ├── App.css
│   ├── types.ts                  # TypeScript types
│   ├── main.tsx                  # entry point
│   └── index.css                 # global styles
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── README.md
```

## Dataset Information

The project has no measured dataset. It uses small JSON configuration files and
one simulated alert file.

### `group-definer/data/beacons.json`

A list of beacons. Each record has:

- `beacon_uuid` — the beacon identifier (string).
- `beacon_group` — the group number (integer).

### `processing-node/data/sensors.json`

A list of sensors. Each record has:

- `sensor_id` — the sensor identifier (string).
- `sensor_name` — the sensor name (string).
- `sensor_group` — the group number (integer).

### `processing-node/data/alert_example.json`

One simulated air quality alert. This file is the simulated dataset. The
structure is:

- `analisys.alert_id` — the alert identifier (string).
- `analisys.timestamp` — the alert time (ISO 8601 string).
- `analisys.sensores` — a list of sensor alerts. Each sensor alert has:
  - `sensor_id` — the sensor identifier.
  - `poluentes` — a list of pollutants. Each pollutant has:
    - `poluente` — the pollutant name (for example `pm25`, `pm4`, `CO2`).
    - `risk_level` — `low`, `moderate`, or `high`.
    - `affected_diseases.disease` — a list of possible health effects.

### WebSocket message format

The Mobile Node sends this JSON to each connected web client:

```json
{
  "timestamp": 1728504516302,
  "topic": "GroupMessageTopic",
  "message": "[Pollutant{name='CO2', riskLevel='moderate', affectedDiseases=AffectedDiseases{disease=[dor de cabeça leve, dificuldade de concentração, fadiga]}}]"
}
```

### Full alert format

```json
{
  "analisys": {
    "alert_id": "alert_1706798417002",
    "timestamp": "2025-10-01T22:40:17.002-03:00",
    "sensores": [
      {
        "sensor_id": "IAQ_6227821",
        "poluentes": [
          {
            "poluente": "pm25",
            "risk_level": "moderate",
            "affected_diseases": {
              "disease": [
                "asma",
                "bronquite",
                "irritação respiratória"
              ]
            }
          },
          {
            "poluente": "pm4",
            "risk_level": "high",
            "affected_diseases": {
              "disease": [
                "irritação respiratória",
                "inflamação sistêmica leve"
              ]
            }
          }
        ]
      }
    ]
  }
}
```

## Requirements

- Docker — runs the Gateway, Kafka, and Zookeeper containers, and the Processing Node and Group Definer containers.
- Java 17 or later — builds and runs the Java modules. Note: the mobile-node backend code uses Java 8 APIs.
- Maven — builds the Java modules.
- Node.js and npm — build and run the web interface.

### Ports

| Service | Port | Protocol |
|---|---|---|
| Gateway | 6200 | UDP |
| Kafka (external) | 6010 | TCP |
| Zookeeper | 6000 | TCP |
| Mobile Node WebSocket | 8080 | TCP |
| Web interface dev server | 3000 | TCP |

## Usage Instructions

### Quick start (three steps)

Use this path to see the web interface without Kafka.

1. Start the Mobile Node backend:

   ```bash
   cd mobile-node
   java -jar target/mobile-node.jar
   ```

   The console shows:

   ```
   WebSocket server started on port 8080
   WebSocket endpoint: ws://localhost:8080/alerts
   ```

2. Start the web interface in another terminal:

   ```bash
   cd mobile-node-ui
   npm install
   npm run dev
   ```

   Open `http://localhost:3000`.

3. Send a test alert. In the Mobile Node terminal, type `A` (Send Alert to PN). The alert shows on the web interface.

### Full system

1. Start the infrastructure (Gateway, Kafka, and Zookeeper):

   ```bash
   docker compose -f start-gw.yml up -d
   ```

2. Compile the utils, Group Definer, Processing Node, and Mobile Node modules:

   ```bash
   source compile-all.sh
   ```

   This script runs `mvn clean install` in `utils/`, `group-definer/`, `processing-node/`, and `mobile-node/`.

3. Start the Processing Node and Group Definer containers:

   ```bash
   docker compose -f contextnet-stationary.yml up --build
   ```

   Alternative without Docker:

   ```bash
   cd processing-node
   java -jar target/processing-node.jar
   ```

4. Start the Mobile Node:

   ```bash
   cd mobile-node/ && java -jar target/mobile-node.jar
   ```

5. Start the web interface:

   ```bash
   cd mobile-node-ui
   npm run dev
   ```

6. Send an alert from the Processing Node. The alert moves from the Processing Node to the Mobile Node to the web interface.

To build the Mobile Node alone with Maven:

```bash
cd mobile-node
mvn clean install
```

### What the web interface shows

- A simulated phone screen in iPhone style, with a notch, a status bar, and a hover 3D effect.
- A connection status at the top.
- Real-time alert cards. Each card shows:
  - `📍` the sensor identifier.
  - `🌫️` the pollutant (for example PM2.5, PM4, CO2).
  - `⚠️` the risk level.
  - `🏥` the related health effects.
- An information panel with the connection status, the risk levels, and counts of received alerts and active sensors.

Example alert card:

```
┌─────────────────────────────┐
│ 📍 IAQ_6227821    🔴 HIGH   │
├─────────────────────────────┤
│ 🌫️ Poluente: PM4           │
│                             │
│ ⚠️ Possíveis efeitos:       │
│   • irritação respiratória  │
│   • inflamação sistêmica    │
└─────────────────────────────┘
```

### Risk levels

| Level | Color |
|---|---|
| LOW | green |
| MODERATE | yellow |
| HIGH | red |

The web interface adjusts to the screen size. On a desktop it shows the phone
and the information panel. On a phone screen it shows the full screen layout.

### Configuration

Change the WebSocket port in the backend, in `MobileNode.java`:

```java
port(8080); // change to the port you want
```

Change the WebSocket address in the frontend, in `App.tsx`:

```typescript
const websocket = new WebSocket('ws://localhost:8080/alerts')
```

Build the web interface for production:

```bash
cd mobile-node-ui
npm run build
```

The output files go to `mobile-node-ui/dist/`.

## Methodology

The alert data moves through the system in these steps:

1. The Group Definer reads `beacons.json` and assigns each beacon to a group.
2. The Processing Node reads `sensors.json` at start. It keeps the map from sensor identifier to group.
3. The Processing Node receives an alert analysis with the structure of `alert_example.json`.
4. For each sensor in the alert, the Processing Node finds the sensor group. If it finds the group, it sends a groupcast message with the pollutant list to the topic `GroupMessageTopic`.
5. Kafka and the Gateway carry the groupcast message to the group members.
6. The Mobile Node receives the message. It sends the message to each connected web client through the WebSocket endpoint `ws://localhost:8080/alerts`.
7. The web interface receives the message and shows an alert card.

## Reproduction Script

The project has no separate dataset generator. The simulated dataset is the file
`processing-node/data/alert_example.json`.

To reproduce the alert flow with this file:

1. Build all modules:

   ```bash
   source compile-all.sh
   ```

2. Start the infrastructure:

   ```bash
   docker compose -f start-gw.yml up -d
   ```

3. Start the Processing Node and Group Definer:

   ```bash
   docker compose -f contextnet-stationary.yml up --build
   ```

4. Start the Mobile Node:

   ```bash
   cd mobile-node && java -jar target/mobile-node.jar
   ```

5. Trigger an alert. Choose one option:
   - In the Mobile Node terminal, type `A` (Send Alert to PN).
   - In the Processing Node terminal, choose `(A) Send Alert to PN` from the menu:

     ```
     (G) Groupcast | (P) Message to PN | (A) Send Alert to PN | (Z) to finish)? A
     ```

The alert then follows the steps in the Methodology section and shows on the web
interface at `http://localhost:3000`.

## Troubleshooting

### The WebSocket does not connect

1. Make sure the Mobile Node runs.
2. Make sure port 8080 is free. On Windows use `netstat -ano | findstr :8080`. On Linux use `ss -ltnp | grep :8080`.
3. Check the browser console for errors.
4. Restart the Mobile Node.

### The web interface does not load

1. Run `npm install` again.
2. Remove and reinstall the packages:

   ```bash
   cd mobile-node-ui
   rm -rf node_modules package-lock.json
   npm install
   npm run dev
   ```

3. Make sure port 3000 is free.

### The alerts do not show

1. Check the Mobile Node logs. Make sure it receives messages.
2. Open the browser DevTools (F12).
3. Open the Network tab, then WS, and read the WebSocket messages.
4. Make sure the JSON format is correct.

## Future Improvements

- Persistent alert history.
- Filters by sensor or pollutant type.
- Browser push notifications.
- Air quality trend charts.
- A map with the sensor locations.
- Dark mode.
- Progressive Web App (PWA) support.

## Citations

This project supports a research paper that is in preparation. Add the paper
reference here after publication. Until then, cite the ContextNet middleware and
this repository (`git@github.com:EnQyMo/ContextNet.git`).

## License and Contribution Guidelines

This project is part of the ContextNet system. The repository has no separate
license file. Contact the repository owners at `github.com/EnQyMo` before you
reuse the code.

To contribute:

1. Fork the repository `EnQyMo/ContextNet`.
2. Create a branch for your change.
3. Open a pull request with a clear description of the change.
