# Tfix-Nachbau

Info-App für einen kleinen Kreis von Tf, nachgebaut nach dem Startmenü des Originals:
Telefonbuch (mit Ril100-Suche und Kategorien), Bf/Strecke (Meine Bahnhöfe, Rabatte,
Abk./Ril100 mit optionaler Quellenangabe, Pausenräume mit Name/Ril100-Suche, Bahnhofspläne
mit Plan-Suche, Streckenbücher), Technik (Baureihen), Funktionen (Link-Liste zu
hilfreichen Seiten), DB Fernverkehr (Legende Dienstauftrag) und SOS (Notruf mit
Standort-SMS) sind bereits eingerichtet. Regelwerke, Zugfahrt, Weg/Zeit und Befehle sind
wie im Original ohne Funktion angelegt. Läuft als Web-App im Browser (Handy, Tablet, PC).

**Funktionen-Links selbst befüllen:** über Verwaltung → Funktionen-Links könnt ihr eigene
Links eintragen (Titel + URL). Zwei öffentlich bekannte Adressen zur Orientierung:
DB Navigator → `https://www.bahn.de/app`, Fahrplan → `https://www.bahn.de`. Für interne
Adressen wie RIS, Tf-Portal, DB Planet, DB Casino etc. kenne ich die genauen URLs nicht –
die trägt ihr am besten selbst ein.

**Bewusst nicht nachgebaut:** die "Bremsberechnung ICE" aus dem Original. Das ist eine
sicherheitsrelevante Berechnung – die gehört nur mit der offiziellen, geprüften Formel
aus dem Regelwerk rein, nicht als Nachbau aus Vermutung. Der Menüpunkt ist als Hinweis
angelegt, falls ihr die offizielle Grundlage später ergänzen wollt.

**Wichtig, falls du die Regeln schon einmal deployt hattest:** die aktualisierte
`firestore.rules` (mit den Collections `bahnhoefe`, `abkuerzungen`, `pausenraeume`,
`rabatte`, `funktionen`, `legende`) muss erneut in der Firebase-Konsole eingefügt/
veröffentlicht werden, sonst funktionieren diese Bereiche nicht.

## 1. Firebase-Projekt einrichten

1. Auf [console.firebase.google.com](https://console.firebase.google.com) ein neues Projekt anlegen.
2. **Authentication** aktivieren → Anmeldemethode "E-Mail/Passwort" einschalten.
3. **Firestore Database** anlegen (Produktionsmodus).
4. Unter "Projekteinstellungen" → "Meine Apps" eine **Web-App** hinzufügen. Die dabei
   angezeigten Config-Werte in eine neue lokale Datei `firebase-config.js` eintragen
   (Vorlage: `firebase-config.example.js` kopieren und umbenennen). `firebase-config.js`
   steht in `.gitignore` und wird nicht mit committet.
5. **API-Key einschränken** (empfohlen, auch wenn der Firebase-Web-Key kein klassisches
   Geheimnis ist – siehe Hinweis unten): Google Cloud Console → APIs & Services →
   Credentials → deinen Key auswählen → "Application restrictions" → "HTTP referrers" →
   eure GitHub-Pages-URL eintragen. Damit funktioniert der Key nur noch von eurer Domain aus.
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

## Hinweis zum "exposed secret"-Alert für den Google API Key

GitHub meldet den Firebase-`apiKey` standardmäßig als Secret, weil es generisch alle
Google-API-Keys so behandelt. Ein Firebase-**Web**-API-Key ist aber kein Bearer-Secret:
er identifiziert nur das Projekt gegenüber Google und ist dafür gedacht, im
Client-Code sichtbar zu sein. Zugriffsschutz für eure Daten kommt ausschließlich aus
`firestore.rules` (freigegebene Nutzer) und Firebase Authentication, nicht aus der
Geheimhaltung des Keys. Trotzdem sinnvoll, um den Alert loszuwerden und den Key sauber
zu halten:
- Key wie oben per HTTP-Referrer einschränken.
- `firebase-config.js` nicht committen (liegt in `.gitignore`, nur `firebase-config.example.js`
  mit Platzhaltern ist im Repo).
- Falls ein echter Key bereits in der Git-Historie eines öffentlichen Repos gelandet ist:
  in der Firebase-Konsole unter Projekteinstellungen einen neuen Web-API-Key erzeugen,
  `firebase-config.js` lokal aktualisieren, alten Key in der Google Cloud Console löschen.

## Hinweise

- Die Notruf-Funktion nutzt einen `sms:`-Link mit Standortlink (Google Maps) – das öffnet
  die SMS-App des Geräts mit vorausgefülltem Text, gesendet werden muss manuell final vom Nutzer.
  Das ist bewusst so gebaut, damit nichts automatisch ohne Bestätigung verschickt wird.
- Es sind keine echten DB-internen Daten (Kurzwahlen, Streckendetails) enthalten – die
  müsst ihr selbst einpflegen.
- Für den produktiven Einsatz: Firestore-Regeln und Freigabe-Workflow sind bewusst einfach
  gehalten (freigegeben-Flag statt komplexerem Rollensystem) und können bei Bedarf erweitert
  werden, ähnlich wie bei der Fahrgastzähler-/Kassenapp.
