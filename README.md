# Mail-Wächter

Sortiert Mails eines bestimmten Absenders im GMX-Postfach vor und schickt eine kurze Telegram-Nachricht:

| Kategorie | Was passiert | Nachricht aufs Handy |
|---|---|---|
| ⚠️ **WICHTIG** (Gesundheit, Notfall, Übergabe, Schule, Behörden …) | bleibt **ungelesen**, Ordner `Vater-Mails/Wichtig` | mit Ton, kurze sachliche Zusammenfassung |
| ❓ **UNKLAR** (KI unsicher, verdächtiger Anhang) | bleibt ungelesen, Ordner `Vater-Mails/Pruefen` | mit Ton, „bitte selbst ansehen“ |
| 📭 **BELANGLOS** (Kleidung, Vorwürfe, Gerede …) | als gelesen markiert, Ordner `Vater-Mails/Belanglos` | **stumm**, nur „belanglos (Thema)“ |

**Es wird nie etwas gelöscht.** Alle Mails bleiben vollständig erhalten, zum Beispiel als Nachweis.

## Wie entschieden wird
1. **KI (Claude)** liest die Mail und ordnet sie anhand der Regeln in `config.json` ein. Die Zusammenfassung gibt nur Fakten wieder, keine Beleidigungen.
2. **Sicherheitsnetz:** Kommt ein Stichwort aus `stichwoerter_immer_wichtig` vor (z. B. *Fieber, Krankenhaus, Unfall*), ist die Mail **immer** WICHTIG, egal was die KI sagt.
3. Fällt die KI aus, wird sicherheitshalber UNKLAR gemeldet. Klappt Telegram nicht, bleibt die Mail im Posteingang und wird beim nächsten Lauf erneut versucht.
4. Zitierte alte Mails (`> …`, „Am … schrieb …“) werden ignoriert.

## Einrichtung (einmalig, ca. 15 Min.)

### 1. GMX: IMAP erlauben
GMX im Browser → **Einstellungen → POP3/IMAP Abruf → „Zugriff erlauben“** aktivieren.
Bei Zwei-Faktor-Anmeldung ein **anwendungsspezifisches Passwort** anlegen und das statt des normalen Passworts verwenden.

### 2. Telegram-Bot anlegen
1. In Telegram **@BotFather** öffnen → `/newbot` → Namen vergeben → du bekommst den **Bot-Token**.
2. Den neuen Bot öffnen und ihm „hallo“ schreiben (vom Handy deiner Freundin).
3. Chat-ID herausfinden:
   ```
   TELEGRAM_BOT_TOKEN=dein_token python3 mailwaechter.py --chat-id
   ```
   (oder im Browser `https://api.telegram.org/bot<TOKEN>/getUpdates` öffnen und `"chat":{"id":…}` ablesen)

### 3. Claude-API-Schlüssel (optional, empfohlen)
Auf console.anthropic.com einen API-Key anlegen. Kosten mit Haiku: Bruchteile eines Cents pro Mail.
Ohne Key wird nur nach Stichwörtern sortiert (alles ohne Treffer gilt dann als belanglos).

### 4. GitHub-Secrets setzen
Repo → **Settings → Secrets and variables → Actions → New repository secret**:

| Secret | Inhalt |
|---|---|
| `GMX_USER` | ihre GMX-Adresse |
| `GMX_PASSWORD` | GMX-Passwort bzw. App-Passwort |
| `SENDER_ADDRESSES` | Mail-Adresse(n) des Ex, mit Komma getrennt |
| `TELEGRAM_BOT_TOKEN` | Token von BotFather |
| `TELEGRAM_CHAT_ID` | Chat-ID aus Schritt 2 |
| `ANTHROPIC_API_KEY` | optional |

Die Adresse des Ex steht absichtlich nur in den Secrets, nicht im Code.

### 5. Testen
- Lokal ohne Postfach: `python3 mailwaechter.py --test tests/wichtig.eml`
- Probelauf ohne Verschieben/Senden: unter **Settings → Secrets and variables → Actions → Variables** die Variable `DRY_RUN` = `1` anlegen, dann **Actions → Mail-Wächter → Run workflow**. Danach die Variable löschen.

## Anpassen
Alles in `config.json`: was als wichtig/belanglos gilt, Stichwörter, Ordnernamen, Bezeichnung des Absenders.

## Hinweise
- **Öffentliches oder privates Repo?** Öffentliche Repos haben unbegrenzte Actions-Minuten. Der Code enthält keine persönlichen Daten, die Logs nur Zählerstände. Bei einem **privaten** Repo reichen die kostenlosen 2.000 Min./Monat nur für etwa einen Lauf alle 30 Minuten (`cron: "*/30 * * * *"`).
- GitHub pausiert geplante Abläufe in öffentlichen Repos nach **60 Tagen ohne Commit**. GitHub schickt vorher eine Mail; ein Klick auf „Enable workflow“ reicht.
- Die Mail-Inhalte gehen zur Einstufung an die Claude-API, die Zusammenfassung an Telegram.
