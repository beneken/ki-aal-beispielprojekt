# Component User Model (Nutzermodell)

Hält ein **internes, einfaches Modell des Nutzers**: seine **Präferenzen** (was er mag und wie er angesprochen werden will), seine **Gewohnheiten** (wiederkehrende Muster im Alltag) und seine **Fähigkeiten** (*Capabilities*: Sehen, Hören, Kognition, Motorik, Sprechen). Das Modell sagt den anderen Komponenten, **wie sie sich auf diesen Menschen einstellen** sollen, nicht, was er erlebt hat (das ist Gedächtnis) und nicht, wer der Assistent ist (das ist Persönlichkeit).

Status: Entwurf vom 2026-10-22, noch nicht abgestimmt. Mit *(Annahme)* markierte Punkte sind Vorschläge. Verwandt: `C-06-prefrontal-cortex.md` (Hauptnutzer des Modells), `C-07-personality.md` (Stil des Assistenten; Gegenstück), `C-03-semantic-memory.md` (Fakten über den Nutzer), `C-05-amygdala.md` (Gefühlslage, flüchtig), `docs/produktsteckbrief.md` (Zielgruppe, Anrede, UI/UX-Leitlinien).

**Datenmodell:** Das Nutzermodell wird als **eigener Knoten `Nutzermodell` am `Person`-Knoten des Nutzers** gespeichert *(Annahme)*, nicht als lose Fakten im semantischen Gedächtnis. Grund: Es hat ein festes Schema, Werte mit Herkunft und braucht gezieltes Einsehen, Ändern und Löschen. `docs/spec/datenmodell.puml` ist entsprechend zu erweitern.

## Ziel und Abgrenzung

- Der Assistent soll sich **auf den Nutzer einstellen**: langsamer sprechen, wenn er schlecht hört; Erinnerungen wiederholen, wenn er vergisst; Vorlieben beachten, ohne dass er sie jedes Mal sagen muss.
- Das Modell ist **klein und erklärbar**: wenige feste Dimensionen mit wenigen Stufen, keine Scores aus Black-Box-Verfahren. Jeder Wert lässt sich in einem Satz begründen und vom Nutzer korrigieren.
- Es ist **kein Diagnoseinstrument.** Es hält fest, **was im Gespräch hilft**, nicht, was medizinisch zutrifft. „Hört schlecht“ ist eine Gesprächsanpassung, keine Befundung. Der Assistent stellt keine Diagnosen und spricht Einschränkungen nicht ungefragt an.
- **Nicht** Teil des Modells: Persönlichkeit des Nutzers (weder erfragt noch geschätzt, siehe `C-07`), Gefühlslage (flüchtig, `C-05`), Erlebnisse (`C-01`), Gesprächsinhalte, Diagnosen und Medikamente (gehören, wenn überhaupt, ins semantische Gedächtnis mit Einwilligung).

| Komponente | Rolle gegenüber dem Nutzermodell |
|---|---|
| Dialogmanager (`C-06`) | **Hauptleser.** Holt das Modell zu Sitzungsbeginn, setzt den **Anpassungsblock** in den Prompt und nutzt die **Anpassungsparameter** (Tempo, Lautstärke, Antwortlänge, Wiederholung, Wartezeit). |
| Persönlichkeit (`C-07`) | Das Nutzermodell kann Stilparameter **begrenzen** (z. B. `satzlaenge` höchstens `KURZ` bei kognitiver Einschränkung), nie erweitern. Das Persönlichkeitsprofil selbst ändert es nicht. |
| Hippocampus (`C-02`) | Kein Schreibzugriff auf Fähigkeiten. Aus Gesprächen entstehen höchstens **Vorschläge** (siehe „Lernen aus dem Gespräch“). |
| Semantisches Gedächtnis (`C-03`) | Liefert Fakten (Hobbys, Familie); das Nutzermodell hält **Einstellungen**, nicht Erinnerungen. Überschneidungen werden nicht dupliziert *(offen)*. |
| Amygdala (`C-05`) | Kein Schreibzugriff; ihre Stimmung ist Sitzungsdaten. |
| Einrichtende / Pflege | Dürfen beim Einrichten Startwerte setzen (mit Einwilligung des Nutzers). |
| Nutzer | Sieht und ändert das Modell jederzeit per Sprache („Was weißt du über mein Hören?“, „Sprich langsamer“, „Vergiss das“). |

