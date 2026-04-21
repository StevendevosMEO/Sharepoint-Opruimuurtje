---
name: opruim-uurtje
description: |
  Automatisch opschonen van MEO's klantenmappen op SharePoint volgens vaste regels.
  
  Dit is een geplande task die maandelijks draait. De skill:
  - Mount alle klantenmappen (A-Z) op MEO's SharePoint
  - Verwijdert automatisch verouderde bestanden volgens 8 regels (oude PDFs, PSDs, inactieve projecten, grote pre-2025 bestanden)
  - Scant na opschoning op zware resterende bestanden (>200MB)
  - Genereert een Excel-rapport met statistieken
  - Stuurt een samenvatting naar Joost, Suzanne en Steven
  
  Ondersteunt "dry-run" mode om veilig te testen. Beschermt altijd materialenmappen en de MEO-map zelf.
  
  Trigger dit met: "opruim uurtje", "voer cleanup uit", "schedule opruim", "run cleanup", of via `/schedule`.
---

# Opruim Uurtje - MEO Klantenmappen Cleanup

## Overzicht

Deze skill voert automatisch de maandelijkse opschoning van MEO's klantenmappen uit op SharePoint. Het volgt 8 duidelijke regels om bestanden en projecten op te ruimen, beschermt gevoelige mappen (Materialen, MEO zelf), en genereert een rapport van zware bestanden ter beoordeling.

**Timing:**
- >3 maanden = 90 dagen
- >3 jaar = 1095 dagen  
- >5 jaar = 1825 dagen
- >6 maanden lege mappen = 180 dagen

**Prefixes (standaard):** HR, WEB, INT  
**Suffixes (stripped):** BV, SB, RT, ROW, RBD, AB, KC, MV, CT, DEF, PZ, YD, V1, V2 en variaties

## Mappenstructuur

```
SharePoint:
├─ Wij zijn MEO - Klanten - A
├─ Wij zijn MEO - Klanten - B
├─ MEO
│  └─ Klanten - C t/m Z
└─ Klanten - Documenten
   └─ Klanten
```

Binnen klantenmappen:
```
KlantNaam/
├─ projecten/
│  └─ werk YYYY/
│     └─ NNNNN ProjectNaam/
│        └─ (submappen)
└─ materialen/ (PROTECTED)
```

PDF-versiemappen matchen: `^\d+\.?\s*(versies|pdf)` (bijv. "4. PDF", "04. Versies")

---

## Workflow

### 1. Setup & Mounting

Vraag gebruiker of alles klaar is, dan:

1. Mount alle 4 mappen via `request_cowork_directory`:
   - "Wij zijn MEO - Klanten - A"
   - "Wij zijn MEO - Klanten - B"
   - "MEO" (bevat Klanten C-Z)
   - "Klanten - Documenten"

2. Activeer verwijderrechten via `allow_cowork_file_delete` op elke map

3. Toon samenvatting van beschermde routes:
   - ✅ Klanten A-Z (alle submappen behalve Materialen)
   - 🛡️ Alle Materialen-mappen (nooit aanraken behalve R5)
   - 🛡️ MEO-map zelf (nooit aanraken)
   - 🛡️ "0 Standaard mappen & documenten" (nooit aanraken)
   - 🗂️ Klanten-Documenten / Klanten (wel opschonen)

### 2. Execute Rules in Order

Pas rules toe in deze **EXACTE volgorde**:

```
R5: Inactieve klantmappen (>5 jaar)
  ↓
R3: Oude projectmappen (>3 jaar)
  ↓
R1: Tussenversie PDFs (>3 maanden)
  ↓
R2: PSD-bestanden (>3 maanden)
  ↓
R6: Grote pre-2025 bestanden
  ↓
R7: Klanten-Documenten opschonen
  ↓
R8: Lege mappen verwijderen (recursief)
```

Houd bij: **aantal items verwijderd** en **ruimte vrijgemaakt** per regel.

### 3. Heavy File Scan

Na opschoning: scan alle klantenmappen op bestanden **>200MB**. Dit omvat:
- Alle 2025-2026 bestanden (worden NIET automatisch verwijderd)
- Alle overgebleven grote bestanden ter beoordeling

### 4. Report & Notify

- Maak Excel-bestand: `Opruim_YYYYMMDD.xlsx`
- Kolommen: Grootte (MB), Datum, Extensie, Pad, Opmerking
- Sorteer op grootte (grootste eerst)
- Top 20 zwaarste
- Totalen: aantal, totale grootte
- Markeer: video's in niet-Materialen, duplicaten, bestanden in "OUD" mappen

Email naar Joost, Suzanne en Steven met:
- Samenvatting per regel (items, ruimte)
- Totaal vrijgemaakt
- Top 5 zware bestanden ter beoordeling
- Link naar Excel

---

## Opschoonregels Gedetailleerd

### Regel 1: Tussenversie PDFs (>3 maanden oud)

