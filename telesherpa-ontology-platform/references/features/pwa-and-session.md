# PWA & dauerhafte Anmeldung auf der Web-Plattform

Die Plattform läuft nicht nur in App und Browser, sondern auch als **installierbare PWA**
(Web-App auf dem Home-Bildschirm). Für Agenten ist das dann relevant, wenn ein Mensch
„ständig neu anmelden" meldet oder wenn Push-Nachrichten im Spiel sind.

## Für wen was gilt

| Kanal | Anmeldung | Push |
|---|---|---|
| **App (iOS/Android)** | native App-Anmeldung | APNs/FCM (App-Tokens) |
| **Browser (Web)** | Session-Cookie, **30 Minuten** | — |
| **PWA (Home-Bildschirm)** | Session-Cookie 30 min **+ Dauer-Anmeldung bis 14 Tage** | Web Push (VAPID) |
| **MCP / REST** | Bearer-Token 10 h + Refresh-Token 30 Tage | — |

## Die 30-Minuten-Session und warum sie absichtlich kurz ist

Die Web-Session läuft **30 Minuten** und wird bei jedem Aufruf verlängert. Danach greift die
Rechte-Prüfung und leitet auf die Login-Seite.
Das ist gewollt — auf einem geteilten Rechner soll eine offene Sitzung nicht tagelang gelten.

## PWA: bis 14 Tage eingeloggt bleiben

In der **installierten** PWA (Home-Bildschirm) gibt es keinen Adressbalken und keine
Lesezeichen — ein erzwungener Login ist dort teuer. Deshalb gilt dort zusätzlich:

```
Session abgelaufen
  → stiller Auto-Login (14 Tage)
      → wenn er greift: die angeforderte Seite erscheint direkt (kein Login-Screen)
      → wenn nicht: sichtbarer Login-Screen, kein Zwischenschritt
```

Bausteine:

- Die PWA meldet sich **clientseitig** als installiert (`display-mode: standalone` bzw.
  `navigator.standalone`) und schickt beim Login ein `pwa`-Kennzeichen mit.
- Serverseitig entsteht daraus eine **Dauersitzung** (14 Tage) plus ein PWA-Merker.
- Der stille Re-Login läuft **ohne Token-Rotation** — sonst würde der Token beim zweiten
  Aufruf gelöscht (der Browser übernimmt `Set-Cookie` aus einer `fetch()`-Antwort nicht).
- **Logout löscht beides** (Dauersitzung + PWA-Merker) — sonst gälte der nächste Login
  im selben Browser wieder als „aus der PWA".

**Wichtig für Agenten:** Es gibt KEINE serverseitige PWA-Erkennung am User-Agent. iOS
liefert aus der PWA denselben UA wie aus Safari. Wer eine Anmeldung „von außen" prüft, kann
deshalb nicht per UA unterscheiden, ob ein Aufruf aus der PWA kommt — das Kennzeichen muss
der Client mitschicken.

## Push-Nachrichten: nur im Home-Bildschirm (iOS)

Web Push funktioniert über VAPID und die Abos des Browsers — **auf iOS aber nur, wenn die
Seite vom Home-Bildschirm gestartet wurde.** Im Safari-Tab zeigt der Schalter nur einen
Hinweis. Für den Empfang muss die Web-App **nicht** laufen (Service Worker).

Die Abos liegen als JSON am Nutzer; der Versand läuft über die Benachrichtigungslogik.
Tote Abos (404/410 vom Push-Dienst) werden entfernt — sonst wachsen sie endlos.

## Was ein Agent daraus zieht

1. **Ein „nicht eingeloggt" im Web ist kein Rechteproblem.** Zuerst prüfen, ob es nur die
   abgelaufene 30-Minuten-Session ist (der Nutzer wird auf die Login-Seite geleitet).
2. **Der Agent selbst nutzt MCP/REST** — dort gilt der Bearer-Token (10 h) und danach der
   Refresh-Token (30 Tage). PWA-Mechanik betrifft ihn nicht.
3. **„Der Login-Screen kommt ständig"** in einer PWA: prüfen, ob die Seite wirklich installiert
   läuft (nicht im Tab) und ob der Nutzer sich zuletzt ausgeloggt hat — der Logout räumt die
   Dauer-Anmeldung bewusst mit weg.
4. **„Keine Push-Nachrichten auf dem iPhone"**: die Web-App muss auf dem Home-Bildschirm liegen.
   Im Tab gibt es keinen Push, unabhängig von Berechtigungen.
