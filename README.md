# IoT Monitoring & Kontrol — REST API Node-RED

Nama    : Dinda Aulia Prastiwi 
NPM     : 2410020095
Kelas   : 5A SI REG BJB

ESP32 (simulator Python) → MQTT (Mosquitto) → Node-RED (REST API + WebSocket + SQLite) → Web Dashboard.

## Cara Menjalankan

1. Terminal 1: `mosquitto -v`
2. Terminal 2: `node-red` (install dulu palette `node-red-node-sqlite`)
3. Import `node-red/flows.json` di Node-RED lalu **Deploy**
4. Terminal 3: `pip install paho-mqtt` lalu `python simulator/simulasi_esp32.py`
5. Buka `dashboard/dashboard.html` di browser

API key default: `rahasia123` (ubah lewat environment variable `API_KEY`, dan samakan di `dashboard.html`).

## Pengaturan CORS (wajib)

Node `http in` Node-RED tidak mendukung method OPTIONS, jadi preflight CORS diatur lewat `settings.js`
(`C:\Users\<nama>\.node-red\settings.js`). Hapus komentar pada `httpNodeCors` dan isi:

```js
httpNodeCors: {
    origin: "*",
    methods: "GET,PUT,POST,DELETE,OPTIONS",
    allowedHeaders: "Content-Type,X-API-Key"
},
```
Lalu restart Node-RED.

## Endpoint

| Endpoint | Method | Fungsi | Auth |
|---|---|---|---|
| `/api/suhu` | GET | Data sensor terakhir | X-API-Key |
| `/api/history` | GET | 50 data terakhir (dari SQLite) | X-API-Key |
| `/api/kontrol` | POST | Kirim perintah ke ESP32 | X-API-Key |
| `/api/threshold` | PUT | Set batas suhu `{"max": 30}` | X-API-Key |
| `/api/threshold` | GET | Baca batas suhu | X-API-Key |
| `ws://localhost:1880/ws/sensor` | WebSocket | Data realtime + alert | - |

Contoh:
```bash
curl -H "X-API-Key: rahasia123" http://localhost:1880/api/suhu
curl -X PUT http://localhost:1880/api/threshold -H "Content-Type: application/json" -H "X-API-Key: rahasia123" -d '{"max": 28}'
```

## Tugas Pengembangan (Nilai Tambah)

1. **PUT /api/threshold + notifikasi** — batas suhu diubah lewat dashboard atau `curl`. Saat suhu baru melewati batas, alert dikirim ke MQTT `iot/alert`, debug panel Node-RED, dan banner dashboard. Input tidak valid ditolak dengan HTTP 400.
   - Log dashboard: `docs/01-threshold-log.png`
   - Banner alert: `docs/01-threshold-banner.png`
   - Pesan di MQTT (`mosquitto_sub`): `docs/01-threshold-mqtt.png`
   - Validasi input salah (400): `docs/01-threshold-400.png`
2. **Grafik real-time Chart.js** — suhu, kelembaban, dan garis batas suhu. Bukti: `docs/02-chart.png`
3. **API key `X-API-Key`** — tanpa key → 401, dengan key → 200. Bukti: `docs/03-apikey.png`
4. **WebSocket** (`/ws/sensor`) — dashboard tanpa polling dan auto-reconnect. Bukti: `docs/04-websocket`
5. **History ke SQLite** — data bertahan setelah restart Node-RED. Bukti: `docs/05-sqlite.png` dan `docs/05-sqlite-restart.png`

Bukti kontrol perangkat (POST `/api/kontrol` → MQTT → simulator): `docs/06-kontrol-lamp on.png` dan `docs/06-kontrol-lamp off.png`

