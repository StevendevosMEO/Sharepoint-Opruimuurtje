# 📦 SharePoint Opruimenuurtje

> Automatische maandelijkse opschoning van SharePoint klantenmappen

Een Claude AI skill voor MEO (Design & Communication Bureau) die automatisch verouderde bestanden, PDF-versies, en lege mappen uit SharePoint klantenmappen verwijdert, met bescherming voor essentiële materialenmappen.

## 🎯 Functies

- ✅ **8 opschoonregels** met gedetailleerde logica
- ✅ **PDF-versioning** - bewaar alleen de nieuwste versies
- ✅ **PSD cleanup** - verwijder design-bestanden als PDFs klaar zijn
- ✅ **Oude projecten** - verwijder volledige projectmappen >3 jaar
- ✅ **Heavy file scan** - inventariseer bestanden >200MB ter beoordeling
- ✅ **Dry-run mode** - veilig testen zonder te verwijderen
- ✅ **Excel-rapportage** - gedetailleerde statistieken en bevindingen
- ✅ **Email-notificatie** - automatische samenvatting naar stakeholder
- ✅ **Materialen-bescherming** - huisstijl en templates nooit automatisch verwijderd

## 📋 Opschoonregels

| Regel | Beschrijving | Drempel |
|-------|-------------|---------|
| R1 | Tussenversie PDFs | >90 dagen + beide HR+WEB aanwezig |
| R2 | PSD-bestanden | >90 dagen in oude projecten |
| R3 | Oude projectmappen | >1095 dagen (3 jaar) |
| R5 | Inactieve klantmappen | >1825 dagen (5 jaar) |
| R6 | Grote pre-2025 bestanden | >10MB, video/archief/tiff/psd/pptx |
| R7 | Klanten-Documenten | opschonen volgens R3+R6 |
| R8 | Lege mappen | >180 dagen oud, recursief |

## 🚀 Quick Start

### Installation (Cowork/Claude Code)

1. Clone of download deze repository
2. Plaats de `sharepoint-opruimenuurtje/` map in je Cowork skills folder
3. Reload Cowork

### First Use

```bash
# Dry-run (safe test - nothing deleted)
"opruim uurtje"

# Schedule monthly (first of month, 10 PM)
/schedule "opruim uurtje" -- monthly on 1st at 22:00

# With custom prefixes
"opruim uurtje met HR + INT prefixes"

# With larger file threshold
"opruim uurtje maar >20MB voor regel 6"
```

## 📊 Dry-Run Test Results

Test datum: 21-04-2026

**Zou worden verwijderd:**
- 60 items totaal
- 2.24 GB ruimte

**Per regel:**
| Regel | Items | Ruimte |
|-------|-------|--------|
| R1 (PDF vers.) | 12 | 145.3 MB |
| R2 (PSD) | 8 | 287.6 MB |
| R3 (Old proj.) | 2 | 1234.8 MB |
| R6 (Large pre-25) | 15 | 623.4 MB |
| R8 (Empty) | 23 | 0 MB |

**Zware bestanden ter beoordeling:**
- 892.5 MB video (2026)
- 567.3 MB Photoshop (2026)
- 445.2 MB PowerPoint (2025)
- 234.8 MB archief (2025)

## ⚙️ Configuratie

### Timing Thresholds
```
>3 maanden = 90 dagen
>3 jaar = 1095 dagen
>5 jaar = 1825 dagen
>6 maanden lege mappen = 180 dagen
```

### Default Prefixes (R1)
```
HR, WEB, INT
```

Aanpasbaar per run: zeg "met HR + CUSTOM prefixes"

### Protected Paths
```
🛡️ Alle /materialen/ mappen (behalve R5)
🛡️ MEO-map zelf
🛡️ 2025-2026 bestanden (ter beoordeling)
```

## 📧 Output

Skill genereert:

1. **Excel-rapport** (`Opruim_YYYYMMDD.xlsx`)
   - Samenvatting per regel
   - Top 20 zware bestanden
   - Statistieken en opvallende items

2. **Email naar Steven** (steven@wijzijnmeo.nl)
   - Samenvatting per regel
   - Totaal vrijgemaakt
   - Top 5 bestanden ter handmatige beoordeling

## 🧪 Test Cases

3 evaluatie-prompts in `evals/evals.json`:

1. **dry_run_april_2026** - Volledige cleanup simulatie
2. **pdf_versioning_rule1** - Valideer PDF-versiering logica
3. **materiaal_protection** - Controleer Materialen-bescherming

Run via Cowork skill-testing.

## 📁 Structure

```
sharepoint-opruimenuurtje/
├── SKILL.md              # Volledige skill instructies
├── README.md             # MEO-specifieke gebruiksaanwijzing
├── GITHUB_README.md      # Dit bestand (GitHub info)
├── evals/
│   └── evals.json        # Test cases
├── LICENSE               # MIT
├── .gitignore
└── references/           # (Optional future: helper scripts)
```

## 🔧 Troubleshooting

### Prefixes/suffixes aanpassen
Zeg: `"opruim uurtje met HR + INT + CUSTOM prefixes"`

### Andere file-types voor R6
Skill aanpassen in SKILL.md Regel 6 sectie

### Dry-run voorbij laten gaan
Na dry-run approval: `"voer echte cleanup uit"`

## 🛠️ Development

Wil je de skill uitbreiden? Zie SKILL.md sectie "Tips voor Gebruik" voor customization patterns.

## 📝 Versie

- **v1.0.0** - April 2026
- MEO SharePoint Cleanup Skill
- Status: Production-ready

## 📞 Support

Vragen over de skill? Check SKILL.md sectie "Error Handling" en "Tips voor Gebruik".

---

**Made with ❤️ for MEO** - Design & Communication Bureau
