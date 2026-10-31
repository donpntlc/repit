# REPIT. – Projektkontext für den KI-Assistenten (Cursor)

> **Für Dean, so benutzt du diese Datei:**
> 1. Öffne in Cursor das Repo **donpntlc/repit** (über „Clone Repository“).
> 2. Leg diese Datei als `AGENTS.md` in den Hauptordner des Repos. Cursor liest sie dann automatisch bei jedem Chat.
> 3. Starte den ersten Chat mit: *„Lies AGENTS.md und index.html, dann sag mir kurz, ob du alles verstanden hast.“*

---

## 0. Wie du mit Dean arbeitest
- Antworte immer auf **Deutsch**, kurz, locker und praktisch. Dean ist Personal Trainer (St. Gallen, Schweiz), kein Programmierer. Erkläre Schritte so, dass er sie ohne Fachwissen versteht.
- Er diktiert oft per Sprache. Tippfehler wie „Rapid“ = REPIT, „Soul“ = SOL.
- **Lösche nie seine Trainingsdaten.** Ändere nie die localStorage-Schlüssel `hantelbuch.*` (sonst sind alle Daten der Nutzer weg).
- Nach jeder Änderung: in der Daten-Ansicht die Versionsnummer hochzählen (`Version TT.MM. HH:MM` in `sData()`), damit Dean auf dem Handy sieht, ob die neue Version da ist. **Immer die echte aktuelle Uhrzeit (Zürich) nehmen, keine Zeit in der Zukunft.** Die Update-Prüfung vergleicht nur auf Ungleichheit, zurückstellen ist also unproblematisch.
- Teste jede Änderung in einer Handy-Ansicht (390 × 844), bevor du pushst.

## 0b. Arbeitsweise (so hat Claude bisher mit Dean gearbeitet, bitte genauso)
1. **Erst kurz sagen, was du machst, dann direkt machen.** Nicht lange planen oder nachfragen, wenn die Aufgabe klar ist. Nur bei echten Unklarheiten **eine** kurze Frage.
2. **Selbst testen, bevor du fertig meldest:** im Browser in der Handy-Ansicht 390 × 844 öffnen, Screenshot anschauen, die Funktion mit Testdaten durchspielen und auf JS-Fehler in der Konsole achten. Nie „sollte gehen“ sagen, ohne es geprüft zu haben.
3. **Danach selbst deployen** (commit + push) und prüfen, ob die neue Versionsnummer live ist (`https://donpntlc.github.io/repit/`).
4. **Antworten kurz und auf Deutsch:** was jetzt neu ist (Liste mit fett markierten Stichworten), wo man es in der App findet, und was Dean tun muss (meist: App ganz schliessen und neu öffnen).
5. **„Nur merken“-Modus:** Wenn Dean sagt „merk dir“, „bin im Training“ oder „nur merken“, baust du **nichts**. Trag den Wunsch unten in Abschnitt 5 (Backlog) dieser Datei ein und bestätige in 1 bis 2 Sätzen.
6. **Design-Regeln:** schwarz/gold, linksbündig, sauber. Keine Pop-up-Dialoge (`alert`/`confirm`), sondern die eigenen Sheets (`openSheet({type:'confirm',…})`). Nach jeder Änderung die Versionsnummer in `sData()` hochzählen.
7. **Mitdenken:** Ist etwas schon vorhanden (z. B. einseitiges Training mit Wechselpause), sag ihm, wo er es findet, statt es neu zu bauen. Fallen dir Fehler auf, behebst du sie gleich mit.
8. Kein Fachchinesisch. Erkläre Dinge so, wie man es einem Trainingskunden erklären würde.

## 1. Was REPIT ist
- Trainings-Tracker als **Web-App (PWA)** fürs iPhone, Ersatz für die App „Strong“. Früher hieß sie „Hantelbuch“, deshalb tragen interne Namen noch „hantelbuch“.
- Live: **https://donpntlc.github.io/repit/** (GitHub Pages, Repo `donpntlc/repit`, Branch `main`).
- Name/Logo: „REP“ weiß + „IT.“ gold. Slogan: **„Rep it. Track it. Repeat.“** Auf dem Homescreen heißt sie „REPIT.“ (mit Punkt).
- Design: schwarz mit Gold. Farben stehen als CSS-Variablen in `:root` (`--bg #0b0b0c`, `--gold #c9a24a`, `--gold-hi #e7c66d`, `--ok`, `--bad`, `--muted` …). Schriften: Barlow Condensed (Titel), Manrope (Text).
- Jeder Nutzer hat seine Daten **nur auf seinem Handy** (kein Server, keine Konten). Teilen geht über Dateien und Links.

## 2. Technik und Aufbau
- **Eine einzige Datei: `index.html`** (rund 170 KB, HTML + CSS + JS in einer Datei, kein Framework, kein Build). Dazu kommen `manifest.webmanifest`, Icons (`icon-180/192/512.png`) und der Ordner `img/`.
- Deploy: `index.html` ändern → commit → **push auf `main`** → GitHub Pages ist nach ca. 1 Minute live. Auf dem iPhone muss man die App eventuell ganz schließen und neu öffnen.
- Lokal testen: `python3 -m http.server` im Repo-Ordner, dann `http://localhost:8000` (für die Handy-Größe die DevTools nutzen).

