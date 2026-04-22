# SharePoint Opruimenuurtje v2

🧹 **Automatically clean up MEO SharePoint without syncing terabytes of data locally.**

Direct Microsoft Graph API access + 8 intelligent cleanup rules + monthly scheduling.

## Features

✨ **Direct API Access** - No OneDrive synchronization needed  
✨ **26 Klantenmappen** - A through Z scanned automatically  
✨ **Intelligent Protection** - materialenmappen and standards never deleted  
✨ **8 Cleanup Rules** - Different strategies for different file types  
✨ **Full Reporting** - Excel exports + email summaries  
✨ **Dry-Run Mode** - Safe preview before execution  
✨ **Monthly Scheduling** - Automated or manual trigger  

## Quick Start

### Installation
```bash
git clone https://github.com/[your-org]/sharepoint-opruimenuurtje-v2.git
cd sharepoint-opruimenuurtje-v2
pip install -r requirements.txt
```

### First Time Setup
```bash
python scripts/graph-cleanup.py --setup
# Browser opens for SharePoint authentication (OAuth)
# Approved once, automatic forever after
```

### Monthly Cleanup
```bash
# Preview changes
python scripts/graph-cleanup.py --dry-run

# Review report, then execute
python scripts/graph-cleanup.py --execute
```

## The 8 Cleanup Rules

| Rule | What | Impact | Safety |
|------|------|--------|--------|
| R1 | Old PDF versions | Keep latest only | ✅ Low |
| R2 | PSDs > 1 year | Move to Archive | ✅ Low |
| R3 | Inactive projects | HR/WEB/INT + 2y old | ⚠️ Medium |
| R4 | ARCHIEF marked | Deleted 3mo+ ago | ✅ Low |
| R5 | Inactive clients | Entire folder > 2y | 🔴 High |
| R6 | Large old files | > 200MB pre-2025 | ⚠️ Medium |
| R7 | Log files | .log, .tmp, debug | ✅ Low |
| R8 | System files | Thumbs.db, .DS_Store | ✅ Low |

**Potential Savings:** ~17-20 GB per month

## Protected Folders

These are NEVER deleted:
- `materialenmappen/*` - Client original materials
- `MEO/` - Organization root
- `0 Standaard mappen & documenten/` - Standard templates

Exception: R5 (inactive clients) can delete entire folder **if** truly inactive AND no protected subfolders exist.

## Documentation

- **[SKILL.md](SKILL.md)** - Comprehensive skill documentation
- **[README.md](README.md)** - User guide and quick reference
- **[evals/](evals/)** - Test cases and evaluation criteria

## Testing

Run the evaluation suite:
```bash
python -m pytest evals/
```

Expected: 3/3 tests pass
- Dry-run analysis
- Execution with error handling
- Protection logic verification

## Configuration

Edit `config.json` to customize:
- **Email recipients** (default: steven@wijzijnmeo.nl, joost@..., suzanne@...)
- **Rules to apply** (default: all 8)
- **Thresholds** (default: 2 years for inactivity, 1 year for PSDs)
- **Schedule** (default: monthly on 1st at 02:00 UTC)

## Safety

✅ **Dry-run before execute** - Always review first  
✅ **Protected hierarchies** - Critical folders never touched  
✅ **Reversible** - Deleted files in recycle bin for 93 days  
✅ **Audit trail** - Excel reports document everything  
✅ **Email confirmation** - All actions reported  

⚠️ **R5 is powerful** - Inactive clients are completely removed. Review carefully.

## Troubleshooting

**"Authorization failed"**
→ OAuth token expired. Re-run with `--auth` to re-authenticate

**"Folder not found"**
→ Check that Klanten folders exist in SharePoint

**"No files found to delete"**
→ Your SharePoint is clean! All good.

## API Details

Uses Microsoft Graph API endpoints:
- `GET /sites/meoklanten/drive/root/children` - List folders
- `GET /sites/meoklanten/drive/items/{id}/children` - List files
- `DELETE /sites/meoklanten/drive/items/{id}` - Delete files

Standard OAuth with delegated permissions (Files.ReadWrite.All on your site).

## License

MIT License - see [LICENSE](LICENSE) for details

## Support

Questions? See [SKILL.md](SKILL.md) for full documentation.
