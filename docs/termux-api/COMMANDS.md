# Termux-API Commands Reference

**Completeness**: 100% dei comandi Termux-API documentati **Date**: 2025-12-17

---

## Overview

Questa guida documenta tutti i comandi Termux-API disponibili, con parametri,
output atteso e priorità di integrazione.

**Prerequisito**: `pkg install termux-api` + App Termux:API installata da
F-Droid

---

## Comandi per Categoria

### 1. System Information

#### termux-battery-status

**Descrizione**: Stato batteria del dispositivo **Parametri**: Nessuno
**Output**: JSON

```json
{
  "health": "GOOD",
  "percentage": 85,
  "plugged": "UNPLUGGED",
  "status": "DISCHARGING",
  "temperature": 28.5,
  "current": -450000
}
```

**Priorità**: ALTA **Note**: Utile per automazioni basate su batteria

---

#### termux-audio-info

**Descrizione**: Informazioni audio device **Parametri**: Nessuno **Output**:
JSON con stato speaker/bluetooth/headset **Priorità**: MEDIA

---

#### termux-wifi-connectioninfo

**Descrizione**: Info connessione WiFi attuale **Parametri**: Nessuno
**Output**: JSON con SSID, BSSID, IP, link_speed, rssi **Priorità**: ALTA

---

#### termux-wifi-scaninfo

**Descrizione**: Scan reti WiFi disponibili **Parametri**: Nessuno **Output**:
JSON array delle reti **Priorità**: BASSA

---

#### termux-telephony-deviceinfo

**Descrizione**: Info dispositivo telefonico **Parametri**: Nessuno **Output**:
JSON con IMEI, network_operator, sim_state, etc. **Priorità**: MEDIA
**Privacy**: Contiene dati sensibili

---

#### termux-telephony-cellinfo

**Descrizione**: Info celle telefoniche **Parametri**: Nessuno **Output**: JSON
con info torre cellulare **Priorità**: BASSA

---

#### termux-info

**Descrizione**: Info sistema Termux **Parametri**: Nessuno **Output**: Testo
con versioni e path **Priorità**: ALTA (debug)

---

### 2. Notifications

#### termux-toast

**Descrizione**: Mostra toast message **Parametri**:

- `-s` short duration
- `-g gravity` (top, middle, bottom)
- `-c color` background
- `-C color` text color **Input**: Testo da stdin o argomento **Output**:
  Nessuno **Priorità**: ALTA **Esempio**: `echo "Hello" | termux-toast -g top`

---

#### termux-notification

**Descrizione**: Crea notifica persistente **Parametri**:

- `-t title`
- `-c content`
- `--icon icon_name`
- `--id notification_id`
- `--priority high|low|default`
- `--action action_name`
- `--on-delete command`
- `--button1 label:command`
- `--button2 label:command`
- `--button3 label:command`
- `--led-color RRGGBB`
- `--led-on ms`
- `--led-off ms`
- `--sound`
- `--vibrate pattern`
- `--type default|media`
- `--group group_name`
- `--alert-once`
- `--ongoing`
- `--image-path path` **Output**: Nessuno **Priorità**: ALTA **Esempio**:
  `termux-notification -t "Title" -c "Content" --id mynotif`

---

#### termux-notification-remove

**Descrizione**: Rimuove notifica **Parametri**: `--id notification_id`
**Priorità**: MEDIA

---

#### termux-notification-list

**Descrizione**: Lista notifiche attive **Output**: JSON array **Priorità**:
BASSA

---

### 3. Clipboard

#### termux-clipboard-get

**Descrizione**: Legge clipboard **Parametri**: Nessuno **Output**: Contenuto
clipboard su stdout **Priorità**: ALTA

---

#### termux-clipboard-set

**Descrizione**: Scrive su clipboard **Input**: Testo da stdin **Output**:
Nessuno **Priorità**: ALTA **Esempio**: `echo "text" | termux-clipboard-set`

---

### 4. Media

#### termux-camera-info

**Descrizione**: Info fotocamere disponibili **Output**: JSON con id, facing,
resolutions **Priorità**: MEDIA

---

#### termux-camera-photo

**Descrizione**: Scatta foto **Parametri**:

- `-c camera_id` (0=back, 1=front)
- `output_file` **Output**: File JPEG **Priorità**: MEDIA **Esempio**:
  `termux-camera-photo -c 0 photo.jpg`

---

#### termux-microphone-record

**Descrizione**: Registra audio **Parametri**:

- `-f output_file`
- `-l limit_seconds`
- `-e encoder` (aac, amr_nb, amr_wb)
- `-b bitrate`
- `-r sample_rate`
- `-c channels` (1, 2)
- `-d` (stop recording)
- `-i` (info current recording) **Priorità**: MEDIA