## Entscheidungen *(alle Annahme)*

| Thema | Vorschlag |
|---|---|
| Standardmodell für Fähigkeiten | Angelehnt an die **ICF** (Internationale Klassifikation der Funktionsfähigkeit, WHO), stark vereinfacht: fünf Bereiche, je vier Stufen. Wir übernehmen die Gliederung, nicht die Codes. |
| Stufen | `UNBEEINTRAECHTIGT`, `LEICHT`, `MITTEL`, `STARK` (grob die ICF-Beurteilungsmerkmale 0 bis 3) plus `UNBEKANNT` (Standard). Kein Zahlenwert, kein Prozent. |
| Wirkungsweise | **Regelbasiert und deterministisch:** Aus Stufen entstehen feste Anpassungsparameter und Zeilen im Anpassungsblock. Das LLM bekommt sie als Anweisung und leitet nichts selbst ab. |
| Herkunft jedes Wertes | `ANGEGEBEN` (Nutzer sagt es selbst), `EINGERICHTET` (Angehörige oder Pflege mit Einwilligung), `BEOBACHTET` (aus dem Gespräch, nur nach Bestätigung). Dazu Datum der letzten Bestätigung. |
| Vorrang | `ANGEGEBEN` schlägt `EINGERICHTET` schlägt `BEOBACHTET`. Eine Beobachtung überschreibt nie eine Angabe. |
| Standard bei `UNBEKANNT` | Wie `UNBEEINTRAECHTIGT`, mit den Standardeinstellungen aus dem Steckbrief (kurze klare Sätze, ruhiges Tempo). |
| Veränderlichkeit | Fähigkeiten ändern sich langsam. Nach 6 Monaten fragt der Assistent beiläufig, ob es noch stimmt. |
| Vergessen | Jeder Wert und das ganze Modell sind einzeln löschbar (Datenschutzbefehle wie in `C-03`). Danach gelten die Standardwerte. |

## Das Modell

```
Nutzermodell
├── praeferenzen    (feste Schlüssel)
├── gewohnheiten    (Liste von Mustern)
└── faehigkeiten    (fünf Bereiche, je Stufe + Hinweise)
```

Jedes Attribut trägt `quelle` (`ANGEGEBEN`, `EINGERICHTET`, `BEOBACHTET`) und `stand` (Datum der letzten Bestätigung).

### Fähigkeiten (Capabilities)

| Bereich | Stufe | Hinweise (feste Liste, optional) | Bedeutung fürs Gespräch |
|---|---|---|---|
| `sehen` | `UNBEEINTRAECHTIGT` … `STARK` | `LESEBRILLE`, `FARBSEHSCHWAECHE`, `NUR_HELLES_LICHT` | Bildschirmtext groß und kontrastreich; Wichtiges **zusätzlich sprechen**; bei `STARK` nie nur visuell. |
| `hoeren` | wie oben | `HOERGERAET`, `LINKS_SCHLECHTER`, `HINTERGRUNDLAERM_PROBLEM` | Langsamer, lauter, kurze Sätze, wichtige Angaben wiederholen; bei `STARK` Text ergänzen. |
| `kognition` | wie oben | `KURZZEITGEDAECHTNIS`, `ORIENTIERUNG`, `AUFMERKSAMKEIT` | Eine Frage pro Antwort, einfache Sätze, Rückbestätigung, mehr Erinnerungen; nie korrigieren oder prüfen („Das habe ich dir schon gesagt“ ist verboten). |
| `motorik` | wie oben | `ZITTERN`, `FEINMOTORIK`, `GEHEN`, `KRAFT` | Sprachbedienung bevorzugen, keine Fingerfertigkeit verlangen, längere Reaktionszeit, bei `STARK` Bestätigung per Sprache. |
| `sprechen` | wie oben | `LEISE`, `UNDEUTLICH`, `STOTTERN`, `DIALEKT` | Längeres Zuhören, mehr Geduld bei Pausen, bei unsicherer Spracherkennung nachfragen statt raten, nicht unterbrechen. |

