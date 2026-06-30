# AMP Controller (SR / DSP-3004D) under Wine — Setup & Fixes

> App: **(SR)AMP Controller V1.1.9** (WPF / .NET Framework, chinesische Herkunft)
> Gerät: **DSP-3004D**, LAN-gesteuert über UDP, IP `192.168.178.30`
> System: CachyOS / Arch Linux, KDE Plasma (Wayland + XWayland), Wine 11.x
> Prefix: `~/.wine-ampcontroller`

---

## 🇬🇧 English

### Overview
Getting the AMP Controller fully working under Wine required five separate fixes.
Each problem and its solution is listed below.

### 1. App would not start (crash on launch)
- **Cause:** The app is a **WPF / .NET Framework 4.5+** application. Wine's built-in
  Mono cannot run WPF and crashed with a `NullReferenceException`.
- **Fix:** Install the **real Microsoft .NET Framework 4.8** into the prefix:
  ```bash
  export WINEPREFIX=$HOME/.wine-ampcontroller
  winetricks -q dotnet48
  ```

### 2. Sub-window ("Set") rendered as a black box
- **Cause:** WPF uses Direct3D hardware acceleration, which renders black/broken
  under Wine.
- **Fix:** Force **software rendering** for WPF via the registry:
  ```
  HKEY_CURRENT_USER\Software\Microsoft\Avalon.Graphics
      DisableHWAcceleration = dword:00000001
  ```

### 3. Device was not discovered on the LAN
- **Cause:** `/etc/hosts` mapped the hostname to `127.0.1.1`. The app resolved its
  own hostname and **bound its discovery UDP socket to localhost**, so it never
  reached the LAN.
- **Fix:** Point the hostname to the real LAN IP in `/etc/hosts`:
  ```
  # before:  127.0.1.1      cachyos-x8664
  # after:   192.168.178.26 cachyos-x8664
  ```
  (Backup saved as `/etc/hosts.bak-ampcontroller`.)
- **Note:** Uses a fixed IP — best to assign a **static DHCP lease** in the router.

### 4. Live readouts stayed at 0 (VU meters, Voltage, Current, Temp, Impedance)
- **Cause:** A stale app instance was stuck in a **heartbeat-only state**
  (only the 27-byte → 10-byte keepalive on UDP `45454`↔`45455`). It never
  completed the `015c` hello / `010d` config handshake, so the device never
  started sending its `06c2` data blocks (460 B) and the 115-byte telemetry
  stream → all values read 0.
- **Fix:** **Cleanly restart the app.** A fresh start runs the full handshake and
  the device unlocks the telemetry stream.
- **Rule of thumb:** If values ever show 0 again → restart the AMP Controller.

### 5. Some characters shown as boxes ("tofu", e.g. Temp `56▯`)
- **Cause:** The app uses a CJK / symbol glyph (e.g. **℃** U+2103) that the
  default Wine font lacks; no per-glyph fallback → empty box.
- **Fix:** Register an installed **Noto Sans CJK** font in the prefix and map the
  Chinese font names onto it:
  ```bash
  cp /usr/share/fonts/noto-cjk/NotoSansCJK-Regular.ttc \
     ~/.wine-ampcontroller/drive_c/windows/Fonts/
  ```
  Registry (`HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes`):
  ```
  SimSun            = Noto Sans CJK SC
  NSimSun           = Noto Sans CJK SC
  宋体               = Noto Sans CJK SC
  Microsoft YaHei   = Noto Sans CJK SC
  微软雅黑           = Noto Sans CJK SC
  MS Shell Dlg      = Noto Sans CJK SC
  MS Shell Dlg 2    = Noto Sans CJK SC
  Tahoma            = Noto Sans CJK SC
  Microsoft Sans Serif = Noto Sans CJK SC
  ```

### How to start the app
```bash
export WINEPREFIX=$HOME/.wine-ampcontroller
wine "$WINEPREFIX/drive_c/Program Files/AMP Controller/AMP controller.exe"
```

### Protocol notes (for reference)
- Control/telemetry: **UDP**, PC port `45454` ↔ device port `45455`.
- Every message is acknowledged by a 10-byte ACK echoing the 2-byte command ID.
- Handshake: `015c` hello (device) → `010d` config requests (PC) →
  `016a`/`06c2` data blocks (device) → continuous 115-byte telemetry stream.
- Idle keepalive: `03d9` (27 B) → `9401` (10 B).
- The device shares its IP with an **Audinate/Dante** chip
  (multicast `224.0.0.233:8708`, payload "Audinate") — unrelated to amp control.

---

## 🇩🇪 Deutsch

### Überblick
Damit der AMP Controller unter Wine vollständig läuft, waren fünf einzelne Fixes
nötig. Jedes Problem und seine Lösung sind unten aufgeführt.