---

#### termux-media-player

**Descrizione**: Controlla media player **Parametri**:

- `play file`
- `pause`
- `stop`
- `info` **Priorità**: BASSA

---

#### termux-media-scan

**Descrizione**: Scan media files **Parametri**: `-r` recursive, file/directory
path **Priorità**: BASSA

---

#### termux-tts-speak

**Descrizione**: Text-to-Speech **Parametri**:

- `-e engine`
- `-l language`
- `-n region`
- `-v variant`
- `-p pitch` (default 1.0)
- `-r rate` (default 1.0)
- `-s stream` (NOTIFICATION, MUSIC, etc.) **Input**: Testo da stdin
  **Priorità**: ALTA **Esempio**: `echo "Ciao" | termux-tts-speak -l it`

---

#### termux-tts-engines

**Descrizione**: Lista TTS engines disponibili **Output**: JSON array
**Priorità**: BASSA

---

#### termux-speech-to-text

**Descrizione**: Speech recognition **Parametri**: Nessuno (avvia ascolto)
**Output**: Testo riconosciuto **Priorità**: MEDIA

---

### 5. Location

#### termux-location

**Descrizione**: Ottiene posizione GPS **Parametri**:

- `-p provider` (gps, network, passive)
- `-r request` (once, last, updates) **Output**: JSON con latitude, longitude,
  altitude, accuracy, etc. **Priorità**: ALTA **Esempio**:
  `termux-location -p gps -r once`

---

### 6. Sensors & Hardware

#### termux-sensor

**Descrizione**: Legge sensori **Parametri**:

- `-l` lista sensori
- `-s sensor_name`
- `-d delay_ms`
- `-n count`
- `-c` cleanup **Output**: JSON con valori sensore **Priorità**: MEDIA

---

#### termux-torch

**Descrizione**: Controlla flashlight **Parametri**: `on` | `off` **Priorità**:
MEDIA

---

#### termux-vibrate

**Descrizione**: Vibrazione **Parametri**:

- `-d duration_ms`
- `-f` force (anche in silent mode) **Priorità**: MEDIA **Esempio**:
  `termux-vibrate -d 500`

---

#### termux-brightness

**Descrizione**: Imposta luminosità **Parametri**: `0-255` o `auto`
**Priorità**: BASSA

---

#### termux-infrared-frequencies

**Descrizione**: Frequenze IR supportate **Output**: JSON array di range
frequenze **Priorità**: BASSA

---

#### termux-infrared-transmit

**Descrizione**: Trasmette IR **Parametri**: `-f frequency` pattern
**Priorità**: BASSA

---

#### termux-fingerprint

**Descrizione**: Autenticazione fingerprint **Output**: JSON con auth_result
**Priorità**: ALTA (security) **Note**: Richiede hardware biometrico

---

### 7. Communication

#### termux-sms-send

**Descrizione**: Invia SMS **Parametri**:

- `-n number` (recipient)
- `-s slot` (SIM slot) **Input**: Messaggio da stdin **Priorità**: ALTA (con
  cautela) **Privacy**: CRITICO

---

#### termux-sms-inbox

**Descrizione**: Legge SMS inbox (deprecato) **Usa**: termux-sms-list

---

#### termux-sms-list

**Descrizione**: Lista SMS **Parametri**:

- `-l limit`
- `-o offset`
- `-t type` (inbox, sent, draft, all)
- `-n` (show phone numbers)
- `-d` (show dates) **Output**: JSON array **Privacy**: CRITICO

---

#### termux-telephony-call

**Descrizione**: Effettua chiamata **Parametri**: `phone_number` **Privacy**:
CRITICO

---

#### termux-call-log

**Descrizione**: Log chiamate **Parametri**:

- `-l limit`
- `-o offset` **Output**: JSON array **Privacy**: CRITICO

---

#### termux-contact-list

**Descrizione**: Lista contatti **Output**: JSON array con name, number
**Privacy**: CRITICO

---

### 8. Storage & Files

#### termux-download

**Descrizione**: Download file con system downloader **Parametri**:

- `-d description`
- `-t title`
- `url` **Priorità**: MEDIA

---

#### termux-share

**Descrizione**: Condivide file/testo **Parametri**:

- `-a action` (edit, send, view)
- `-c content-type`
- `-d` (default activity)
- `-t title` **Input**: File o stdin **Priorità**: MEDIA

---

#### termux-open

**Descrizione**: Apre file con app associata **Parametri**:

- `--send` (send action)
- `--view` (view action)
- `--chooser` (mostra chooser)
- `--content-type type`
- `file_path` **Priorità**: MEDIA

