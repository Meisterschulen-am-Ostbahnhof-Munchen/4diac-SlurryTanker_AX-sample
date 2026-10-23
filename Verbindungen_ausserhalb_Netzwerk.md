# Verbindungen außerhalb des Netzwerks (SlurryTanker.SUB)

Gefunden beim Erzeugen der PDF-Doku mit `IEC61499_FunctionBlock_SVG`: In
`Ventilsteuerung/4diacIDE-workspace/test/FBs/Ventilsteuerung/func/SlurryTanker.SUB`
verlaufen drei Verbindungen über den Rand des Netzwerks hinaus. Alle anderen `.SUB`-Dateien
des Projekts sind unauffällig (automatisch geprüft: alle Wegpunkte aller Verbindungen gegen
den Netzwerkrahmen).

## Die Stellen

| # | Zeile | Art | Verbindung | dx1 | dx2 | dy | Überstand in der SVG |
|---|---|---|---|---|---|---|---|
| 1 | 153 | Event | `Heckschieber.EO_Heckschieber_END` → `Automatik.CNF_Schieber` | 1386.67 | 793.33 | -633.33 | links 51 px, rechts 184 px |
| 2 | 186 | Adapter | `Verteiler.OUT_heben` → `Vorsteuerventil_2A.IN3` | 80 | 80 | **-6880** | oben 724 px |
| 3 | 189 | Adapter | `Verteiler.OUT_senken` → `Vorsteuerventil_2B.IN3` | 80 | 80 | **-6226.67** | oben 610 px |

Bei Nr. 2 und 3 liegt der waagerechte Mittelabschnitt (`dy`) weit oberhalb aller Bausteine.
Bei Nr. 1 läuft die U-Schleife (`dx1`/`dx2`) rechts und links über den Rand.

## Vermutung: kein 4diac-Bug, sondern die Gruppen

Alle beteiligten Bausteine liegen in **Gruppen** (`<Attribute Name="GroupName" …/>`). Die
Koordinaten der Gruppenmitglieder passen nur dann in ihre Gruppen, wenn sie **relativ zur Gruppe**
gelten:

| Gruppe | x | y | Breite | Höhe |
|---|---|---|---|---|
| funktionen | -1060 | -2900 | 12353 | 10700 |
| Vorsteuerventile | -1000 | -7200 | 12293 | 4200 |
| Vorgewendeautomatik | -800 | 7866.67 | 12800 | 5073 |

| Baustein | Gruppe | x, y in der Datei | absolut (Gruppe + Datei) |
|---|---|---|---|
| Verteiler | Vorgewendeautomatik | 7013, 0 | 6213, 7867 |
| Automatik | Vorgewendeautomatik | 1200, 1633 | 400, 9500 |
| Heckschieber | Vorgewendeautomatik | 7020, 3460 | 6220, 11327 |
| Vorsteuerventil_2A | Vorsteuerventile | 2600, 800 | 1600, -6400 |
| Vorsteuerventil_2B | Vorsteuerventile | 2600, 2200 | 1600, -5000 |

- Alle Gruppenmitglieder haben Koordinaten zwischen 0 und Breite/Höhe ihrer Gruppe.
  Absolut gelesen läge z. B. `Verteiler` (y = 0) gar nicht in „Vorgewendeautomatik“ (y ab 7867).
- Mit den absoluten Positionen passt `dy = -6880` zu Nr. 2: Verteiler liegt bei y ≈ 7870, der
  Mittelabschnitt dann bei y ≈ 1000, das Ziel Vorsteuerventil_2A bei y ≈ -6400. Die Leitung
  bleibt also innerhalb der Zeichnung. Entsprechendes gilt für Nr. 3.
- Der SVG-Konverter ignoriert Gruppen und behandelt die Koordinaten als absolut. Dadurch liegen
  die drei Gruppen übereinander, und genau die Verbindungen zwischen verschiedenen Gruppen schießen
  über den Rand hinaus.

Damit wäre es ein Bug im **Konverter** (Gruppen-Offset fehlt, und Gruppen werden nicht
gezeichnet), nicht in 4diac. Das betrifft dann auch das gesamte Layout dieses Netzwerks im PDF,
nicht nur die drei Leitungen.

## Bitte in 4diac prüfen

1. `SlurryTanker` in 4diac öffnen und nachsehen, ob die drei Verbindungen dort sauber zwischen den
   Gruppen verlaufen.
2. Wenn ja: Bug im Konverter. Fix wäre, die Gruppenposition zu den Koordinaten der Mitglieder zu
   addieren (eventuell zusätzlich ein kleiner Innenabstand der Gruppe; den müsste man an
   4diac abmessen) und die Gruppen als Rahmen zu zeichnen.
3. Wenn die Leitungen auch in 4diac ausreißen: Dann sind die gespeicherten `dx1`/`dx2`/`dy`-Werte
   veraltet (z. B. nach Verschieben in eine Gruppe). Abhilfe: Verbindung in 4diac neu ziehen.
