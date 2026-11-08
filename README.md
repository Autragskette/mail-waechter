# Mail-Wächter

Sortiert die Mails **eines bestimmten Absenders** im GMX-Postfach vor und meldet sich per Telegram.

| Kategorie | Was mit der Mail passiert | Nachricht aufs Handy |
|---|---|---|
| ⚠️ **WICHTIG** (Gesundheit, Notfall, Übergabe, Schule, Behörden, Drohungen) | bleibt **ungelesen**, Ordner `Vater-Mails/Wichtig` | **mit Ton**, sachliche Kurzfassung |
| ❓ **UNKLAR** (KI unsicher, nur Anhang, KI nicht erreichbar) | bleibt ungelesen, Ordner `Vater-Mails/Pruefen` | **mit Ton**, „bitte selbst ansehen“ |
| 📭 **BELANGLOS** (Kleidung, Vorwürfe, Gerede) | als gelesen markiert, Ordner `Vater-Mails/Belanglos` | **stumm**, eine Sammelnachricht pro Durchlauf |

Alle anderen Mails (von anderen Absendern) werden **nicht angefasst**.
**Es wird nie etwas gelöscht.**

## Sicherheitsnetze
- **Stichwörter schlagen immer durch:** Steht z. B. *Fieber, Krankenhaus, Unfall, Polizei* in der Mail, ist sie WICHTIG, auch wenn die KI etwas anderes meint oder jemand versucht, die KI per Mailtext auszutricksen.
- **Im Zweifel wird gemeldet:** Ist die KI unsicher oder nicht erreichbar, enthält die Mail fast nur einen Anhang oder lässt sie sich nicht lesen, dann ist sie UNKLAR und es gibt eine Meldung mit Ton.
- **Keine stille Panne:** Bei jedem Fehler (Passwort falsch, GMX gestört …) kommt **❌ Fehleralarm** aufs Handy. Klappt die Telegram-Nachricht nicht, bleibt die wichtige Mail im Posteingang.
- **Tägliches Lebenszeichen gegen 19 Uhr** (stumm): „✅ Mail-Wächter läuft. Heute: …“ Kommt es mal nicht, bitte selbst ins Postfach schauen.
- **Schläft nicht ein:** GitHub würde den Ablauf nach 60 Tagen pausieren. Der Wächter verhindert das täglich selbst.
- Neuer Text **unter** einem Zitat wird erkannt. Alte, zitierte Zeilen (`> …`) lösen keinen Fehlalarm aus.

Grenzen: Die Prüfung läuft alle ~10 Minuten (kein Sekunden-Alarm). Schreibt er von einer **neuen Adresse**, wird die Mail nicht erkannt und liegt ganz normal im Posteingang. Dann die Adresse bei `SENDER_ADDRESSES` ergänzen.

---

## Einrichtung (alles am Handy möglich, ca. 20–30 Min.)

> Tipp: Für GitHub im Handy-Browser oben im Menü **„Desktop-Version“** einschalten, dann sind alle Knöpfe sichtbar.

### Schritt 1: GitHub-Konto (kostenlos)
github.com → **Sign up** → Konto anlegen.

### Schritt 2: Eigene Kopie des Wächters anlegen
1. Vorlage öffnen: **https://github.com/Autragskette/mail-waechter**
2. Grüner Knopf **„Use this template“ → „Create a new repository“**
3. Name z. B. `mail-waechter`, **Public** auswählen (kostenlos und unbegrenzt; es liegen dort keine persönlichen Daten) → **Create repository**

Ab jetzt gehört alles ihr. Die Vorlage braucht sie danach nicht mehr.

### Schritt 3: GMX vorbereiten
GMX im Browser → **E-Mail → Einstellungen → POP3/IMAP Abruf** → **„Zugriff erlauben“** aktivieren → Speichern.
Nutzt sie die Zwei-Faktor-Anmeldung, dort ein **App-Passwort** erzeugen und das statt des normalen Passworts nehmen.