**Doel:** Verwijder oude PDF-tussenversies, bewaar de laatste per groep.

**Logica:**
1. Zoek projectmappen (naam begint met 3+ cijfers, NIET exact 4 cijfers) die >90 dagen niet gewijzigd zijn
2. Zoek PDF-versiemappen (naam matcht `^\d+\.?\s*(versies|pdf)`)
3. Check: zijn er zowel HR- als WEB-bestanden aanwezig? (ook INT als prefix)
   - HR: `^(HR|Hartslag)[\s.\-]`
   - WEB: `^(WEB|KLEINE\s+WEB|EXTRA\s+KLEINE\s+WEB)[\s.\-]`
   - INT: `^INT[\s.\-]`
4. **Alleen als BEIDE aanwezig**: groepeer bestanden op basisnaam
   - Strip nummering van voor: `^\d{1,3}[.\s-]+`
   - Strip suffixes van achter: BV, SB, RT, ROW, RBD, AB, KC, MV, CT, DEF, PZ, YD, V1, V2 etc.
   - Groepeer op basisnaam
5. **Per groep:** verwijder ALLE versies behalve de HOOGSTE (numeriek)
6. Doe dit OOK in submappen van de PDF-map

**Voorbeelden:**
```
Bestand: "01 WEB - Design v1.pdf" → groep="WEB - Design", nummer=01
Bestand: "02 WEB - Design v2.pdf" → groep="WEB - Design", nummer=02
Bestand: "03 WEB - Design v3.pdf" → groep="WEB - Design", nummer=03
→ Verwijder 01, 02; behoud 03

Bestand: "04. PDF - HR Huisstijl AB.pdf" → groep="PDF - HR Huisstijl", nummer=04
Bestand: "05. PDF - HR Huisstijl BV.pdf" → groep="PDF - HR Huisstijl", nummer=05
→ Verwijder 04; behoud 05
```

---

### Regel 2: PSD-bestanden (>3 maanden oud)

**Doel:** Verwijder design-bestanden als PDFs klaar zijn.

**Logica:**
1. In projectmappen (>90 dagen oud) waar R1 heeft gewerkt (HR+WEB/INT PDFs bestaan)
2. Verwijder ALLE `.psd` en `.psb` bestanden in die projectmap
3. **NIET in Materialen-mappen**

---

### Regel 3: Oude projectmappen (>3 jaar)

**Doel:** Verwijder volledige projectmappen die niet meer gebruikt worden.

**Logica:**
1. Zoek projectmappen (naam begint met 3+ cijfers) waarvan ALLE bestanden >1095 dagen oud zijn
2. Verwijder hele map
3. **NIET als "materiaal(en)" in pad zit**

---

### Regel 4: Materialen Beschermen (ALTIJD)

**Doel:** Bescherm herbruikbare merkbestanden.

**Logica:**
- Mappen met "materiaal" of "materialen" in het pad: **NOOIT verwijderen**
- Dit zijn huisstijl, logo's, templates — herbruikbaar over projecten heen
- Uitzondering: R5 (hele klantmap >5 jaar) overwrites dit

---

### Regel 5: Inactieve Klantmappen (>5 jaar)

**Doel:** Verwijder geheel inactieve klanten.

**Logica:**
1. Klantmappen waarvan ALLE bestanden >1825 dagen oud zijn
2. Verwijder **hele klantmap** (INCLUSIEF Materialen)
3. Dit is de enige regel die Materialen mag raken
4. **NIET de MEO-map zelf**

---

### Regel 6: Grote Pre-2025 Bestanden

**Doel:** Verwijder zware legacy-bestanden.

**Logica:**
1. In ALLE klantenmappen A-Z: zoek bestanden met deze criteria:
   - Grootte: **>10MB**
   - Datum: **vóór 2025-01-01**
   - Type (video, audio, archief, tiff, psd, pptx, afbeelding, overig):
     - Video/Audio: .mp4, .mov, .avi, .wav, .wmv, .m4v, .mkv
     - Archief: .zip, .rar, .7z
     - TIFF: .tif, .tiff
     - Photoshop: .psd, .psb
     - PowerPoint: .pptx, .ppt
     - Afbeelding: .bmp, .eps
     - Overig: .wpress, .pst

2. Verwijder deze bestanden
3. **NIET in Materialen-mappen**

---

### Regel 7: Klanten-Documenten Opschonen

**Doel:** Opschonen van aparte Klanten-Documenten bibliotheek.

**Logica:**
1. In `Klanten-Documenten / Klanten`:
   - Verwijder hele klantmappen waarvan ALLE bestanden vóór 2025-01-01 zijn
   - In actieve klantmappen: verwijder grote bestanden (>10MB) van vóór 2025-01-01 (zie Regel 6 types)

---

### Regel 8: Lege Mappen Verwijderen

**Doel:** Opruimen van lege directorystructuur.

