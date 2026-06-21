# 🥋 Hük-tti Infoskærm — Opsætningsguide

## Filer i dette projekt
- `index.html` — Selve infoskærmen
- `slides.json` — Alt indhold du redigerer online
- `SETUP.md` — Denne guide

---

## Del 1: Upload til GitHub (gratis hosting)

1. Opret en konto på **github.com** hvis du ikke har en
2. Klik "New repository" → navngiv det fx `huktti-signage`
3. Upload `index.html` og `slides.json`
4. Gå til Settings → Pages → Source: `main` branch
5. Din URL bliver: `https://DITBRUGERNAVN.github.io/huktti-signage`

**Redigér slides:** Klik på `slides.json` → blyant-ikon → rediger → "Commit changes"
Skærmen opdaterer automatisk inden for ~10 minutter.

---

## Del 2: Raspberry Pi opsætning

### Krav
- Raspberry Pi 3B+ eller 4 (anbefales)
- Raspberry Pi OS (Bullseye eller nyere)
- HDMI-kabel til TV
- Internetforbindelse (WiFi eller kabel)

### Installation

```bash
# 1. Opdater systemet
sudo apt update && sudo apt upgrade -y

# 2. Installér Chromium (normalt allerede installeret)
sudo apt install -y chromium-browser

# 3. Deaktivér skærmslukning
sudo raspi-config
# → Display Options → Screen Blanking → No

# 4. Opret autostart-fil
mkdir -p ~/.config/autostart
nano ~/.config/autostart/signage.desktop
```

Indsæt dette i filen (skift URL til din GitHub Pages URL):
```ini
[Desktop Entry]
Type=Application
Name=Hüktti Signage
Exec=chromium-browser \
  --kiosk \
  --noerrdialogs \
  --disable-infobars \
  --disable-session-crashed-bubble \
  --disable-features=Translate \
  --autoplay-policy=no-user-gesture-required \
  --no-first-run \
  https://DITBRUGERNAVN.github.io/huktti-signage
StartupNotify=false
```

```bash
# 5. Gem og genstart
sudo reboot
```

### Vend skærm (hvis nødvendigt)
Tilføj i `/boot/config.txt`:
```
display_rotate=2  # 180 grader
```

---

## Del 3: Facebook RSS-feed

Facebook blokerer direkte adgang, men via **RSSHub** kan du hente opslag som RSS.

### Mulighed A: RSSHub.app (nemmest)
1. Gå til `https://rsshub.app`
2. Søg efter "Facebook" → vælg "Facebook Page"
3. Find din sides ID (fx fra sidens URL: facebook.com/**huktti**)
4. URL-format: `https://rsshub.app/facebook/page/SIDE-ID`
5. Indsæt URL i `slides.json` under `facebook_rss_1`

### Mulighed B: Facebook Graph API (officiel)
Kræver oprettelse af Facebook App på developers.facebook.com.
Kontakt mig, hvis du vil have hjælp til det.

---

## Del 4: Redigér slides.json

### Skjul en slide midlertidigt (uden at slette den)
Sæt `"active": false` på slide-blokken. Slidet springes over i rotationen, men forbliver i filen, så du nemt kan genaktivere det senere ved at sætte `"active": true` igen.

```json
{
  "id": "info3",
  "type": "info",
  "title": "Sommerlejr",
  "active": false   ← denne slide vises ikke lige nu
}
```

### Tilføj en helt ny slide
Kopier en eksisterende slide-blok ind i `"slides"`-listen, giv den et unikt `"id"`, og udfyld indholdet. Der er ingen grænse for antal slides — de roterer automatisk i den rækkefølge, de står i listen.

### Fjern en slide permanent
Slet hele `{ ... }`-blokken for den pågældende slide fra `"slides"`-listen (pas på at fjerne det omkringliggende komma korrekt, så JSON'en stadig er gyldig).

### Ændre global slide-hastighed
`"slide_duration": 8000` i `"settings"` = 8 sekunder for alle slides som standard.

### Give en enkelt slide sin egen varighed
Tilføj `"duration"` direkte på den slide (i millisekunder) — den tilsidesætter den globale indstilling. Praktisk til billedslides, der gerne må stå længere:
```json
{ "id": "gallery1", "type": "gallery", "duration": 15000, ... }
```

---

## Del 5: Billedgalleri-slide (op til 20 billeder)

Opret en slide med `"type": "gallery"`:

```json
{
  "id": "gallery1",
  "type": "gallery",
  "title": "Billeder fra klubben",
  "icon": "📸",
  "duration": 12000,
  "images": [
    { "url": "https://din-billede-url.dk/foto1.jpg", "caption": "Sommerstævne 2025" },
    { "url": "https://din-billede-url.dk/foto2.jpg", "caption": "Grading august" }
  ],
  "active": true
}
```

**Sådan virker det:**
- 1–6 billeder: vis dem som et **roterende fuldskærmskarrusel** (skift hver 3,5 sek.) ved at tilføje `"mode": "carousel"` på slidet.
- 7–20 billeder: vises automatisk som et **gittergalleri** der skalerer pænt (3×3, 4×4, 5×4 osv. afhængig af antal billeder).
- `"caption"` er valgfri tekst der vises ovenpå billedet.

**Hvor får jeg billed-URL'er fra?**
Nemmeste løsning: upload billederne til samme GitHub-repo (i en mappe fx `images/`) og brug den "raw" GitHub-URL, eller brug en gratis billedhost som **imgur.com**. Billeder skal være tilgængelige via en offentlig URL — Raspberry Pi'en henter dem direkte fra nettet.

**Eksempel med GitHub-hostede billeder:**
```
https://raw.githubusercontent.com/DITBRUGERNAVN/huktti-signage/main/images/foto1.jpg
```

### Ændre farver
```json
"colors": {
  "primary": "#0a1628",   ← baggrund
  "accent": "#c8102e",    ← rød accent
  "gold": "#f0b429"       ← guldfarve
}
```

---

## Del 6: Standby-tilstand & træningsskema

Standby-skærmen vises automatisk uden for træningstid. Den styres af `"training_schedule"`:

```json
"training_schedule": [
  { "day": 1, "label": "Begyndere", "start": "17:00", "end": "18:30" }
]
```

`"day"` er ugedag: 0=søndag, 1=mandag, 2=tirsdag … 6=lørdag.

Slideshowet starter automatisk **30 min før** træningsstart og skifter tilbage til standby **60 min efter** træningsslut. Disse to tal står i `index.html` øverst i scriptet (`STANDBY_BEFORE_MIN` og `STANDBY_AFTER_MIN`), hvis du vil justere dem.

---

## Tip til drift

- **Daglig genstart:** Tilføj i crontab (`crontab -e`):
  `0 6 * * * sudo reboot`
  (genstarter hver dag kl. 06:00)

- **Skærm til/fra automatisk:**
  ```bash
  # Sluk skærm kl. 22:
  0 22 * * * vcgencmd display_power 0
  # Tænd skærm kl. 06:
  0 6 * * * vcgencmd display_power 1
  ```

- **Lokal backup:** Gem en kopi af `slides.json` på Pi'en som fallback.
