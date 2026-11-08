# Mail-Wächter

Sortiert die Mails **eines bestimmten Absenders** im GMX-Postfach vor und meldet sich per Telegram.

| Kategorie | Was mit der Mail passiert | Nachricht aufs Handy |
|---|---|---|
| ⚠️ **WICHTIG** (Gesundheit, Notfall, Sicherheit, Übergabe, Schule, Behörden, Drohungen) | bleibt **ungelesen**, Ordner `Vater-Mails/Wichtig` | **mit Ton**, sachliche Kurzfassung |
| ❓ **UNKLAR** (KI unsicher, nur Anhang, KI gestört, neue Absenderadresse) | bleibt ungelesen, Ordner `Vater-Mails/Pruefen` | **mit Ton**, „bitte selbst ansehen“ |
| 📭 **BELANGLOS** (Kleidung, Vorwürfe, Gerede) | als gelesen markiert, Ordner `Vater-Mails/Belanglos` | **stumm**, eine Sammelnachricht pro Durchlauf |

Mails von anderen Absendern werden **nicht angefasst**. **Es wird nie etwas gelöscht.**

## Wie entschieden wird
1. **Stichwörter (über 240, in 9 Gruppen):** Kommt eines vor, ist die Mail **immer** WICHTIG, egal was die KI sagt. Auch Wortteile zählen („schmerz“ trifft *Bauchschmerzen*), Groß/klein und ä/ae sind egal. Die Liste steht in `config.json`.
2. **KI (Claude):** liest jede Mail und ordnet sie nach den Regeln `wichtig_wenn` / `belanglos_beispiele` in `config.json` ein. Ist sie unsicher, gilt die Mail als UNKLAR. Die Zusammenfassung enthält nur Fakten, keine Beleidigungen.
3. **Im Zweifel melden:** Ein Anhang ohne Text, eine unlesbare Mail oder eine nicht erreichbare KI → UNKLAR, mit Ton.

## Sicherheitsmechanismen
| Was passieren kann | Was dann passiert |
|---|---|
| GMX-Störung, falsches Passwort, Netzfehler, Programmfehler | sofort **❌ Fehleralarm** über Telegram **und** über den Wachdienst |
| GitHub fällt aus, Ablauf pausiert, Konto gesperrt, Telegram-Bot defekt | **Wachdienst healthchecks.io** meldet sich nach spätestens ~1 Std. **selbst** (eigener Telegram-Bot + E-Mail) |
| KI-Guthaben leer / KI gestört | ⚠️ Warnung mit Grund. Mails werden trotzdem gemeldet, die Stichwörter prüfen weiter |
| Telegram-Nachricht kommt nicht durch | wichtige Mail bleibt im Posteingang, Lauf schlägt fehl → Alarm |
| Mail landet im **Spamverdacht**, in **„Unbekannt“** oder durch einen Filter in einem anderen Ordner | alle Ordner werden durchsucht (außer Gesendet, Entwürfe, Papierkorb). Die Meldung sagt, wo sie lag |
| Er schreibt von einer **neuen Adresse** | wird über seinen Namen erkannt (`SENDER_NAMES`) → Meldung „🆕 neue Adresse“ |
| GitHub würde nach 60 Tagen pausieren | wird täglich automatisch verhindert |
| Alles still, aber trotzdem unsicher? | **täglich gegen 19 Uhr** stummes Lebenszeichen „✅ läuft“ |
| Neuer Text unter einem Zitat | wird erkannt. Alte zitierte Zeilen (`> …`) lösen keinen Fehlalarm aus |

Ehrliche Grenzen: Geprüft wird etwa alle 10 Minuten, zu Stoßzeiten verspätet sich GitHub auch mal. Schreibt er unter neuem Namen **und** neuer Adresse, kann ihn nichts automatisch erkennen.

---

## Einrichtung (am Handy, ca. 30–40 Min.)

> Für GitHub im Handy-Browser **„Desktop-Version“** einschalten, sonst fehlen Knöpfe.

### 1. GitHub-Konto
github.com → **Sign up** (kostenlos).

### 2. Eigene Kopie anlegen
**https://github.com/Autragskette/mail-waechter** → **Use this template → Create a new repository** → Name `mail-waechter`, **Public** → **Create repository**.
Bis die Zugangsdaten eingetragen sind, wartet der Wächter still.

