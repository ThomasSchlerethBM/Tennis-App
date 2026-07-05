# U9 Kleinfeld Live-Tracker - Tennis App

Eine kostenlose, eigenständige Web-App zum Live-Erfassen, Verfolgen und Berichten von
U9-Kleinfeld-Tennis-Mannschaftswettkämpfen (Bayerischer Tennis-Verband, Sommer 2026),
nach den Regeln der offiziellen BTV-Ausschreibung.

Läuft komplett im Browser, ohne Installation, ohne App Store — einfach einen Link öffnen.
Alle Geräte (Eltern, Trainer, Zuschauer, Oberschiedsrichter) sehen in Echtzeit denselben
Stand, weil die Daten über Firebase Realtime Database synchronisiert werden.

---

## Inhalt

- [Was die App kann](#was-die-app-kann)
- [Technischer Aufbau](#technischer-aufbau)
- [Einrichtung / Deployment](#einrichtung--deployment)
- [Die fünf Tabs im Überblick](#die-fünf-tabs-im-überblick)
- [Admin-Bereich](#admin-bereich)
- [Links & QR-Codes](#links--qr-codes)
- [Regel-Logik im Detail](#regel-logik-im-detail)
- [Bekannte Einschränkungen](#bekannte-einschränkungen)
- [Fehlerbehebung](#fehlerbehebung)

---

## Was die App kann

- **Live-Punkterfassung** für 4 Einzel + 2 Doppel nach BTV-Kleinfeld-Regeln (Kurzsätze bis 4,
  No-Ad, Satz-Tiebreak bei 4:4, Match-Tiebreak bei Satzgleichstand)
- **Motorik-Wettkampf** (Koordinationsslalom, Zielwurfstaffel, Einbeinsprung) mit Stoppuhr
  oder manueller Sieger-Wahl
- **Mehrgeräte-Echtzeit-Sync** über Firebase — jedes Gerät mit dem Link sieht sofort denselben
  Stand, ganz ohne manuelles Aktualisieren
- **Live-Übersicht** im TV-Broadcast-Stil (reine Zuschauer-Ansicht, kein Bearbeiten möglich)
- **Automatischer Spielbericht** im Layout des offiziellen BTV-Formulars, druckbar/als PDF
  exportierbar, inkl. digitaler Unterschriften (Finger/Maus) für Oberschiedsrichter sowie
  Heim- und Gast-Mannschaftsführer
- **3-Spieler-Kader** möglich — Einzel/Doppel, die Position 4 benötigen, werden automatisch
  als w.o. gewertet und verrechnet
- **Positions-Hilfe**: drehbare Court-Grafik zeigt, wer gerade aufschlägt/returniert (diagonal,
  mit Seitenwechsel-Umschaltern für unterschiedliche Blickwinkel)
- **Popups** für 40:40 (No-Ad-Regel) und Seitenwechsel
- **QR-Codes & Teilen-Links** für: ein einzelnes Spiel (nur dieses Spiel bearbeitbar), die
  Live-Übersicht (nur Zuschauen), und die Oberschiedsrichter-Unterschrift (kein Passwort nötig)
- **Admin-Passwortschutz** (Standard: `1234`) für Setup, Motorik und Bericht
- **Meldeliste-Import**: komplette Vereins-/Rangliste (bis 30 Spieler) einfügen, per Dropdown
  auf Position 1–8 zuordnen

## Technischer Aufbau

Bewusst **ohne Build-Schritt** gehalten (gleiches Muster wie andere Projekte des Autors):

| Baustein | Technologie |
|---|---|
| UI | React 18 (UMD-Build, `React.createElement`, "classic" JSX runtime) |
| JSX-Transformation | Babel Standalone, im Browser zur Laufzeit |
| Datenhaltung | Firebase Realtime Database (Echtzeit-Sync über alle Geräte) |
| QR-Codes | `qrcodejs` (client-seitig, ohne externen Bilddienst) |
| Styling | reines CSS (keine Frameworks), Google Fonts (Oswald + Inter) |
| Hosting | GitHub Pages (eine einzige `index.html`) |

Alles steckt in **einer Datei** (`index.html`). Kein `npm install`, kein Bundler — die Datei
lädt React/Babel/Firebase per CDN-Skript-Tag und transformiert ihren eigenen App-Code
(`<script type="text/plain" id="app-source">`) zur Laufzeit über `Babel.transform(...)`
mit erzwungenem `runtime: "classic"` (wichtig: der neue automatische JSX-Runtime würde ein
`import`-Statement erzeugen, das als normales `<script>` nicht ausführbar ist).

### Datenstruktur in Firebase

Jede "Kategorie" liegt unter einem eigenen Schlüssel (kein einzelner großer Datenblock),
damit zwei Geräte, die gleichzeitig an unterschiedlichen Spielen schreiben, sich nicht
gegenseitig überschreiben:

```
u9-meta            → Vereinsnamen, Liga, Termin, Bemerkungen, Unterschriften (Base64-PNG)
u9-players          → Aufstellung Heim/Gast (Positionen 1–8)
u9-playerpool       → komplette Meldeliste je Verein (bis 30 Namen)
u9-clubs            → Vereinsverzeichnis (für Dropdown-Auswahl)
u9-doubles          → Doppel-Paarungen (welche Positionen spielen zusammen)
u9-motorik          → Zeiten/Weiten + Sieger je Übung
u9-match-e1 … e4    → Einzel 1–4 (Live-Stand + Punkteverlauf)
u9-match-d1 … d2    → Doppel 1–2 (Live-Stand + Punkteverlauf)
```

**Wichtige Firebase-Eigenheit:** leere Arrays/Objekte werden beim Speichern nicht abgelegt
(RTDB lässt den Schlüssel dann einfach weg). Die App normalisiert das beim Einlesen
(`normalizeLive`, `normalizeMatch`), damit z.B. ein leeres `history`-Array nicht zu einem
Absturz führt.

## Einrichtung / Deployment

1. **Firebase-Projekt anlegen** unter [console.firebase.google.com](https://console.firebase.google.com)
   → Realtime Database aktivieren (Region z.B. `europe-west1`)
2. **Config eintragen**: In `index.html` ganz oben den `firebaseConfig`-Block mit den eigenen
   Werten aus *Projekteinstellungen → Ihre Apps → SDK-Snippet* füllen
3. **Security Rules** setzen (Firebase Console → Realtime Database → Regeln):
   ```json
   { "rules": { ".read": true, ".write": true } }
   ```
   (offen für den Anfang; kann später mit eigenen Regeln eingeschränkt werden)
4. **Hosten**: `index.html` in ein GitHub-Repository legen → Settings → Pages →
   "Deploy from a branch" → `main` / `root` → fertig. URL folgt dem Muster
   `https://DEIN-NUTZERNAME.github.io/DEIN-REPO/`
5. Admin-Passwort ändern: im Code nach `ADMIN_PASSWORD` suchen (Standard `"1234"`)

**Wichtig:** Die Datei muss als **echte Webseite** geöffnet werden (lokal per Doppelklick
oder über die gehostete URL) — eine Vorschau innerhalb einer Chat-/KI-Oberfläche blockiert
die externen Netzwerkaufrufe zu Firebase/CDN und zeigt dann einen Fehler.

## Die fünf Tabs im Überblick

| Tab | Zugriff | Zweck |
|---|---|---|
| **Live** | öffentlich | Reine Zuschauer-Ansicht im Broadcast-Stil, nichts editierbar |
| **Spiele** | öffentlich | Liste aller 6 Spiele, hier wird tatsächlich live gepunktet |
| **Setup** | 🔒 Admin | Vereine, Spieler, Meldeliste, Doppel-Aufstellung, Reset |
| **Motorik** | 🔒 Admin | Die 3 Motorik-Übungen erfassen |
| **Bericht** | 🔒 Admin | Automatisch generierter Spielbericht + Unterschriften + Druck |

Ausführliche Schritt-für-Schritt-Anleitung siehe **Info-Tab in der App** bzw. die separate
PDF-Anleitung.

## Admin-Bereich

- Passwort-geschützt (Standard `1234`), gilt für Setup/Motorik/Bericht
- Einmal freigeschaltet bleibt ein Gerät angemeldet, bis über "Admin abmelden" (im Setup-Tab)
  ausgeloggt wird
- **Gefahrenzone** (Setup-Tab, ganz unten): "Alles zurücksetzen" löscht Mannschaften, Spieler,
  Motorik-Ergebnisse und alle 6 Spielstände unwiderruflich für **alle** Geräte — nur für den
  Start eines neuen Spieltags gedacht, mit doppelter Sicherheitsabfrage
- Admins können zusätzlich **jedes einzelne Spiel** im Spiele-Tab separat zurücksetzen

## Links & QR-Codes

Drei Arten von Direktlinks (jeweils mit "🔗 Link" + "▦ QR-Code"-Button in der App):

| Link-Muster | Zweck | Passwort nötig? |
|---|---|---|
| `#match=e1` (o.ä.) | Nur dieses eine Spiel bearbeiten, kein Zugriff auf den Rest | Nein |
| `#live` | Nur die Live-Übersicht ansehen (Zuschauer) | Nein |
| `#osr` | Nur Name + Unterschrift des Oberschiedsrichters | Nein |

Alle drei Links funktionieren nur zuverlässig, wenn alle dieselbe gehostete URL öffnen
(z.B. die GitHub-Pages-Adresse), nicht irgendeine Kopie der Datei.

## Regel-Logik im Detail

- **Zählweise**: Kurzsätze bis 4 Spiele, No-Ad (bei 40:40 entscheidet der nächste Punkt),
  Tiebreak bis 7 bei 4:4 in Spielen, Match-Tiebreak bis 10 bei Satzgleichstand 1:1 — exakt
  wie in der offiziellen Ausschreibung beschrieben
- **Aufschlagrotation**: wechselt automatisch nach jedem Spiel zwischen den Teams; im
  Tiebreak nach der Standard-Regel (1 Punkt, danach alle 2 Punkte)
- **Seitenwechsel**: automatisches Popup nach ungerader Gesamt-Spielzahl im Satz bzw. alle
  6 Punkte im Tiebreak
- **3-Spieler-Kader**: reduziert eine Mannschaft ihren Kader auf 3 Spieler (Setup → "–
  Spieler"), werden Einzel/Doppel, die Position 4 benötigen, automatisch als w.o. gewertet —
  außer es wird trotzdem ein Ergebnis eingetragen (z.B. mit Ersatzspieler)
- **Punkte gesamt**: Tennis max. 12 (2 je gewonnenem Match), Motorik max. 6 (2 je Übung,
  1 bei Gleichstand)

## Bekannte Einschränkungen

- Kein automatischer Datenabruf von der BTV-Website möglich (Cross-Origin-Sperre der
  BTV-Server) — Meldelisten/Ranglisten müssen manuell kopiert und eingefügt werden
  (die App erkennt Namen und Rang automatisch aus dem eingefügten Text)
- Sponsoren-Logos (ESB, BTV, Tennis Point, Dunlop, Generali/DVAG etc.) werden im
  Spielbericht bewusst nur als **Text** angezeigt, nicht als Grafik — das sind fremde
  Markenzeichen, die nicht ohne Weiteres nachgebildet werden sollten
- Spieler-ID-Nummern werden nicht erfasst (nur Namen) — das BTV-Portal trägt diese beim
  Einlesen ohnehin automatisch ein
- Bei zwei Geräten, die **exakt gleichzeitig** dasselbe Spiel bearbeiten, gilt "der letzte
  Schreibvorgang gewinnt" (Firebase-Standardverhalten) — für ein einzelnes Match sollte
  idealerweise ein Gerät federführend punkten

## Fehlerbehebung

- **Weißer Bildschirm / Fehlermeldung im roten Kasten**: Screenshot der angezeigten
  Fehlermeldung machen (die App zeigt jetzt Fehler sichtbar an, statt nur zu verschwinden)
- **"Firebase konnte nicht geladen werden"**: Die Datei wurde vermutlich in einer
  Vorschau-/Sandbox-Umgebung geöffnet statt als echte Webseite — Datei lokal per Doppelklick
  oder über die gehostete URL öffnen
- **Permission-Denied-Fehler beim Speichern**: Firebase Security Rules prüfen (siehe oben)

---

*Erstellt für den Bayerischen Tennis-Verband, Region Südbayern, U9 Kleinfeld Sommer 2026.*