`sprechen` ergänzt die vier genannten Bereiche, weil die Spracherkennung die Schnittstelle des Produkts ist. Weitere Bereiche (Mobilität, Alltagsaktivitäten) erst bei Bedarf.

### Präferenzen

| Schlüssel | Werte | Wirkung |
|---|---|---|
| `anrede` | `SIE`, `DU` | **Quelle** der Anrede; `C-07` liest sie. |
| `anredename` | Text | Wie der Nutzer genannt werden will („Frau Huber“, „Resi“). |
| `sprechtempo` | `LANGSAM`, `NORMAL`, `SCHNELL` | Tempo der Sprachausgabe. |
| `lautstaerke` | `LEISE`, `NORMAL`, `LAUT` | Grundlautstärke. |
| `antwortlaenge` | `KURZ`, `MITTEL` | Wunsch zur Länge, wirkt wie ein Stilparameter in `C-07`. |
| `proaktiv` | `NEIN`, `NUR_ERINNERUNGEN`, `JA` | Ob der Assistent von sich aus ansprechen darf (Einwilligung, `C-06`). |
| `themen_mag` / `themen_meidet` | Liste von Begriffen | Themen, die er gern hat oder nicht möchte („Garten“, „Krankheiten“). |
| `ausgabe` | `NUR_SPRACHE`, `SPRACHE_UND_TEXT` | Kanal der Ausgabe. |

Präferenzen sind **Einstellungen**, keine Urteile über Personen.

### Gewohnheiten

| Attribut | Beschreibung |
|---|---|
| `art` | `TAGESABLAUF`, `GESPRAECHSZEIT`, `REGELMAESSIG`. |
| `beschreibung` | Kurzer Text („Frühstück um 8 Uhr“); keine Gesundheitsdetails ohne Einwilligung. |
| `zeitfenster` | Optional: Wochentage und Uhrzeitbereich. |
| `bedeutung` | Was der Assistent damit darf: `ERINNERN`, `NICHT_STOEREN`, `ANSPRECHEN`. |

Beispiel: „Zwischen 13 und 15 Uhr ruht Frau Huber“ → `NICHT_STOEREN`; der Assistent spricht dann nicht von sich aus an. Gewohnheiten dienen **nur dem Anpassen von Zeitpunkt und Ton**, nicht der Überwachung. Sie werden vom Nutzer genannt oder bestätigt, nicht heimlich aus Nutzungszeiten errechnet.

## Schnittstelle *(Annahme)*

Bibliothek mit kleiner Schnittstelle; der Speicher liegt hinter `NutzermodellStore` (Neo4j und eine Variante im Speicher für Tests, wie bei den übrigen Komponenten).

| Operation | Beschreibung |
|---|---|
| `lade(nutzer_id) → Nutzermodell` | Liefert das Modell; fehlt es, ein Standardmodell (alles `UNBEKANNT`). |
| `anpassung(modell) → Anpassung` | Berechnet **deterministisch** Anpassungsparameter und **Anpassungsblock**. Kein LLM, kein Seiteneffekt. |
| `setze(nutzer_id, schluessel, wert, quelle)` | Setzt einen Wert; prüft Quellenvorrang und erlaubte Werte; ungültige Werte werden abgelehnt, nicht geraten. |
| `schlage_vor(nutzer_id, schluessel, wert)` | Legt einen **Vorschlag** aus Beobachtung an (noch nicht wirksam). |
| `bestaetige(...)` / `verwirf(...)` | Der Nutzer bestätigt oder lehnt ab; Verworfenes kommt 90 Tage nicht wieder. |
| `zeige(nutzer_id) → Text` | Fasst das Modell in einfachen Sätzen zusammen („Ich weiß, dass du lieber langsam sprichst …“). |
| `vergiss(nutzer_id, schluessel \| alle)` | Löscht einen Wert oder das gesamte Modell. |