### Code-Struktur (Abschnitte in `index.html`, Kommentar `/* ---------- … ---------- */`)
| Abschnitt | Inhalt |
|---|---|
| Übungsbibliothek | `LIB` (`[name, kategorie]`), `CATS`, `EXDESC` (Beschreibung: `[ausführung, hauptmuskeln, mit, „so lernst du es“]`), `IMG_ALIAS` (Strong-Namen → Bild) |
| State | globales Objekt `S` (tab, routines, workouts, body, active, sheet, custom, gear, goal …) |
| Speicherung | `LS='hantelbuch.v1'`, `lsSave()`, `saveMeta()`, `saveWorkout()`, `saveActive()`. **Zwei Modi:** `refs` gesetzt = Claude-Artifact-Cloud. Auf GitHub ist `refs` null, dann gilt localStorage. Fotos liegen in IndexedDB (`idbGet/idbSet`, Schlüssel `ph:local-…`). |
| Render | `renderAll / renderNav / renderMain / renderSheet`. Ansichten: `vRoutines, vHistory, vTraining, vActive, vExercises, vBody`. Tabs (`TABS`): Routinen, Verlauf, **Training (Mitte)**, Übungen, Körper |
| Sheets | untere Einblendungen, Map `{picker, menu, routine, hist, ex, confirm, summary, rest, data, setrest, rpe, order, rorder, tpl, client, intro, share, planin, pay, rday, gender, gear}` → Funktionen `sXxx()`. Öffnen mit `openSheet({type:'…'})`, schließen mit `closeSheet()`, Zurück-Stapel in `S.sheet.back` |
| Aktionen | **ein** Objekt `ACT` mit allen Klick-Handlern. Buttons rufen sie über `data-act="name" data-v="…" data-i="…"` auf. Eingabefelder laufen über `data-in="…"` (Input-Handler mit `k==='…'`-Zweigen) |
| Rest timer / Toast / Sound | Pausentimer, `toast()`, eigener Timer-Sound, iPhone-Kurzbefehl „REPIT Timer“ |
| Fotos | Formcheck-Fotos mit Datum |
| Strong-Import | `parseCSV`, `importStrong`, `routinesFromImport`. `STRONG_CUTOFF` gilt nur, wenn schon importierte Trainings da sind (Deans Oktober-Schnitt) |
| Pläne teilen (Coach) | `planPayload`, `sShare` (+ Gewichtsrechner `calcKg/calcFactor`), `takePlan`, `readPlan`, `shareOut`, Links `#plan=` (`checkPlanHash`) |
| Backup | `backupOut()` (JSON inkl. Fotos als Base64), `backupIn()` (führt zusammen statt zu überschreiben). **Ein** Knopf „Daten importieren“ (`#anyIn` → `importAny`) erkennt selbst: Backup, Coach-Plan (`app:'hantelbuch'`) oder CSV |
| Abo-Stufen | vorbereitet, **`PAYWALL=false`** (alles frei). `TIERS` free/pro/coach, `need()`, `needSlot()`, `sPay()`. Sperren hängen per `wrap()` vor den ACT-Handlern |

### Bilder
- `img/<slug>-m.jpg` (Mann) und `img/<slug>-w.jpg` (Frau), `slug()` aus dem Übungsnamen. `exImg(name)` nutzt `IMG_ALIAS` und `baseName()`.
- Wahl Mann/Frau: `hantelbuch.gender`.

### Wichtige localStorage-Schlüssel (nie umbenennen)
`hantelbuch.v1` (Hauptdaten), `.gender`, `.intro`, `.pomode` (Hinweise: full/short/off), `.rlabel` (num/day), `.exmode` (Übungsliste: cat/az/oft), `.clients`, `.warmex`, `.tier`, `.sound`, `.soundRaw`, `.iostimer`, `.lockbell`

### Datenmodell (kurz)
- Routine: `{id, name, note, exercises:[{name, target:'10-12', sets, rest, note, plan:[{kg,reps,rpe,rest,type}]}], folder?:'coach', client?, day?:'Mo'…}`
- Training: `{id, name, start, routineId, note, exercises:[{name, target, note, sets:[{kg, reps, rpe, done, type:'n'|'w', rest}]}]}`
- Varianten: `"Dips · Aufwärmen"`. `baseName()` / `variantOf()` trennen am „ · “, jede Variante hat ihren eigenen Verlauf.
- Anstrengung wird im Feld `rpe` gespeichert: `'8'` = Stufe 1, `'9'` = 2, `'9.5'` = 3, `'10'` = 4 (`effLvl()`, `effBadge()`).

