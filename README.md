# LoRa secure nodes (PlatformIO) - not tested on real hardware yet

## Layout
- `include/`  shared code + settings: `config.h` (pins, 3 h interval), `secrets.h` (Wi-Fi, Adafruit, AES key; git-ignored), `peer_keys.h` (public keys)
- `src/<name>/main.cpp`  one program per folder; `platformio.ini` has one env per program
- `host_tests/`  logic test that runs on your PC: `g++ -std=c++11 -I include host_tests/test_logic.cpp -o t && ./t`
- `tools/make_aes_key.py`  makes the AES key

## Setup
1. `cp include/secrets.h.example include/secrets.h` and fill it in Then create these feeds in Adafruit IO: lora-log, gateway-hb, n1-batt, n1-rssi, n1-loc (map), n2-batt, n2-rssi, n2-loc.
2. `python3 tools/make_aes_key.py` -> paste the line into include/secrets.h (same key on both nodes).

## Run order (env names)
1. `cloud_test`   fake data -> Adafruit IO (no LoRa/ATECC)
2. `atecc_setup`  once per board; type LOCK; paste each printed public key into include/peer_keys.h (board 1 -> NODE_A, board 2 -> NODE_B)
3. bench: in config.h set FAST_BENCH_MODE 1 and LORA_TX_DBM 10, then flash `node_b` on one board and `node_a` on the other
4. real test: FAST_BENCH_MODE 0 (every 3 h), battery power
5. `compass_test` when the compass arrives, then `twoway` on both boards (change MY_NODE_ID at the top of src/node_twoway/main.cpp: 1 / 2, and HAVE_COMPASS 1)

Build/upload: `pio run -e node_b -t upload`, monitor: `pio device monitor`.

## GPS must be switched off in sleep
NEO-6M draws ~45 mA constantly. GPS GND -> NPN collector (2N2222/2N3904), emitter -> GND, base -> 1k -> GPIO 12 (PIN_GPS_PWR). Without it 4xAA lasts ~1.5 days.
