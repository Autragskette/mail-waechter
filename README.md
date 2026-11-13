# Mail-Wächter

Sortiert die Mails **eines bestimmten Absenders** im GMX-Postfach vor und meldet sich per Telegram.

| Kategorie | Was mit der Mail passiert | Nachricht aufs Handy |
|---|---|---|
| ⚠️ **WICHTIG** (Gesundheit, Notfall, Sicherheit, Übergabe, Schule, Behörden, Drohungen) | bleibt **ungelesen**, Ordner `Vater-Mails/Wichtig` | **mit Ton**, sachliche Kurzfassung |
| ❓ **UNKLAR** (KI unsicher, nur Anhang, KI gestört, neue Absenderadresse) | bleibt ungelesen, Ordner `Vater-Mails/Pruefen` | **mit Ton**, „bitte selbst ansehen“ |
| 📭 **BELANGLOS** (Kleidung, Vorwürfe, Gerede) | als gelesen markiert, Ordner `Vater-Mails/Belanglos` | **stumm**, eine Sammelnachricht pro Durchlauf |

Mails von anderen Absendern werden **nicht angefasst**. **Es wird nie etwas gelöscht.**

## Wie entschieden wird
**Mit KI (empfohlen):** Claude liest jede Mail komplett und entscheidet nach dem **Sinn**:
- Nur was das **gemeinsame Kind** (`KIND_NAME`) betrifft, kann wichtig sein. Alles über **andere Kinder** (`ANDERE_KINDER`) ist belanglos.
- Beleidigungen, Vorwürfe und Nebensächliches („für die Reha braucht er den Koffer“) sind belanglos.
- Wichtige Meldungen enthalten eine **sachliche Kurzfassung** ohne Beleidigungen.
- Stichwörter sind nur ein Hinweis an die KI. **Notfall-Netz:** Eindeutige Notfallwörter (Notarzt, Notaufnahme, 112, vermisst, Polizei …) machen eine Mail, die die KI für belanglos hält, trotzdem zu „bitte ansehen“, außer sie nennt nur ein anderes Kind. Abschaltbar mit `notfall_netz_aktiv: false`.
- Ein Anhang ohne Text → „bitte ansehen“, weil die KI Anhänge nicht lesen kann (`anhang_ohne_text_melden`).

**Ohne KI (kostenlos):** Die Stichwörter entscheiden. Ein Treffer bedeutet WICHTIG, außer es geht nur um ein anderes Kind oder es ist eine Beleidigung wie „geisteskrank“ (`beleidigungen_ignorieren`).

**Fällt die KI aus:** Die Stichwörter springen ein, Mails ohne Treffer kommen als „bitte ansehen“, dazu eine Warnung.

## Sicherheitsmechanismen
| Was passieren kann | Was dann passiert |
|---|---|
| GMX-Störung, falsches Passwort, Netzfehler, Programmfehler | sofort **❌ Fehleralarm** über Telegram **und** über den Wachdienst |
| GitHub fällt aus, Ablauf pausiert, Konto gesperrt, Telegram-Bot defekt | **Wachdienst healthchecks.io** meldet sich nach spätestens ~1 Std. **selbst** (eigener Telegram-Bot + E-Mail) |
| KI-Guthaben leer / KI gestört | ⚠️ Warnung mit Grund. Mails werden trotzdem gemeldet, die Stichwörter prüfen weiter |
| Telegram-Nachricht kommt nicht durch | wichtige Mail bleibt im Posteingang, Lauf schlägt fehl → Alarm |
| Mail landet im **Spamverdacht**, in **„Unbekannt“** oder durch einen Filter in einem anderen Ordner | alle Ordner werden durchsucht (außer Gesendet, Entwürfe, Papierkorb). Die Meldung sagt, wo sie lag (beim Spam nicht – dorthin darf man ihn per GMX-Filter bewusst umleiten, damit keine Vorschau aufs Handy kommt; dabei NICHT „Absender blockieren“ nehmen) |
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
| `KIND_NAME` | Vorname des gemeinsamen Kindes |
| `ANDERE_KINDER` | Vornamen ihrer anderen Kinder, mit Komma (deren Themen sind belanglos) |

Die Kindernamen stehen absichtlich nur in den Secrets und nicht im öffentlichen Code.

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

## Neue Version übernehmen
Actions → **„Aktualisieren“** → **Run workflow**. Holt die neueste Version aus der Vorlage, testet sie und übernimmt sie nur, wenn alle Tests bestehen. Die eigene `config.json` bleibt erhalten, außer man hakt „Auch config.json überschreiben“ an.

## Kosten
GitHub, Telegram und healthchecks.io sind kostenlos. Die Claude-API kostet pro Mail einen Bruchteil eines Cents. Gekauftes Guthaben verfällt nach einem Jahr.