## 3. Deans Trainingssystem (Produktlogik, bitte so beibehalten)
1. **REPIT-Idee:** keine festen Tage. Die Routinen laufen im Kreis 1 → 2 → 3 … und dann wieder von vorne. Fällt ein Tag aus, kommt einfach das nächste Training. Die App markiert es golden mit „Als Nächstes“ (`nextRid()`). Feste Wochentage sind **optional** (Umschalter „1 · 2 · 3 | Wochentage“, dann wird das Training von heute mit „Heute“ markiert).
2. **Progressive Overload pro Satz:** Jede Übung hat einen Zielbereich (z. B. 10–12). **Jeder Satz wird einzeln bewertet:** War ein Satz letztes Mal über dem oberen Ziel, wird **nur in diesem Satz** das Gewicht erhöht (`incKg`, wieder bei der unteren Zahl starten). Die anderen Sätze: +1 Wdh. War der Satz mit Stufe 2 bewertet: gleiche Wdh., aber sauber. Logik in `setGoals()`, `applyGoals()` (trägt die Ziele beim Start einer Routine ein) und `poHint()`. Aufwärmsätze (`type:'w'`) und Aufwärmübungen (`isWarmEx`) werden ignoriert.
3. **Hinweise in 3 Stufen** (Einstellung in Daten): Anfängerhilfe (ausführlich) · Trainingshinweis (eine Zeile) · Aus.
4. **Anstrengung in 4 Stufen statt RPE:** 1 Noch Luft · 2 Noch 1, aber unsauber · 3 Nur noch halb · 4 Muskelversagen. Die Farben (grün, gold, orange, rot) erscheinen als kleine Hochzahl in der Spalte „Vorher“. Grundsatz: **saubere Ausführung vor Gewicht.**
5. **Einführung beim ersten Start** (`INTRO`, 3 Seiten, später erneut über „So funktioniert REPIT“).
6. **Coach-Bereich:** Routinen → „Für Kunden (Coach)“, eine Rubrik pro Kunde (`client`). „Plan teilen“ verschickt als Datei oder Link, optional mit Gewichten und Notizen und mit Gewichtsrechner (Mann/Frau, Größe, Gewicht, Level).
7. **6er-Split-Vorlage** (`SPLIT6`, `applySplit`) mit Levels Anfänger/Fortgeschritten/Profi, skaliert nach Körpergewicht (Basis: Dean ≈ 80 kg).
8. Weitere Funktionen: Übungen nach Rubriken / A–Z / Häufigste, Kampfsport-Drills unter Athletik, Geräte-Einstellungen pro Übung, Notizen (die letzte sichtbar, ältere aufklappbar), Kraftwert (geschätztes 1RM mit Erklärung), Körper mit Gewicht, Start-/Zielgewicht samt Ziellinie und Formcheck-Fotos.

## 4. Geschäftsmodell (vorbereitet, noch nicht aktiv)
- Kostenlos: 2 eigene Trainingstage. **Pro: 20 / Jahr oder 3 / Monat.** **Coach: ca. 80 / Jahr** (Kunden-Rubriken, ganze Splits teilen, Rechner). Einzelne Tage teilen kann jeder.
- Für echtes Bezahlen fehlen noch Konten, ein Zahlungsanbieter (z. B. Stripe) und ein Server (Supabase ist bei Dean verbunden). Bis dahin bleibt `PAYWALL=false`.

## 5. Offene Aufgaben (REPIT)
- Bilder (macht Dean mit ChatGPT, dann ab nach `img/`):
  - Frauen-Bilder prüfen (lockeres T-Shirt ohne Ausschnitt, lange lockere Jogginghose).
  - **Überkopf-Trizeps neu:** langes Seil, Kabel **ganz unten** eingestellt.
  - Twisted Curls, Deans eigene Übungen (Chest Press Band, Cable Crunch Sideways, Decline Crunch, Shoulder Warm Up Cable, Bicep Curl Machine, Triceps Extension, Cable Pulldown, Stairmaster …), die Kampfsport-Drills.
  - Muskelgruppen-Bilder für Routinen (Legday, Push, Pull …).
- Später: Konten und Sync (Coach sieht Kundentrainings, Pläne kommen automatisch an), Paywall aktivieren.

### Backlog vom 17.10. – erledigt (live seit 19.10., APPV „19.10. 10:30“)
Anstrengung Stufe 2 „Letzte unsauber“, Erinnerung im Anfängermodus, Gewichtsschritte pro Gerät, Timer-Ton (vorher/ersetzen, auch im Reiter Routinen), Supersatz, Wochenbericht mit Export, Dragon Flag, Split-Wechsel A/B im Wochentakt, kcal pro Training, Training nachträglich bearbeiten inkl. Endzeit, automatische Update-Prüfung. Offen sind nur noch die Bild-Wünsche (Reverse Fly Cable überkreuzt, Überkopf-Trizeps, Dragon Flag, Seitheben am Kabel in Deans Variante: Rolle leicht unter Hüfthöhe, Kabel hinter dem Rücken, ca. 30° schräg zum Turm).

### Backlog vom 19.10. – erledigt (live seit 26.10., APPV „26.10. 12:45“)
Anstrengung 1–4 neu (Zu leicht · Letzte unsauber · Nur halb geschafft · Alles gegeben), Zirkel mit 3+ Übungen („Zur Gruppe davor hinzufügen“), Pause nur nach der letzten Übung einer Gruppe (Ansicht bleibt stehen, bis Pause vorbei/„Weiter“), Aufwärm-Markierung (blau) im Training und Routinen-Editor, „Deficit Deadlift (Trap Bar)“ inkl. Umbenennung im Verlauf (Migration `tb1`), Check-in vor/nach Training (Schalter in Daten, Schlüssel `hantelbuch.checkin`, Felder `food`/`feel` am Training, Auswertung im Wochenbericht).