### 1. App startete nicht (Absturz beim Start)
- **Ursache:** Die App ist eine **WPF-/.NET-Framework-4.5+**-Anwendung. Das in
  Wine eingebaute Mono kann WPF nicht ausführen und stürzte mit einer
  `NullReferenceException` ab.
- **Fix:** Das **echte Microsoft .NET Framework 4.8** in den Prefix installieren:
  ```bash
  export WINEPREFIX=$HOME/.wine-ampcontroller
  winetricks -q dotnet48
  ```

### 2. Unterfenster ("Set") wurde schwarz angezeigt
- **Ursache:** WPF nutzt Direct3D-Hardwarebeschleunigung, die unter Wine
  schwarz/fehlerhaft rendert.
- **Fix:** **Software-Rendering** für WPF per Registry erzwingen:
  ```
  HKEY_CURRENT_USER\Software\Microsoft\Avalon.Graphics
      DisableHWAcceleration = dword:00000001
  ```

### 3. Gerät wurde im LAN nicht gefunden
- **Ursache:** `/etc/hosts` mappte den Hostnamen auf `127.0.1.1`. Die App löste
  ihren eigenen Hostnamen auf und **band ihren Discovery-UDP-Socket an localhost**
  — so erreichte sie nie das LAN.
- **Fix:** Den Hostnamen in `/etc/hosts` auf die echte LAN-IP zeigen lassen:
  ```
  # vorher:  127.0.1.1      cachyos-x8664
  # nachher: 192.168.178.26 cachyos-x8664
  ```
  (Backup gespeichert als `/etc/hosts.bak-ampcontroller`.)
- **Hinweis:** Nutzt eine feste IP — am besten im Router eine **feste
  DHCP-Zuweisung** für den PC einrichten.

### 4. Messwerte blieben auf 0 (VU-Meter, Voltage, Current, Temp, Impedance)
- **Ursache:** Eine hängende App-Instanz steckte in einem **reinen
  Heartbeat-Zustand** (nur das 27-Byte- → 10-Byte-Keepalive auf UDP
  `45454`↔`45455`). Sie durchlief nie den `015c`-Hello-/`010d`-Config-Handshake,
  also begann das Gerät nie, seine `06c2`-Datenblöcke (460 B) und den
  115-Byte-Telemetrie-Stream zu senden → alle Werte blieben 0.
- **Fix:** **App sauber neu starten.** Ein frischer Start durchläuft den
  kompletten Handshake und das Gerät gibt den Telemetrie-Stream frei.
- **Merksatz:** Falls die Werte je wieder auf 0 stehen → AMP Controller neu starten.

### 5. Einige Zeichen als Kästchen ("Tofu", z. B. Temp `56▯`)
- **Ursache:** Die App nutzt ein CJK-/Sonderzeichen (z. B. **℃** U+2103), das der
  Standard-Wine-Schrift fehlt; keine Glyph-Ersetzung → leeres Kästchen.
- **Fix:** Eine installierte **Noto-Sans-CJK**-Schrift im Prefix registrieren und
  die chinesischen Schriftnamen darauf mappen:
  ```bash
  cp /usr/share/fonts/noto-cjk/NotoSansCJK-Regular.ttc \
     ~/.wine-ampcontroller/drive_c/windows/Fonts/
  ```
  Registry (`HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes`):
  ```
  SimSun            = Noto Sans CJK SC
  NSimSun           = Noto Sans CJK SC
  宋体               = Noto Sans CJK SC
  Microsoft YaHei   = Noto Sans CJK SC
  微软雅黑           = Noto Sans CJK SC
  MS Shell Dlg      = Noto Sans CJK SC
  MS Shell Dlg 2    = Noto Sans CJK SC
  Tahoma            = Noto Sans CJK SC
  Microsoft Sans Serif = Noto Sans CJK SC
  ```

### App starten
```bash
export WINEPREFIX=$HOME/.wine-ampcontroller
wine "$WINEPREFIX/drive_c/Program Files/AMP Controller/AMP controller.exe"
```

### Protokoll-Notizen (zur Referenz)
- Steuerung/Telemetrie: **UDP**, PC-Port `45454` ↔ Geräte-Port `45455`.
- Jede Nachricht wird mit einem 10-Byte-ACK quittiert, das die 2-Byte-Befehls-ID
  zurückspiegelt.
- Handshake: `015c` Hello (Gerät) → `010d` Config-Requests (PC) →
  `016a`/`06c2` Datenblöcke (Gerät) → laufender 115-Byte-Telemetrie-Stream.
- Leerlauf-Keepalive: `03d9` (27 B) → `9401` (10 B).
- Das Gerät teilt sich die IP mit einem **Audinate/Dante**-Chip
  (Multicast `224.0.0.233:8708`, Payload "Audinate") — irrelevant für die
  Amp-Steuerung.