---

#### termux-open-url

**Descrizione**: Apre URL nel browser **Parametri**: `url` **Priorità**: MEDIA

---

#### termux-storage-get

**Descrizione**: Richiede file da storage Android **Parametri**: `output_file`
**Priorità**: BASSA

---

#### termux-saf-\* (Storage Access Framework)

**Comandi**:

- `termux-saf-create` - Crea file
- `termux-saf-dirs` - Lista directory SAF
- `termux-saf-ls` - Lista contenuto
- `termux-saf-managedir` - Gestisci accesso dir
- `termux-saf-mkdir` - Crea directory
- `termux-saf-read` - Legge file
- `termux-saf-rm` - Rimuove file
- `termux-saf-stat` - Info file
- `termux-saf-write` - Scrive file **Priorità**: BASSA **Note**: Per accesso
  storage esterno Android 11+

---

### 9. System Control

#### termux-volume

**Descrizione**: Controlla volume **Parametri**:

- Senza parametri: mostra volumi
- `stream volume` (imposta volume) **Streams**: alarm, music, notification,
  ring, system, call **Priorità**: MEDIA

---

#### termux-wake-lock

**Descrizione**: Acquisisce wake lock (previene sleep) **Priorità**: MEDIA

---

#### termux-wake-unlock

**Descrizione**: Rilascia wake lock **Priorità**: MEDIA

---

#### termux-wallpaper

**Descrizione**: Imposta wallpaper **Parametri**:

- `-f file`
- `-u url`
- `-l` (lockscreen) **Priorità**: BASSA

---

#### termux-wifi-enable

**Descrizione**: Enable/disable WiFi **Parametri**: `true` | `false`
**Priorità**: BASSA **Note**: Potrebbe richiedere permessi speciali

---

### 10. Dialogs

#### termux-dialog

**Descrizione**: Mostra dialog interattivo **Parametri**:

- `-t title`
- `-l` list widget
- `-i` text input
- `-m` multi-select
- `-p` password input
- `-r` radio buttons
- `-s` spinner
- `-d` date picker
- `-T` time picker
- `-c` confirm dialog
- `-C` counter
- `--values v1,v2,v3`
- `--range min,max` **Output**: JSON con risultato **Priorità**: MEDIA

---

### 11. Other

#### termux-nfc

**Descrizione**: Operazioni NFC **Parametri**: Vari per lettura/scrittura tag
**Priorità**: BASSA

---

#### termux-usb

**Descrizione**: Info dispositivi USB **Parametri**:

- `-l` lista devices
- `-r` richiedi permessi
- `-e command` esegui con permessi USB **Priorità**: BASSA

---

#### termux-job-scheduler

**Descrizione**: Schedula job periodici **Parametri**:

- `--pending` lista job
- `--cancel-all` cancella tutti
- `--cancel id` cancella specifico
- `--job-id id`
- `--script path`
- `--period-ms ms`
- `--network type`
- `--battery-not-low`
- `--storage-not-low`
- `--charging`
- `--idle`
- `--persisted` **Priorità**: MEDIA

---

#### termux-keystore

**Descrizione**: Gestione chiavi crittografiche **Parametri**:

- `list` lista chiavi
- `delete alias` elimina chiave
- `generate alias [-a algorithm] [-s size]` genera chiave
- `sign alias` firma dati
- `verify alias` verifica firma **Priorità**: BASSA (avanzato)

---

## Priorità di Integrazione

### Fase 1 - Core (ALTA)

1. termux-battery-status
2. termux-clipboard-get
3. termux-clipboard-set
4. termux-toast
5. termux-notification
6. termux-tts-speak
7. termux-wifi-connectioninfo
8. termux-info

### Fase 2 - Extended (MEDIA)

1. termux-location
2. termux-camera-photo
3. termux-vibrate
4. termux-torch
5. termux-dialog
6. termux-volume
7. termux-audio-info
8. termux-sensor

### Fase 3 - Advanced (BASSA)

1. termux-sms-send (con cautela)
2. termux-microphone-record
3. termux-speech-to-text
4. termux-job-scheduler
5. termux-saf-\*
6. Altri

---

## Security Considerations

### Comandi con Privacy Implications

- `termux-sms-*` - Accesso SMS
- `termux-call-log` - Log chiamate
- `termux-contact-list` - Rubrica
- `termux-telephony-deviceinfo` - IMEI
- `termux-location` - Posizione

### Raccomandazioni

1. **Conferma utente obbligatoria** per comandi privacy-sensitive
2. **Non includere** output sensibile nel contesto LLM
3. **Log audit** per comandi critici
4. **Rate limiting** per prevenire abuse

---

_Author: DioNanos_