### Erledigt 29.10. (APPV „29.10. 17:45“)
Stufe 4 heißt „Alles gegeben | Muskelversagen“, Zahl im ausgewählten Kreis lesbar, „Ziel heute“-Hinweis je nach Fall (alle Stufe 2 → gleiche Wdh. sauber; gemischt → Satz für Satz).

### Noch offen (REPIT)
- **Sprache Englisch** in den Einstellungen (alle Texte, Übungen, Einführung, Hinweise).
- Bild „Deficit Deadlift (Trap Bar)“ (Trap Bar, erhöht auf Platte) – nutzt bis dahin das Kreuzheben-Bild.

## 5b. Strategie / Fahrplan (Deans Entscheidung)
- **Wichtig (29.10.):** Die zusammengeführte App (**SOL-GYM**, siehe 5c) wird eine **eigene, separate App**. Die einzelnen Apps (PREPIT bzw. SOL Planer und REPIT) bleiben **einzeln nutzbar**. **Jede Änderung muss immer in allen betroffenen Versionen gemacht werden** (Einzel-App und zusammengeführte App) → am besten gemeinsamer Code/gemeinsame Module, damit nichts auseinanderläuft.
1. **Erst Web-App:** REPIT und PREPIT laufen vorerst als **Web-Apps** (GitHub Pages, aufs iPhone über „Zum Home-Bildschirm“). Kein App Store, solange die Funktionen noch wachsen. So geht jede Änderung sofort live.
2. **Dann verknüpfen:** PREPIT (Ernährung) und REPIT (Training) werden zu **einer App „SOL-GYM“** zusammengeführt, mit gemeinsamen Daten (z. B. Trainings-kcal → Tagesdefizit) und einem gemeinsamen Abo.
3. **Später App Store,** wenn beides stabil läuft. Dazu kommen Konten, Sync und Bezahlen.
- **Offene Technik-Frage, vor dem Start mit Dean klären:** PREPIT ist **Flutter**, REPIT ist **eine HTML-Datei**. Mögliche Wege: REPIT in Flutter nachbauen, PREPIT als Web-App neu bauen, oder vorerst zwei Web-Apps mit gemeinsamem Startbildschirm und Datenaustausch. Schlag ihm die Optionen kurz und als Rangliste vor und entscheide nicht allein.

