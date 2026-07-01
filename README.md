# AMP Controller unter Wine — Anleitung

App: (SR)AMP Controller V1.1.9
Getestetes-Gerät: DSP-3004D
Wine-Prefix: ~/.wine-ampcontroller

---

## 🇩🇪 Deutsch

### Einrichtung (einmalig)

1. Eigenes Wine-Prefix anlegen und Installer ausführen:
   ```bash
   export WINEPREFIX=$HOME/.wine-ampcontroller
   wine "(SR)AMP Controller V1.1.9.exe"
   ```

2. .NET Framework 4.8 installieren (sonst startet die App nicht):
   ```bash
   winetricks -q dotnet48
   ```

3. Software-Rendering aktivieren (sonst sind Fenster schwarz):
   ```bash
   wine reg add "HKCU\Software\Microsoft\Avalon.Graphics" /v DisableHWAcceleration /t REG_DWORD /d 1 /f
   ```

4. Hostnamen auf die LAN-IP zeigen lassen (sonst wird das Gerät nicht gefunden).
   In `/etc/hosts` die Zeile `127.0.0.1 cachyos-x8664` ändern zu (Ip addresse / hostname zu deiner abändern!):
   ```
   192.168.178.26  cachyos-x8664
   ```

5. CJK-Schrift einrichten (sonst Kästchen/Tofu statt Zeichen wie ℃):
   ```bash
   cp /usr/share/fonts/noto-cjk/NotoSansCJK-Regular.ttc \
      ~/.wine-ampcontroller/drive_c/windows/Fonts/
   wine reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes" /v "SimSun" /d "Noto Sans CJK SC" /f
   ```

### App starten
```bash
export WINEPREFIX=$HOME/.wine-ampcontroller
wine "$WINEPREFIX/drive_c/Program Files/AMP Controller/AMP controller.exe"
```

### Wenn alle Werte auf 0 stehen
App neu starten. Beim sauberen Start verbindet sie sich richtig mit dem Gerät.

---

## 🇬🇧 English

### Setup (one time)

1. Create the Wine prefix and run the installer:
   ```bash
   export WINEPREFIX=$HOME/.wine-ampcontroller
   wine "(SR)AMP Controller V1.1.9.exe"
   ```

2. Install .NET Framework 4.8 (the app won't start without it):
   ```bash
   winetricks -q dotnet48
   ```

3. Enable software rendering (otherwise windows are black):
   ```bash
   
   wine reg add "HKCU\Software\Microsoft\Avalon.Graphics" /v DisableHWAcceleration /t REG_DWORD /d 1 /f
   ```

4. Point the hostname to your LAN IP (otherwise the device is not found).
   In `/etc/hosts`, change the line `127.0.0.1 cachyos-x8664` to like (Change ip to match yours!):
   ```
   192.168.178.26  cachyos-x8664
   ```

5. Add a CJK font (otherwise some characters like ℃ show as boxes):
   ```bash
   cp /usr/share/fonts/noto-cjk/NotoSansCJK-Regular.ttc \
      ~/.wine-ampcontroller/drive_c/windows/Fonts/
   wine reg add "HKLM\Software\Microsoft\Windows NT\CurrentVersion\FontSubstitutes" /v "SimSun" /d "Noto Sans CJK SC" /f
   ```

### Start the app
```bash
export WINEPREFIX=$HOME/.wine-ampcontroller
wine "$WINEPREFIX/drive_c/Program Files/AMP Controller/AMP controller.exe"
```

### If all values show 0
Restart the app. A clean start connects to the device properly.