### Schritt 4: Telegram-Bot anlegen
1. Telegram installieren (falls noch nicht da).
2. Kontakt **@BotFather** öffnen → `/newbot` → Namen eingeben (z. B. `Mail Wächter`) → Benutzernamen eingeben (muss auf `bot` enden, z. B. `mein_mailwaechter_bot`).
3. BotFather schickt einen **Token** (sieht aus wie `123456789:ABC…`) → kopieren und sicher aufheben.
4. Den neuen Bot öffnen → **Start** → „hallo“ schreiben.
5. Im Browser aufrufen (Token einsetzen):
   `https://api.telegram.org/bot<TOKEN>/getUpdates`
   Dort steht `"chat":{"id":123456789` → diese Zahl ist die **Chat-ID**.

### Schritt 5: Claude-API-Schlüssel (für die KI-Sortierung)
1. **console.anthropic.com** → Konto anlegen.
2. **Billing / Plans & Billing** → Guthaben kaufen (kleinster Betrag reicht). **Auto-Reload ausgeschaltet lassen**, dann kann nie mehr abgebucht werden.
3. **API Keys → Create Key** → Schlüssel kopieren (wird nur einmal angezeigt).

Ein Claude-Abo (Pro) wird dafür **nicht** gebraucht. Die API wird separat und nach Verbrauch abgerechnet.
Ohne Schlüssel funktioniert der Wächter auch, dann **kostenlos**, aber nur mit Stichwörtern. Alles ohne Stichwort gilt dann als belanglos.

### Schritt 6: Zugangsdaten als „Secrets“ eintragen
In ihrem Repo: **Settings → Secrets and variables → Actions → New repository secret**, sechsmal:

| Name (genau so schreiben) | Inhalt |
|---|---|
| `GMX_USER` | ihre GMX-Adresse |
| `GMX_PASSWORD` | GMX-Passwort bzw. App-Passwort |
| `SENDER_ADDRESSES` | Mail-Adresse des Vaters (mehrere mit Komma) |
| `TELEGRAM_BOT_TOKEN` | Token aus Schritt 4 |
| `TELEGRAM_CHAT_ID` | Chat-ID aus Schritt 4 |
| `ANTHROPIC_API_KEY` | Schlüssel aus Schritt 5 |

Secrets sind verschlüsselt. Niemand kann sie lesen, auch nicht bei einem öffentlichen Repo.

### Schritt 7: Startdatum setzen (empfohlen)
Damit nicht alle alten Mails auf einmal gemeldet werden:
gleiche Seite → Reiter **Variables → New repository variable** → Name `START_DATUM`, Wert = **heutiges Datum**, z. B. `08.11.2026`.
(Ohne diese Variable werden auch alle älteren Mails von ihm aus dem Posteingang einsortiert, je 40 pro Durchlauf.)

### Schritt 8: Einrichtung testen
Repo → Reiter **Actions** (falls ein Knopf „I understand … enable“ erscheint: antippen) → links **„Einrichtung testen“** → **Run workflow** → **Run workflow**.
Nach ~1 Minute kommt in Telegram der Bericht 🔧 mit ✅ oder ❌ pro Punkt. Bei ❌ steht dabei, was zu tun ist.

**Fertig.** Ab jetzt läuft der Wächter alle 10 Minuten von selbst. Das Handy muss dafür nicht an sein, das läuft bei GitHub.

### Optional: Echtheitstest mit eigener Mail
Bei `SENDER_ADDRESSES` vorübergehend eine zweite eigene Adresse ergänzen (`vater@…, test@…`), von dort eine Mail „Er hat Fieber“ und eine „Loch in der Hose“ schicken und die Nachrichten abwarten. Danach die Testadresse wieder entfernen.

---

## Anpassen
In `config.json` (im Repo antippen → Stift-Symbol):
- `wichtig_wenn` / `belanglos_beispiele`: Regeln für die KI in normaler Sprache
- `stichwoerter_immer_wichtig`: Wörter, die **immer** Alarm auslösen (Wortanfang genügt: `verletz` trifft *verletzt, Verletzung*)
- `ordner`, `absender_bezeichnung`

Probelauf ohne Verschieben: Variable `DRY_RUN` = `1` anlegen, danach wieder löschen.

## Kosten
- GitHub: kostenlos (öffentliches Repo)
- Telegram: kostenlos
- Claude-API: nach Verbrauch, pro Mail ein Bruchteil eines Cents. Gekauftes Guthaben verfällt nach einem Jahr.
