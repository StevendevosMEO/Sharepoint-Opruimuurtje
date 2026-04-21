# Opruim Uurtje - MEO Klantenmappen Cleanup Skill

## 📦 Wat zit erin?

- **SKILL.md** - Volledige instructies voor automatische cleanup van MEO's klantenmappen
- **evals/evals.json** - 3 realistische testcases voor validatie

## ✨ Wat doet deze skill?

Automatische maandelijkse opschoning van MEO's klantenmappen op SharePoint:

1. **Mount alle klantenmappen** (A-Z + Klanten-Documenten)
2. **Past 8 cleanup-regels toe in volgorde:**
   - R5: Inactieve klantmappen (>5 jaar) verwijderen
   - R3: Oude projectmappen (>3 jaar) verwijderen
   - R1: Tussenversie PDFs verwijderen (>3 maanden, maar ALLEEN als HR+WEB/INT aanwezig)
   - R2: PSD-bestanden verwijderen (>3 maanden)
   - R6: Grote pre-2025 bestanden verwijderen (>10MB)
   - R7: Klanten-Documenten opschonen
   - R8: Lege mappen verwijderen (recursief)
3. **Genereert Excel-rapport** met:
   - Top 20 zware bestanden (>200MB) ter handmatige beoordeling
   - Statistieken per regel
   - Opvallende items gemarkeerd
4. **Stuurt samenvatting** naar Joost, Suzanne en Steven

## 🛡️ Beschermingen

- **Materialenmappen:** NOOIT verwijderd (behalve R5: volledige inactieve klantmap)
- **MEO-map:** NOOIT aangeraakt
- **2025-2026 bestanden:** NOOIT automatisch verwijderd (ter beoordeling)
- **Dry-run mode:** Standaard veilig; echte verwijderingen alleen op verzoek

## 📅 Timing

- >3 maanden = 90 dagen
- >3 jaar = 1095 dagen
- >5 jaar = 1825 dagen
- >6 maanden lege mappen = 180 dagen

## 🎯 Prefixes & Suffixes

**Standaard prefixes voor Regel 1:** HR, WEB, INT  
(Aanpasbaar: zeg "met INT + EXTRA prefixes")

**Suffixes die gestrip worden:** BV, SB, RT, ROW, RBD, AB, KC, MV, CT, DEF, PZ, YD, V1, V2 etc.

## 🚀 Hoe gebruiken?

### Direct runnen:
```
"Opruim uurtje"
```

### Schedulen (maandelijks):
```
/schedule "Opruim uurtje" -- maandelijks op eerste dag om 22:00
```

### Met opties:
```
"Opruim uurtje, maar zeg wat verwijderd WORDT zonder echt te verwijderen (dry-run)"
"Opruim uurtje met ALLEEN HR-bestanden"
"Opruim uurtje met extra grote bestandlimiet van 50MB"
```

## 📊 Test Cases

Er zijn 3 evals opgesteld:

1. **dry_run_april_2026** - Volledige april-run in dry-run mode
2. **pdf_versioning_rule1** - Valideer Regel 1 (PDF-versies) correct werkt
3. **materiaal_protection** - Controleer materialenmappen altijd gespaard zijn

Run deze via skill-testing om te valideren dat alle regels goed implementeren.

## 📝 Volgende Stappen

1. Lees SKILL.md grondig door
2. Voer test cases uit en review output
3. Installeer skill in Cowork
4. Voer eerste keer in dry-run uit (feedback vragen)
5. Schedule maandelijks (bijv. `/schedule`)
6. Ontvang maandelijks rapport van Opruim Uurtje

## ❓ Vragen?

- **Prefixes/suffixes aanpassen?** Zeg het gewoon aan Claude
- **Andere extensies voor R6?** Aanpasbaar
- **Timing thresholds?** Aanpasbaar per maand
- **Materialen-exception?** Alleen R5 overwrites bescherming

---

**Gemaakt:** April 2026  
**Voor:** MEO (design/communicatiebureau)  
**Status:** Klaar voor testing en scheduling