### 3. GMX vorbereiten
GMX im Browser → **E-Mail → Einstellungen → POP3/IMAP Abruf → „Zugriff erlauben“** → Speichern.
Bei Zwei-Faktor-Anmeldung ein **App-Passwort** erzeugen und das verwenden.

### 4. Telegram-Bot
1. In Telegram **@BotFather** → `/newbot` → Name → Benutzername (endet auf `bot`).
2. Den **Token** kopieren.
3. Den neuen Bot öffnen → **Start** → „hallo“ schreiben.
4. Im Browser `https://api.telegram.org/bot<TOKEN>/getUpdates` öffnen → bei `"chat":{"id":` steht die **Chat-ID**.
5. Den Chat **anpinnen**, Benachrichtigungen für den Bot **nicht** stummschalten.

### 5. Wachdienst healthchecks.io (kostenlos, **Pflicht**)
1. **healthchecks.io** → Konto anlegen (kostenloser Tarif).
2. **Add Check** → Name „Mail-Wächter“ → **Period: 10 minutes**, **Grace: 1 hour** → speichern.
3. Die **Ping-URL** kopieren (`https://hc-ping.com/…`).
4. **Integrations → Telegram → Add Integration** → dem Healthchecks-Bot folgen und bestätigen. **E-Mail** bleibt zusätzlich aktiv.

So kommt eine Warnung auch dann, wenn GitHub oder der eigene Bot komplett ausfallen.

### 6. Claude-API-Schlüssel
**console.anthropic.com** → Konto → **Billing**: Guthaben kaufen (kleinster Betrag reicht), **Auto-Reload aus** → **API Keys → Create Key** → kopieren.
Ein Claude-Abo wird **nicht** gebraucht. Ohne Schlüssel läuft der Wächter kostenlos, aber nur mit Stichwörtern.

### 7. Secrets eintragen
Repo → **Settings → Secrets and variables → Actions → New repository secret**:

| Name | Inhalt |
|---|---|
| `GMX_USER` | ihre GMX-Adresse |
| `GMX_PASSWORD` | GMX-Passwort bzw. App-Passwort |
| `SENDER_ADDRESSES` | Mail-Adresse(n) des Vaters, mit Komma getrennt |
| `SENDER_NAMES` | sein Vor- und Nachname, wie er in Mails erscheint (z. B. `Max Mustermann`), mehrere mit Komma |
| `TELEGRAM_BOT_TOKEN` | Token aus Schritt 4 |
| `TELEGRAM_CHAT_ID` | Chat-ID aus Schritt 4 |
| `HEALTHCHECK_URL` | Ping-URL aus Schritt 5 |
| `ANTHROPIC_API_KEY` | Schlüssel aus Schritt 6 |

Secrets sind verschlüsselt und auch bei einem öffentlichen Repo für niemanden lesbar.

### 8. Startdatum
Gleiche Seite → **Variables → New repository variable** → `START_DATUM` = heutiges Datum, z. B. `08.11.2026`.
Ohne Startdatum werden auch alle alten Mails von ihm einsortiert.

### 9. Testen
**Actions** → (ggf. „enable“ antippen) → **Einrichtung testen → Run workflow**.
Nach ~1 Minute kommt in Telegram der Bericht 🔧 mit ✅/❌/⚠️ pro Punkt.

### 10. Echtheitstest (empfohlen)
Bei `SENDER_ADDRESSES` vorübergehend eine zweite eigene Adresse ergänzen, von dort „Er hat Fieber“ und „Loch in der Hose“ schicken, die Meldungen abwarten, danach die Testadresse wieder entfernen.

---

## Anpassen (`config.json`, im Repo antippen → Stift)
- `stichwoerter_immer_wichtig`: Gruppen von Wörtern, die **immer** Alarm auslösen.
  Standard: wird überall gefunden, auch im Wort. `^wort`: nur am Wortanfang. `=wort`: nur als ganzes Wort.
- `wichtig_wenn` / `belanglos_beispiele`: Regeln für die KI in normaler Sprache.
- Probelauf ohne Verschieben: Variable `DRY_RUN` = `1`, danach wieder löschen.

## Kosten
GitHub, Telegram und healthchecks.io sind kostenlos. Die Claude-API kostet pro Mail einen Bruchteil eines Cents. Gekauftes Guthaben verfällt nach einem Jahr.