**Logica:**
1. Na alle andere regels: recursief alle lege mappen opzoeken
2. Mappen ouder dan 180 dagen verwijderen
3. Mappen met alleen verborgen bestanden (`.` prefix) = leeg
4. **Meerdere passes:** herhaal tot 10x (verwijderen van submappen kan parent leeg maken)
5. **NIET in Materialen-mappen**

---

## Rapportage: Zware Bestanden (>200MB)

Na opschoning: scan alle resterende bestanden >200MB.

**Output: Excel-bestand met columns:**
- Grootte (MB)
- Datum (YYYY-MM-DD)
- Extensie
- Volledige Pad
- Opmerking (bijv. "Video in niet-Materialen", "Duplicaat?", "OUD-map")

**Sortering:** Grootte aflopend (grootste eerst)

**Samenvattingen:**
- Totaal aantal bestanden
- Totale grootte (GB)
- Top 20 grootste
- Aantallen per categorie (video, psd, etc.)

**Flagging:**
- 🎬 Video's in niet-Materialen mappen
- ⚠️ Bestanden in "OUD" mappen
- 🔍 Mogelijke duplicaten (vergelijk filenames en maten)
- 📅 Bestanden van 2025-2026 (ter handmatige beoordeling)

---

## Dry-Run Mode

Standaard wordt cleanup in **dry-run** mode uitgevoerd:

- Scan en rapporteer wat ZOU verwijderd worden
- Verwijder GEEN bestanden
- Toon: "Zou X items verwijderen, Y GB ruimte vrijmaken"
- Gebruiker kan feedback geven voordat echte cleanup loopt

**Voor echte run:** gebruiker kan dit expliciet aanvragen na review van dry-run.

---

## Beschermde Routes (NOOIT automatisch aan)

```
🛡️ /Klanten/*/materialen/           (BESCHERMD)
🛡️ /Klanten/*/materiaal/            (BESCHERMD)
🛡️ /MEO/                             (BESCHERMD - zelf aanraken)
🛡️ /0 Standaard mappen & documenten/ (BESCHERMD - standaard mappen)
✅ Alles ander in Klanten A-Z        (OPSCHOONBAAR)
✅ Klanten-Documenten/Klanten/       (OPSCHOONBAAR)
```

**Exceptions:**
- R5 verwijdert hele klantmap = inclusief Materialen (maar NIET "0 Standaard mappen & documenten")
- R4 staat altijd, overschrijft R1-R3

---

## Statistieken & Output

Houd bij:

```
R1 - Tussenversie PDFs:
  - Items verwijderd: X
  - Ruimte vrijgemaakt: Y GB

R2 - PSD-bestanden:
  - Items verwijderd: X
  - Ruimte vrijgemaakt: Y GB

... (voor elke regel)

TOTAAL:
  - Items verwijderd: X
  - Ruimte vrijgemaakt: Y GB
```

Excel-rapport: `Opruim_YYYYMMDD.xlsx`

---

## Email Samenvatting

Verstuur naar: Joost (joost@wijzijnmeo.nl), Suzanne (suzanne@wijzijnmeo.nl), Steven (steven@wijzijnmeo.nl)

**Format:**
```
Onderwerp: Opruim Uurtje - Maandrapport YYYYMM

Body:
- Samenvatting per regel (verwijderingen & ruimte)
- Totaal vrijgemaakt
- Top 5 zware bestanden ter beoordeling
- Link/bijlage naar Excel-rapport

Bijlage: Opruim_YYYYMMDD.xlsx
```

---

## Stap-voor-Stap Uitvoering

1. **Vraag confirmatie** aan gebruiker (dry-run, timing, go/no-go)
2. **Mount alle mappen** via `request_cowork_directory`
3. **Activeer verwijderrechten** via `allow_cowork_file_delete`
4. **Scan alle klantenmappen** A-Z systematisch
5. **Pas regels toe** in volgorde: R5 → R3 → R1 → R2 → R6 → R7 → R8
6. **Houd statistieken** bij per regel
7. **Scan zware bestanden** (>200MB)
8. **Genereer Excel-rapport** met details en opvallende items
9. **Stuur email** met samenvatting + bijlage
10. **Toon eindsamenvatting** in chat met resultaten en insights

---

## Error Handling

Bij fouten:
- Log welk bestand/map probleem gaf
- Ga door met volgende items (niet stoppen)
- Rapporteer errors in samenvatting en email
- Markeer in Excel: "ERROR - niet verwijderd"

---

## Tips voor Gebruik

- **Eerste keer?** Voer in dry-run uit, review rapport, dan echter cleanup
- **Scheduled:** Setup via `/schedule` voor maandelijks automatisch (bijv. eerste dag van maand om 22:00)
- **Feedback?** Na email: Joost/Suzanne/Steven kunnen items voor handmatig behoud markeren
- **Custom prefixes?** Zeg "met INT + EXTRA prefixes" of "ALLEEN HR", skill adapteert

