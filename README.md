# Cyw43Power (Arduino Pico W / Pico 2 W)

Minimal wrapper to control the CYW43 WiFi power-save/performance mode on Raspberry Pi Pico W and Pico 2 W when using the Earle Philhower Arduino core. It also exposes helpers for reading RSSI and link status.

Why
- Some access points aggressively manage power-save clients, causing periodic disconnects or higher latency.
- Disabling power save (“NoSave”) often stabilizes the connection and reduces latency.

Features
- setPowerMode(Default | NoSave | Performance)
- getRSSI(int8_t& outDbm)
- linkStatus()

Compatibility
- Boards: Raspberry Pi Pico W (RP2040 + CYW43439), Raspberry Pi Pico 2 W (RP2350 + CYW43439)
- Core: Earle Philhower Arduino-Pico (earlephilhower/arduino-pico)
- Architectures: rp2040 (covers both RP2040 and RP2350 in this core)

Important: library layout (no src/)
- This library must be installed in a flat layout (legacy Arduino library layout).
- Do NOT place the source files under a src/ subdirectory.
- Expected structure:
  Cyw43Power/
    Cyw43Power.h
    Cyw43Power.cpp
    library.properties
    keywords.txt
    README.md
    examples/
      DisablePowerSave/
        DisablePowerSave.ino

Installation

Option A: Manual install
1) Download/clone this repository.
2) Copy the folder “Cyw43Power” into your Arduino sketchbook libraries folder:
   - Linux/macOS: ~/Arduino/libraries/
   - Windows: Documents/Arduino/libraries/
3) Ensure the files are directly inside Cyw43Power/ (not under src/).
4) Restart Arduino IDE.
5) Open: File → Examples → Cyw43Power → DisablePowerSave.

Option B: Using git
- Clone directly into your libraries folder:
  git clone https://github.com/k-madsenDK/Cyw43Power.git ~/Arduino/libraries/Cyw43Power

Quick start
```cpp
#include <WiFi.h>
#include <Cyw43Power.h>

const char* ssid = "SSID";
const char* pass = "PASS";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED) { delay(250); Serial.print("."); }
  Serial.println("\nWiFi connected");

  // Disable power save for maximum radio performance
  bool ok = Cyw43Power::setPowerMode(Cyw43Power::NoSave);
  Serial.println(ok ? "NoSave OK" : "NoSave UNAVAILABLE");

  int8_t rssi;
  if (Cyw43Power::getRSSI(rssi)) {
    Serial.print("RSSI(dBm)="); Serial.println(rssi);
  }
}

void loop() {
  // Re-apply NoSave after reconnects, if needed
  static unsigned long t = 0;
  if (WiFi.status() == WL_CONNECTED && millis() - t > 30000) {
    Cyw43Power::setPowerMode(Cyw43Power::NoSave);
    t = millis();
  }
  delay(50);
}
```

API
- bool setPowerMode(PowerMode mode)
  - PowerMode: Default | NoSave | Performance (Performance maps to NO_POWERSAVE if the macro is unavailable)
  - Returns true on success, false if the driver headers/symbols are not exposed by your core build.
- bool getRSSI(int8_t& outDbm)
  - Returns true and sets outDbm on success. Typical range: -30..-90 dBm (negative values).
- int linkStatus()
  - Returns the driver link status (CYW43_LINK_DOWN, CYW43_LINK_UP, etc.), or -1 if unavailable.

Notes
- Thread safety: Calls are wrapped in cyw43_arch_lwip_begin()/end() to avoid races with LWIP background threads in the Arduino core.
- Reapply after reconnect: The radio power mode resets on reconnect. Call setPowerMode again after WL_CONNECTED.
- Graceful fallback: If the core doesn’t expose pico/cyw43_arch.h and cyw43.h, functions return false / -1 without crashing.
- RSSI signature: This library uses the newer signature int cyw43_wifi_get_rssi(cyw43_t*, int32_t*).

License
- MIT (see LICENSE)