### Anpassung (abgeleitet)

| Parameter | Werte | Abgeleitet aus |
|---|---|---|
| `tempo` | `LANGSAM`, `NORMAL` | `sprechtempo`, `hoeren`, `kognition` |
| `lautstaerke` | `NORMAL`, `LAUT` | `lautstaerke`, `hoeren` |
| `satzlaenge_max` | `KURZ`, `MITTEL` (nie mehr, als `C-07` erlaubt) | `kognition`, `hoeren`, `antwortlaenge` |
| `wiederholung` | `AUS`, `WICHTIGES`, `RUECKBESTAETIGEN` | `kognition`, `hoeren` |
| `ausgabe_kanal` | `NUR_SPRACHE`, `SPRACHE_UND_TEXT` | `hoeren`, `sehen`, `ausgabe` |
| `wartezeit` | normal, verlängert (Stille bis zur Rückfrage) | `sprechen`, `motorik`, `kognition` |
| `proaktivitaet` | `NEIN`, `NUR_ERINNERUNGEN`, `JA` | `proaktiv`, Gewohnheiten (`NICHT_STOEREN`) |
| `themen_meiden` | Liste | `themen_meidet` |
| `mehr_erinnerungen` | ja/nein | `kognition` (Kurzzeitgedächtnis) |

Beispiel **Anpassungsblock** (`hoeren = MITTEL`, `kognition = LEICHT`): „Sprich langsam und deutlich. Verwende kurze Sätze. Wiederhole wichtige Angaben einmal. Stelle höchstens eine Frage. Weise nicht auf Einschränkungen hin und korrigiere nicht.“

**Konfliktregel:** Bei Widerspruch gilt die **verständlichere** Anpassung (kürzer vor länger, langsamer vor schneller). Das Nutzermodell kann Sicherheitsregeln nie aufweichen; Notfalltexte (`C-06`) erscheinen bei Hör- oder Sehschwäche zusätzlich als Text und bleiben in kurzen Sätzen.

## Lernen aus dem Gespräch

Der Assistent darf Hinweise sammeln, aber **eng begrenzt**:

- **Ausdrückliche Bitten** („Sprich lauter“, „Ich sehe das nicht gut“) sind eine **Angabe** und gelten sofort.
- **Beobachtbare Signale** *(Annahme)*: wiederholtes „Wie bitte?“ oder Bitte um Wiederholung (→ `hoeren`); wiederholt niedrige Erkennungssicherheit trotz Wiederholung (→ `sprechen`).
- **Schwelle:** Mindestens 3 Hinweise an verschiedenen Tagen, bevor ein Vorschlag entsteht. Belege werden nur gezählt, nie im Wortlaut gespeichert.
- **Vorschlag, nie Urteil:** Der Assistent fragt beiläufig („Soll ich etwas langsamer sprechen?“). Bei „Nein“ wird verworfen.
- **Keine Diagnoseaussage.** Nie „Du hörst schlecht“, sondern nur, was der Assistent **anders machen** will.
- **Kognition wird nie aus dem Gespräch abgeleitet**, auch nicht als Vorschlag. Sie ist nur `ANGEGEBEN` oder `EINGERICHTET`. Das Ableiten solcher Zustände wäre Profilbildung und Diagnose durch die Hintertür.
- Ohne Einwilligung in die Beobachtung gibt es keine Beobachtungen und keine Vorschläge.

## Rahmenbedingungen

