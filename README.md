# SharePoint Opruimenuurtje

🧹 **Automatisch opruimen van SharePoint - Verwijdert oude bestanden die je niet meer nodig hebt**

Dit is een skill die jouw SharePoint opschoon houdt door automatisch oude, ongebruikte bestanden te verwijderen - maar alleen die je echt niet meer nodig hebt.

---

## Wat doet het?

De skill scant je SharePoint en verwijdert bestanden op basis van **8 regels**. Elke regel kijkt naar iets anders:

1. **Oude PDF versies** - Houdt alleen de nieuwste versie van een PDF
2. **Oude design bestanden** - Archiveert Photoshop files ouder dan 1 jaar
3. **Inactieve projectmappen** - Markeert mappen die 2+ jaar niet gebruikt zijn
4. **Oude ARCHIEF mappen** - Verwijdert mappen met "ARCHIEF" label die >3 maanden oud zijn
5. **Inactieve klantmappen** - Verwijdert hele klantenmappen als >2 jaar niet gebruikt (MET BEVEILIGING)
6. **Grote oude bestanden** - Verwijdert grote bestanden (>200MB) van voor vorig jaar
7. **Logbestanden** - Ruimt op debug/log bestanden
8. **Systeembestanden** - Verwijdert Windows/Mac cache bestanden

---

## Wat is BESCHERMD?

Sommige mappen worden **NOOIT verwijderd**, zelfs niet als ze oud zijn:

- **Materialen mappen** - Client originele bestanden (altijd beschermd)
- **Standaard mappen** - Templates en standaard documenten
- **Organisatie root** - Bedrijfsfolders

Als een klantmap een Materialen map bevat, kan het zelf nooit helemaal verwijderd worden.

---

## Simpel gezegd

Je kunt dit als volgt instellen:

```bash
# 1. Setup
python scripts/sharepoint_opruim.py

# Dat's het!
```

De skill:
- ✅ Scant alle mappen
- ✅ Kijkt wat oud is
- ✅ Genereert rapport
- ✅ Stuurt email met resultaten
- ✅ Kan maandelijks automatisch draaien

---

## Instellingen (config.json)

Je kunt kiezen welke regels je wilt gebruiken:

```json
{
  "regels_ingeschakeld": {
    "R1": true,    // Aan: oude PDF versies verwijderen
    "R2": true,    // Aan: oude PSDs archiveren
    "R3": false,   // Uit: inactieve projecten alleen markeren
    "R4": false,   // Uit: ARCHIEF mappen alleen markeren
    "R5": false,   // Uit: klantenmappen alleen markeren (voorzichtig!)
    "R6": true,    // Aan: grote oude bestanden verwijderen
    "R7": true,    // Aan: logbestanden opruimen
    "R8": true     // Aan: systeembestanden opruimen
  }
}
```

- **Groene regels** (R1, R2, R6, R7, R8): Veilig - standaard aan
- **Gele regels** (R3, R4): Voorzichtig - standaard uit
- **Rode regel** (R5): Verwijdert hele mappen - standaard uit!

---

## Resultaten

Na afloop krijg je:

- 📊 JSON rapport met exacte aantallen
- 📧 Email naar team met samenvatting
- 📝 Logbestand met alle details
- 💾 Hoeveel GB je hebt vrijgemaakt

---

## Beveiligingen

De skill heeft meerdere beveiligingen ingebouwd:

✅ **Materialen mappen beschermd** - Nooit verwijderd  
✅ **Audit trail** - Alles wordt gelogd  
✅ **Reversible** - Verwijderde bestanden zitten 93 dagen in recycle bin  
✅ **Foutafhandeling** - Slaat bestanden over als ze vergrendeld zijn  
✅ **Email rapportage** - Team weet wat er gebeurd is  

---

## Scheduling

Je kunt dit maandelijks laten draaien:

**Windows:**
- Windows Task Scheduler - plan voor 1e van maand

**Mac/Linux:**
- Cron job: `0 2 1 * * python sharepoint_opruim.py`

**Cloud:**
- GitHub Actions - via workflow

---

## Voor wie dit gebruikt:

- Teams met veel bestanden in SharePoint
- Organisaties die ruimte willen besparen
- Iedereen die automatische opruiming wil zonder handwerk

---

## Dat's het!

Dit is geen ingewikkelde tool - het doet wat het zegt:
1. Scant bestanden
2. Markeert oude dingen
3. Verwijdert wat je niet meer nodig hebt
4. Stuurt je een rapport

Vragen? Kijk in SKILL.md voor technische details.
