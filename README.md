# Tfix-Nachbau

Info-App für einen kleinen Kreis von Tf: Kontakte (Kurzwahlen), Notruf mit Standort-SMS,
Strecken-/Baureihen-Infos. Läuft als Web-App im Browser (Handy, Tablet, PC).

## 1. Firebase-Projekt einrichten

1. Auf [console.firebase.google.com](https://console.firebase.google.com) ein neues Projekt anlegen.
2. **Authentication** aktivieren → Anmeldemethode "E-Mail/Passwort" einschalten.
3. **Firestore Database** anlegen (Produktionsmodus).
4. Unter "Projekteinstellungen" → "Meine Apps" eine **Web-App** hinzufügen. Die dabei
   angezeigten Config-Werte in `firebase-config.js` eintragen.
5. Die Datei `firestore.rules` in der Firebase-Konsole unter Firestore → Regeln einfügen
   und veröffentlichen (oder per Firebase CLI deployen: `firebase deploy --only firestore:rules`).

## 2. Ersten Admin anlegen

1. In der Firebase-Konsole unter Authentication einen Nutzer mit E-Mail/Passwort anlegen
   (das ist dein eigenes Konto).
2. In Firestore die Collection `benutzer` anlegen, darin ein Dokument mit der **UID**
   dieses Nutzers als Dokument-ID (UID steht in der Authentication-Liste):
   ```
   benutzer/{UID}
     email: "deine@mail.de"
     freigegeben: true
     admin: true
   ```
3. Danach kannst du dich in der App anmelden und über den Tab "Verwaltung" → "Nutzer"
   weitere Kollegen freischalten, sobald die sich einmal mit ihrem (in Authentication
   angelegten) Konto angemeldet haben.

Für jeden weiteren vertrauten Kollegen: Konto in Authentication anlegen (E-Mail + Passwort
mitteilen), er meldet sich einmal an (wird zunächst abgewiesen, da nicht freigegeben),
danach erscheint er in der Verwaltung und du schaltest ihn frei.

## 3. Daten befüllen

Kontakte, Strecken und Baureihen werden direkt in der App unter "Verwaltung" gepflegt
(nur für freigegebene Nutzer sichtbar/bearbeitbar). Es sind absichtlich keine Beispieldaten
enthalten – die trägt ihr selbst ein.

## 4. Deployment (GitHub Pages)

1. Dieses Verzeichnis in ein GitHub-Repo pushen (privates Repo empfohlen, da die
   Firebase-Config darin liegt – der Zugriffsschutz erfolgt aber ohnehin über die
   Firestore-Regeln, nicht über Geheimhaltung der Config).
2. Repo-Einstellungen → Pages → Branch auswählen, auf dem `index.html` liegt.
3. Die von GitHub ausgegebene URL ist danach die App-Adresse für dich und deine Kollegen.

## Hinweise

- Die Notruf-Funktion nutzt einen `sms:`-Link mit Standortlink (Google Maps) – das öffnet
  die SMS-App des Geräts mit vorausgefülltem Text, gesendet werden muss manuell final vom Nutzer.
  Das ist bewusst so gebaut, damit nichts automatisch ohne Bestätigung verschickt wird.
- Es sind keine echten DB-internen Daten (Kurzwahlen, Streckendetails) enthalten – die
  müsst ihr selbst einpflegen.
- Für den produktiven Einsatz: Firestore-Regeln und Freigabe-Workflow sind bewusst einfach
  gehalten (freigegeben-Flag statt komplexerem Rollensystem) und können bei Bedarf erweitert
  werden, ähnlich wie bei der Fahrgastzähler-/Kassenapp.