- **Gesundheitsdaten (Art. 9 DSGVO).** `faehigkeiten` sind Gesundheitsangaben; sie werden nur mit **ausdrücklicher Einwilligung** geführt. Ohne sie gilt das Standardmodell; Präferenzen wie Anrede oder Tempo dürfen gespeichert werden, wenn der Nutzer sie ausdrücklich nennt.
- **Datensparsamkeit.** Nur Stufen und Hinweiswörter aus festen Listen, **kein Freitext zu Gesundheit**, kein Wortlaut aus Gesprächen.
- **Einsehen, Berichtigen, Löschen** per Sprache (`zeige`, `setze`, `vergiss`). Angehörige sehen nur, was der Nutzer freigibt.
- **Keine Weitergabe.** Das LLM bekommt **nur den Anpassungsblock**, nie Stufenwerte oder Bezeichnungen von Einschränkungen. Das Modell gelangt nicht in Logs.
- **Hilfe statt Bevormundung.** Der Assistent verweigert nichts, weil er etwas für unzumutbar hält; er wählt nur eine passende Form. Wünsche des Nutzers gehen vor.
- **Kein Überwachungsinstrument.** Keine Auswertung von Tagesverläufen oder Fähigkeiten für Dritte, kein Alarm an Angehörige aus dem Modell heraus. Alarme sind Sache der Notfallregeln (`C-06`) mit eigener Einwilligung.

## Logging

Wie bei den übrigen Komponenten (siehe `CLAUDE.md`): Logger `user_model` / `user_model.ausgabe`, `--log-level`, Umgebungsvariable `USER_MODEL_LOG_LEVEL`. **Nie** protokolliert werden Stufen, Hinweise, Präferenzwerte, Themen, Namen oder Gewohnheiten; nur Zähler, Dauern, Ausnahmetypen und technische IDs.

## Tests *(Vorschlag)*

- Standardmodell (alles `UNBEKANNT`) liefert die Standardanpassung des Steckbriefs.
- Jede Stufe und jeder Hinweis erzeugt die erwarteten Anpassungsparameter (Tabelle als Test).
- Konfliktregel: widersprüchliche Wünsche → verständlichere Variante; `satzlaenge_max` übersteigt nie die Grenze aus `C-07`.
- Quellenvorrang: `BEOBACHTET` überschreibt nie `ANGEGEBEN`; ungültige Werte werden abgelehnt.
- Vorschläge: erst nach der Schwelle, nie wirksam ohne Bestätigung; `kognition` wird nie automatisch gesetzt.
- Notfalltext bei `hoeren = STARK` enthält die Textausgabe.
- Der Anpassungsblock enthält **keine** Stufenwerte oder Diagnosebegriffe.
- `vergiss` entfernt Wert und Vorschläge; danach gilt der Standard.
- Logs (`caplog`) enthalten keine Werte, Themen oder Namen.

## Offene Punkte

- **Einwilligung:** eine oder zwei (Gesundheitsdaten vs. Komfort-Einstellungen), und wer sie einholen darf (Nutzer, Betreuer, Angehörige).
- **Erfassung zu Beginn:** Fragebogen beim Einrichten oder nur im Gespräch? Wer darf `EINGERICHTET` setzen?
- **Stufenskala:** Reichen vier Stufen, stimmt die Zuordnung zu Anpassungen? Mit Fachleuten (Altersmedizin, Hörakustik) und echten Nutzern prüfen.
- **Schwellen** (3 Hinweise, 90 Tage, 6 Monate) sind Annahmen und an Erfahrung zu eichen.
- **Mehrere Nutzer** im Haushalt: nicht vorgesehen (ein Nutzer je Gerät).
- **Abgrenzung zu `C-03`:** Ist ein Hobby eine Präferenz oder ein Fakt? Vor der Umsetzung entscheiden, damit nichts doppelt gehalten wird.
- Schlüsselnamen (`themen_mag`, `ausgabe`, Gewohnheitsarten) mit dem Steckbrief abgleichen.
