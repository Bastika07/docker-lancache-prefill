# LANCache Prefill auf TrueNAS SCALE

Es gibt zwei Wege, den Container auf TrueNAS SCALE (24.10 "Electric Eel" oder neuer, Docker-basiertes App-System) zu betreiben:

## 1. Custom App via YAML (empfohlen, sofort nutzbar)

1. Dataset für die Daten anlegen, z.B. `tank/apps/lancache-prefill`.
2. **Apps → Discover Apps → ⋮ (drei Punkte) → Install via YAML**
3. Inhalt von [`docker-compose.yaml`](docker-compose.yaml) einfügen und anpassen:
   - `dns:` auf die IP eures **LanCache-DNS** setzen — sonst laufen die Downloads am Cache vorbei! (Entfernen, falls der LanCache-DNS netzwerkweit als DNS dient.)
   - `volumes:` auf das angelegte Dataset zeigen lassen.
   - `TZ` auf eure Zeitzone setzen, damit die Cron-Zeiten stimmen.
4. App installieren und warten, bis die Prefill-Tools heruntergeladen wurden (App-Log beobachten).

### Erstkonfiguration (einmalig pro Prefill-Tool)

In der TrueNAS-Shell (System → Shell) oder per SSH:

```bash
docker exec -u prefill -it lancache-prefill bash
cd /lancacheprefill/SteamPrefill
./SteamPrefill select-apps     # Steam-Login + Spiele auswählen
exit
```

Für Epic/Battle.net analog mit `EpicPrefill` bzw. `BattleNetPrefill`. Danach die App neu starten — erst dann wird der Cron-Job aktiv (unkonfigurierte Tools werden beim Start automatisch deaktiviert).

## 2. Katalog-App (für eine Einreichung in den offiziellen Community-Train)

Der Ordner [`lancache-prefill/`](lancache-prefill/) enthält die App im Format des [truenas/apps](https://github.com/truenas/apps)-Repos (lib_version 2.x), analog zur offiziellen `lancache-monolithic`-App:

```
lancache-prefill/
├── app.yaml                      # Metadaten (Katalog-Eintrag)
├── ix_values.yaml                # Image, Konstanten, UMASK-Default
├── questions.yaml                # UI-Formular (Prefill-Optionen, DNS, Storage, ...)
├── README.md
└── templates/
    └── docker-compose.yaml       # Jinja2-Template (ix_lib)
```

Hinweise:

- Third-Party-Kataloge werden vom Docker-basierten App-System **nicht** mehr unterstützt; diese Struktur ist daher nur für einen PR gegen `truenas/apps` gedacht (Ziel: `trains/community/lancache-prefill/1.0.0/`).
- Das Verzeichnis `templates/library/` (vendored ix-lib) wird dort von der CI automatisch erzeugt und liegt deshalb hier nicht bei. Ebenso müssen `templates/test_values/` (CI-Render-Tests) vor einer Einreichung noch ergänzt werden.
- Die UI bildet alle Container-Umgebungsvariablen ab: Aktivierung/Parameter/Cron-Zeitplan je Prefill-Tool, globaler Zeitplan, Updates, Log-Cleanup, UID/GID sowie die DNS-Server des Containers (→ LanCache-DNS eintragen).