## 5c. SOL-GYM (Deans Wunsch vom 29.10., gilt ab jetzt)
- **Namen:** Training = **REPIT**, Ernährung = **PREPIT** (bisher „SOL Tracker“), gemeinsame App = **SOL-GYM**. Slogan-Idee: „Prep it – Track it“.
- SOL-GYM enthält beide Bereiche zum Auswählen (REPIT / PREPIT), **inhaltlich 1:1 wie die Einzel-Apps**. Die Einzel-Apps bleiben bestehen.
- **Gebaut 29.10. (Variante „Hülle“, von Dean gewählt):** Live unter **https://donpntlc.github.io/repit/sol-gym/** (Ordner `sol-gym/` im Repo `repit`). Oben Umschalter PREPIT / REPIT, darunter laufen die echten Apps unverändert in je einem Rahmen (`../../sol-tracker/` und `../`). Gleiche Adresse (donpntlc.github.io) → gemeinsamer Speicher innerhalb von SOL-GYM. Letzter Bereich wird gemerkt (`solgym.tab`). Version `SOLGYM_V` in `sol-gym/index.html`. Ein eigenes Repo `sol-gym` konnte Claude nicht anlegen; kann Dean später machen und den Ordner umziehen.
- **Einheitlicher Look (30.10.) – abgelehnt und zurückgesetzt:** Versuch, PREPIT auf REPIT-Schriften (Manrope + Barlow Condensed) umzustellen. Dean fand die schmale Schrift in PREPIT zu „gequetscht“ und wollte die alte Optik zurück. Auf den Sicherungsstand zurückgesetzt (Commit `cb7318d`). **Barlow Condensed nicht mehr in PREPIT einsetzen.** Für einen neuen Versuch vom gesicherten Stand ausgehen und eher bei REPIT ansetzen oder nur kleine gemeinsame Merkmale (Logo, Gold-Akzente) angleichen.
- **Sicherung vor dem Design-Umbau (30.10.):** Branch `sicherung-30-10` in `repit`, Branches `sicherung-30-10-source` (Quellcode) und `sicherung-30-10-live` (fertige Web-App) in `sol-tracker`. Zurück geht so: den Sicherungs-Stand auf `main` bzw. `source` zurückspielen. Tags lassen sich über den Proxy nicht pushen, darum Branches.
- **Zweiter Versuch (30.10. nachmittags):** nur kleine Erkennungszeichen. PREPIT: Logo „PREP“ + „IT.“ (gold) über dem Datum in der Tracking-Kopfzeile, alte Schrift bleibt (Commit `1091a59`). REPIT: feine Goldlinie (50 % Gold) über die ganze Breite unter dem Kopf, wie in PREPIT (APPV „30.10. 15:20“). Sicherung des Stands davor: `sicherung-30-10-nachmittag` (repit), `sicherung-30-10-nachmittag-source` und `-live` (sol-tracker).
- **SOL-GYM ohne obere Leiste (30.10. abends, Deans Wunsch):** Die Umschalt-Leiste ist weg (nahm zu viel Platz). Gewechselt wird per **Tipp aufs Logo oben links** in der jeweiligen App; daneben ein goldener Wechsel-Pfeil (⇄). Nur in SOL-GYM aktiv (Erkennung `window.top !== window.self`); die Einzel-Apps bleiben unverändert. Technik: App schickt `parent.postMessage('solgym:switch')`, SOL-GYM zeigt den anderen Bereich. REPIT: Block `/* SOL-GYM */` vor `Render`, Klasse `gymmode` blendet „Nur auf diesem Gerät gespeichert“ aus. PREPIT: `lib/sol_gym.dart` (+ `_web`/`_stub`), Logo in der Tracking-Kopfzeile (nur dort). Startseite mit Auswahl bleibt, mit Hinweis „Wechseln: oben links aufs Logo tippen“. Sicherung davor: `sicherung-30-10-abend` (repit), `sicherung-30-10-abend-source` und `-live` (sol-tracker).
- **GESAMT-Ansicht + Daten-Umzug (30.10. spät):** Logo-Tipp öffnet jetzt eine Auswahl **REPIT · PREPIT · GESAMT** (statt direkt umzuschalten). GESAMT ist Teil von `sol-gym/index.html` (kein eigenes Repo), liest nur: REPIT `hantelbuch.v1` (kcal über `wKcal` aus dem REPIT-Rahmen), PREPIT `flutter.tracked_foods_v1` / `weight_history_v1` / `goals_v1` (doppelt JSON-kodiert). Inhalt: Zeitraum 7/30/90 Tage, 4 Kennzahlen, Kalorien pro Tag mit Ziellinie und Trainingstagen, Gewicht (PREPIT vor REPIT), bewegtes Gewicht pro Woche, Vergleich Trainingstag vs. Ruhetag, Anleitung zum Umzug. PREPIT hat neu **Einstellungen → Daten sichern & umziehen** (JS `solDataExport/solDataImport` in `pages/index.html`, sichert alle `flutter.*`-Schlüssel als `PREPIT-Backup-*.json`, Laden ersetzt und lädt neu). Getestet: Export → Import in leere App klappt.
- **PREPIT-Look für REPIT + SOL-GYM (31.10. nachts):** Je ein abschaltbarer CSS-Block „PREPIT-Look“ am Ende der Styles in `index.html` und `sol-gym/index.html`: Roboto statt Barlow/Manrope, Überschriften normal geschrieben mit kurzem Goldstrich darunter (wie „Mahlzeiten:“), Karten #17171a mit 14 px Rundung, hellere Grautöne (#a39e92), Labels ohne Grossbuchstaben, aktiver Tab fett. Dean: Kopfzeile so lassen (neue Version gefällt). REPIT APPV „31.10. 00:30“. Sicherung davor: Branch `sicherung-31-10-vor-look` (repit, enthält auch SOL-GYM/Gesamt).
- **Logos (31.10. nachts):** PREPIT-Kopfzeile im REPIT-Look: goldenes Besteck-Symbol im Kästchen (38 px, wie REPIT-Hantel) + „PREP“/„IT.“ 22 px w900, darunter Datum klein (Commit `d15b63b`, Sicherung davor `sicherung-31-10-vor-logo-source`/`-live`). SOL-GYM-Logo: **„Sol“ in Schreibschrift Great Vibes** (Ersatz auf iPhone: Snell Roundhand) + „GYM“ in Gold, auf Startseite und in Gesamt (SOLGYM_V „31.10. 00:45“, Sicherung davor `sicherung-31-10-vor-schrift`). App-Symbol (icon-*.png) noch altes „SOL / GYM“, Schriftdatei war in Claudes Umgebung nicht ladbar.
- **Update Logos (31.10. ca. 01:15):** Schreibschrift verworfen. SOL-GYM nutzt jetzt das **SOL-Tracker-Monogramm** (`sol-gym/sol-mark.png`, Ränder weich ausgeblendet) + „SOL“/„GYM“ in **Cinzel** wie im SOL Tracker; App-Symbol (`sol-gym/icon-*.png`) = SOL-Tracker-Icon. Schriftzug oben in allen Apps kleiner: 18 px, Kästchen 32 px, Slogan 10 px. PREPIT: Schriftzug in eigener Logo-Schrift `SolLogo` (Roboto Black, `assets/fonts/Roboto-Black.ttf`, geholt von `scripts/logo_schrift_holen.sh` im Build), Masse durch `kWebUiScaleEff` gegen die 88-%-Web-Verkleinerung ausgeglichen; zweite Zeile „PREP IT. TRACK IT.“, bei anderem Tag das Datum in Gold; Kopf-Symbole kompakt. Sicherungen: `sicherung-31-10-vor-sol-logo` (repit), `sicherung-31-10-vor-schriftzug-source` (sol-tracker).
- **Logo-Tipp in PREPIT hing (31.10. morgens, behoben):** Flutter verschluckt auf Touch-Geräten oft den ersten Tipp. SOL-GYM hängt sich deshalb selbst an das PREPIT-Dokument (`hookPrep`, gleiche Adresse) und öffnet die Auswahl bei kurzem Tipp oben links (x < 200, y < 54) sofort. Doppelte Nachricht von Flutter wird 900 ms ignoriert (`DIRECT`), der Geister-Klick auf den Hintergrund 450 ms nach dem Öffnen auch (`OPENED`). SOLGYM_V „31.10. 09:50“. Sicherung davor: `sicherung-31-10-vor-tippfix`.
- **31.10. Vormittag:** App-Symbole von SOL-GYM heissen jetzt `sol-gym/sol-icon-*.png` (neuer Name, damit das iPhone das SOL-Tracker-Logo neu lädt; Sicherung `sicherung-31-10-vor-icon`). Schriftzug „SOL-GYM“: Cinzel 800, eng (.02em), kleiner goldener Strich (`.dash`), GYM gleich gross in Gold. Kopfleisten einheitlich **49 px** wie PREPIT: REPIT `header.top` (margin -22/-13, Logo mittig, links bündig mit PREPIT), Gesamt `#all .top`. REPIT APPV „31.10. 10:25“, SOLGYM_V „31.10. 10:25“.
- **31.10. Mittag:** Logo-Monogramm + App-Symbole aus dem **scharfen Original** auf Deans Mac (`~/SOL/Logo-Konzept/Generated_image3.png`, 1376×768; Quadrat 304,0–1072,768). Dateien `sol-gym/sol-mark.png` (600 px) und `sol-gym/solgym-icon-{180,192,512}.png` (wieder neuer Name gegen iPhone-Cache). Sicherung davor `sicherung-31-10-vor-scharfes-logo`. Schrift „SOL-GYM“: Dean findet Cinzel zu weich, will **kantiger, Richtung römische Schrift**. Vergleichsseite mit 6 Varianten: `https://donpntlc.github.io/repit/sol-gym/schrift.html` (Castoro Titling, Cormorant SC, Cinzel 600, Marcellus SC, Forum, Cinzel Decorative) – Auswahl offen.
- **31.10. 12:00:** SOL-GYM-Schriftzug jetzt **Cormorant SC 700** (Deans Wahl Nr. 2) mit leichtem 3D-Look: Verlaufsfüllung (hell → dunkler, GYM in Goldverlauf) + Prägeschatten per `filter: drop-shadow`. Sicherung davor `sicherung-31-10-vor-cormorant`. Vergleichsseite `sol-gym/schrift.html` bleibt online.
- **31.10. 12:45 Meissel-Look:** Dean will den 3D-Stil des Monogramms (Mittelkante pro Strich, eine Seite hell, eine im Schatten). Umgesetzt mit SVG-Filtern `#chisel` (Startseite) und `#chisel-s` (klein, Gesamt-Kopf) in `sol-gym/index.html`: Blur der Buchstabenform als Höhenkarte + `feDiffuseLighting`/`feSpecularLighting` (Licht von links oben, azimuth 235). Flache Farben #f4ead2 (SOL) / #d4ad58 (GYM). Sicherung davor `sicherung-31-10-vor-meissel`. **Dean: gefällt nicht → zurückgesetzt (Revert), es gilt wieder Cormorant SC mit leichtem 3D-Verlauf von 12:00. Meissel-/Lichtfilter nicht mehr vorschlagen.** 13:00: GYM-Goldverlauf unten heller (#f8e6b0 → #e9c977 → #d6b05c → #c29a47), Dean fand das Braun unten zu dunkel.
- **PREPIT Live-Aktualisierung (31.10.):** Offene Fenster zeigten Änderungen erst nach Schliessen/Öffnen (Flutter baut Modal-Fenster bei `setState` des Hauptscreens nicht neu). Fix: `_MainScreenState` hat `_liveRev` (ValueNotifier), `setState` ist überschrieben und zählt hoch; die Mahlzeit-Liste („Getrackt: …“, `_openMealTrackedListPopup`) hört per `ValueListenableBuilder` darauf und rechnet `_updateMealTotals()` neu. Neue Fenster mit Live-Daten bitte genauso einbinden. Commit `9384a65`, Sicherung `sicherung-31-10-vor-live-update-source`.
- **Umzugs-Anleitung (31.10. 13:40):** In SOL-GYM eine Vollbild-Anleitung `#guide` („Daten in SOL-GYM holen“), geöffnet über den Link auf der Startseite („Daten aus REPIT & SOL Tracker holen ›“) und unten in Gesamt (`[data-guide]`). Je 4 Schritte für REPIT (alte App: Verlauf → oben rechts Daten → Backup speichern → In Dateien sichern; SOL-GYM: Verlauf → Daten importieren) und PREPIT (alte SOL-Tracker-App neu öffnen → Einstellungen → Daten sichern (Datei); SOL-GYM: Einstellungen → Daten laden (Datei) → Datei wählen). Hinweise: danach nur noch SOL-GYM; PREPIT-Laden ersetzt, REPIT ergänzt. Sicherung davor `sicherung-31-10-vor-umzug-anleitung`.
- **Neues SOL-Logo (31.10. 14:45):** Dean hat mit ChatGPT eine schärfere Version erstellt (1678×937, per Chat geschickt). Quadrat um das Monogramm (x 362–1299, y 0–937) → `sol-gym/sol-mark.png` (720 px) und App-Symbole `sol-gym/sol-v2-icon-{180,192,512}.png` (neuer Name gegen iPhone-Cache). Logo-Grösse angepasst (`.hmark` 168 px, `.lg .mk` 40 px), weil das Monogramm mehr vom Bild füllt. Sicherung davor `sicherung-31-10-vor-neues-logo`.
- **Diagramme in Gesamt (31.10. 15:05):** Ein Tipp auf einen Balken/Punkt lässt die Auswahl stehen (vorher verschwand sie beim Loslassen, man musste gedrückt halten). `pointerleave` setzt nur noch bei Maus zurück. Sicherung davor `sicherung-31-10-vor-tipp-auswahl`.
- **Startseite (31.10. 15:20):** Reihenfolge der Karten REPIT · PREPIT · GESAMT (wie in der Auswahl beim Logo-Tipp). Sicherung davor `sicherung-31-10-vor-reihenfolge`. Erklärbilder (8 Karten, Willkommen/iPhone/Wechseln/Gesamt/Umzug/Coach 1+2/Kunde) wurden für Dean erstellt.
- **Defizit in Gesamt (31.10. 18:30):** Neuer Bereich „Defizit“ in `sol-gym/index.html`: Defizit pro Tag = eingespart (PREPIT-Bedarf − gegessen) + Training (REPIT), gestapelte Balken (`defChart`, Überschuss rot nach unten), Ziellinie = Phasen-Ziel kcal/Tag (`flutter.phase_goal_v1`), KPIs Ø Defizit/Tag und ≈ Fett (÷ 7000). „Wochen-Check mit der Waage“: pro Woche erwartet (kg) vs. Wochenschnitt-Gewicht zur Vorwoche (grün ±0.3 kg), ab 2 Wochen „Bedarf laut Waage“. Bedarf kommt aus dem neuen PREPIT-Schlüssel `sol_gym_bridge_v1` ({maint, goal, mode, at}, geschrieben in `_saveAppState`), Fallback Ziel + Phasen-Defizit. Sicherung davor `sicherung-31-10-vor-defizit` (repit), `sicherung-31-10-vor-schritte-defizit-source`/`-live` (sol-tracker).
- **PREPIT 31.10. abends:** Advanced-Modus hat Schalter „Schritte anzeigen“ (Einstellungen → Advanced, Schlüssel `advanced_show_steps_v1`, Getter `_showStepsCard`) → Schritte-Karte wie im Aggressiv Cut. Texte vereinfacht: „Letzte Mahlzeit: wie viel?“ (Normaler Anteil / Alles, was übrig ist / Etwas für Snacks übrig lassen) und „Tag auffüllen“ (ein Satz).
- **Fortschritt statt Defizit (31.10. 19:45):** Der Bereich heisst jetzt „Fortschritt“, oben Umschalter **Kalorien | Gewicht** (`solgym.dtab`). Kalorien: zweiter Umschalter **Tag | Woche** (`solgym.dper`), je grosse Zahl + Aufteilung Ernährung/Training, Woche = Mo–So (`chDW`, Ziellinie = Ziel × 7). Gewicht: grosse Zahl, Ziel-Chip, Kurve mit Ziellinie + Prognose bis Zieldatum (`goalChart`, wie PREPIT Phasen-Ziel). Darunter ein Abgleich „Laut Kalorien / Laut Waage“ mit einem Satz; Wochentabelle + Bedarf laut Waage hinter „Details pro Woche“. Alter Bereich „Gewicht“ entfernt. Sicherung davor `sicherung-31-10-vor-defizit-umbau`.
- **PREPIT 31.10. ca. 19:30:** Diät-Sortierung im Hinzufügen-Fenster und in der Datenbank = **wenigste kcal pro 1 g des gewählten Makros** (Protein/KH/Fett/Ballaststoffe; < 1 g/100 g ans Ende), Zeile zeigt „· x kcal/g“. Geschmacksfilter **Süß & salzig · Süß · Salzig** im neuen Hinzufügen-Fenster zurück (`_pickerFlavor`, ging beim Umbau vom 30.10. verloren). Sicherungen `sicherung-31-10-vor-diaet-sortierung-source`, `sicherung-31-10-vor-geschmack-source`.
- **REPIT Kopfzeile fix (31.10. 20:55):** `header.top` ist `position:sticky` (Hintergrund per box-shadow/clip-path auf volle Breite), `html` overflow hidden, **`body` ist der Scroll-Container** (`overscroll-behavior-y:none`) → kein Mitwackeln der ganzen Seite, gilt Einzel-App + SOL-GYM. `window.scrollTo` ersetzt durch `pageScroll()` (scrollt `document.body`). Neue Scroll-Aufrufe bitte immer über `pageScroll()`. APPV „31.10. 20:55“. Sicherung davor `sicherung-31-10-vor-fixe-kopfzeile`.
- **Kein Seitwärts-Scrollen (31.10. 19:58):** `body` hat `overflow-x:hidden`, `.wrap` `overflow-x:clip`, Goldlinie `.top::after` ohne 100vw-Trick (left:-50vw;width:200vw, wird abgeschnitten). Seite scrollt nur noch hoch/runter; Chip-Reihen (`.cats`) scrollen weiter seitlich. Kein `touch-action` auf body setzen (würde seitliche Chip-Reihen blockieren).
- **iPhone-Hinweis:** Jedes Home-Bildschirm-Icon hat einen eigenen Speicher. Daten aus dem einzelnen REPIT-Icon einmal per Backup → „Daten importieren“ in SOL-GYM übernehmen. Echter Abgleich erst mit Konten (Supabase).
- **Danach (Verknüpfung):**
  1. **Trainingsbewertung:** Am Ende jedes Trainings eine Bewertung 1–3 (schlecht · normal · sehr gut). Auswertung zusammen mit der Ernährung und dem Wohlbefinden: Was habe ich an Tagen mit sehr gutem Training gegessen?
  2. **Trainings-kcal im Defizitbalken:** REPIT schätzt die verbrannten kcal (Übungen, Sätze, Cardio, später Schritte). Diese fliessen in den grossen Ziel-/Defizitbalken von PREPIT ein (z. B. „noch 60 Tage“ → näher ans Ziel). Ein Überschuss aus der Ernährung wird wieder abgezogen. Verlauf sichtbar.

## 6. PREPIT (bisher SOL Tracker) / Zusammenführung
- Ernährungs-App von Dean: Repo **`donpntlc/macro_tracker`** (Flutter), lokal `~/macro_tracker-1`. Web-Build mit `./web_bauen.sh` (baut die App und teilt sie in Upload-Pakete auf). In `lib/main.dart` gibt es `_webZoom` mit `kWebUiScale = 0.88`, damit die Web-Version weniger eng wirkt. **Dieser Build ist noch nicht ausgeführt und hochgeladen.**
- Geplant: **eine App „SOL-GYM“** mit zwei Bereichen: **PREPIT** (Ernährung, links) und **REPIT** (Training). Ein gemeinsames Abo für beide (kostenlos / Monat / Jahr).
- Design-Wunsch: **feine Trennlinien zwischen einzelnen Lebensmitteln und Rezepten** in Listen.
- **Kalorien aus dem Training:** Standardmässig **nicht** zum Essensbudget dazurechnen. Nur das Gesamtdefizit des Tages zeigen, am Wochenende das Wochendefizit in Fett umgerechnet (7000 kcal ≈ 1 kg). Optionaler Schalter: Trainings-kcal zum Tagesbudget addieren (für den Alltag ohne Diät oder Aufbau).
- **Erledigt 26.10.:** keine Nachkommastellen, Einkaufslisten + „Füllen“ (bis Zielmitte, in jeder Mahlzeit unter den Balken) zurückgeholt, Trennlinien in der Datenbank. Quellcode jetzt in **`donpntlc/sol-tracker`, Branch `source`**; jeder Push baut automatisch (GitHub Action) und veröffentlicht nach `main` (Pages). Fehlerlog bei Build-Fehlern im Branch `build-logs`.
- **Noch offen PREPIT:** Trainings-kcal manuell eintragen + Schalter „zum Budget addieren“ (Wochen-Fettäquivalent 7000 kcal gibt es schon im Wochen-Recap), Mahlzeiten vorplanen und abhaken.
- **Keine Nachkommastellen im Tracking:** Gramm, kcal und Prozent immer ganzzahlig gerundet anzeigen (z. B. „851 / 2234–2306 kcal“, „44 / 213 g“, „21 %“). Gilt für Tagesübersicht, Makros, Mahlzeiten, Zielmitte usw. Intern darf weiter genau gerechnet werden.
- **Mahlzeiten vorplanen und abhaken:** Mahlzeiten für heute/die nächsten Tage im Voraus eintragen (Lebensmittel + Menge, auch „Plan kopieren“ von gestern/Vorlage). Geplante Einträge erscheinen ausgegraut mit Haken-Kästchen und zählen noch nicht. Abhaken = gegessen → zählt ins Tracking. Menge beim Abhaken noch änderbar. Vorschau „Wenn du alles isst: X kcal / P / K / F“. Optional: Einkaufsliste aus dem Plan füllen.
- Es gibt außerdem die App „SOL Planer“ (`sol_day_planner`).
- **Zwei verschwundene Funktionen in SOL Tracker wiederherstellen** (sehr wichtig für Dean). Ursache gefunden: das Repo wurde von `main` auf den älteren Branch `master` umgestellt. Der vollständige Stand (Stash vom 21.06.) ist als Branch **`restore-sol`** gesichert. Genaue Schritte stehen in **`CURSOR_RESTORE.md`** im Repo:
  1. **Einkaufsliste**
  2. **„Füllen“:** Man tippt auf ein Makro bzw. auf kcal (Kohlenhydrate, Kalorien, Eiweiss, Fett …) und dann auf „Füllen“. Die App füllt bis zur Zielmitte auf und sagt, wie viel Gramm man vom gewählten Lebensmittel noch nehmen kann.
