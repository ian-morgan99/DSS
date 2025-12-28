# Qt VS Tools Release Monitoring

This directory contains a GitHub Actions workflow that automatically monitors Qt VS Tools releases and notifies the team when version 3.4+ (with the MSB4044 bug fix) is available.

## Workflow: `monitor-qt-vstools.yml`

### Purpose

Automatically checks for new Qt VS Tools releases on the 1st of every month and creates an issue when version 3.4+ is detected.

### What it Does

1. **Checks Visual Studio Marketplace** for the latest Qt VS Tools version
2. **Checks GitHub releases** (qt-labs/vstools) for additional information
3. **Scans changelog** for mentions of MSB4044, ReadLinesFromFile, or Issue #44
4. **Creates an issue** when version 3.4+ is detected with:
   - Version information
   - Bug fix status
   - Release notes preview
   - Complete testing protocol
   - Links to documentation and resources

### Schedule

- **Automatic:** Runs on the 1st of every month at 9:00 AM UTC
- **Manual:** Can be triggered manually via GitHub Actions UI

### How to Trigger Manually

1. Go to **Actions** tab in GitHub
2. Select **Monitor Qt VS Tools Releases** workflow
3. Click **Run workflow** button
4. Click **Run workflow** to confirm

### Issue Labels

Created issues are automatically labeled with:
- `qt-vstools-release` - Identifies release notification issues
- `monitoring` - Indicates automated monitoring
- `action-required` - Requires team review and testing

### Testing Protocol

When a new version is detected, the created issue includes the complete testing protocol from [UPGRADE_PATH_PLAN.md](../../UPGRADE_PATH_PLAN.md):

1. Create isolated VM environment
2. Install Visual Studio 2022 + all Qt versions (6.4.0, 6.5.1, 6.6.1, 6.8.0, 6.10.0)
3. Install new Qt VS Tools version
4. Test build with each Qt version
5. Verify no MSB4044 errors
6. Validate Qt Designer and MOC generation

### Related Documentation

- [BLOCKING_FACTORS.md](../../BLOCKING_FACTORS.md) - Details on MSB4044 bug
- [UPGRADE_PATH_PLAN.md](../../UPGRADE_PATH_PLAN.md) - Testing protocol and upgrade steps
- [VISUAL_STUDIO_UPGRADE_ANALYSIS.md](../../VISUAL_STUDIO_UPGRADE_ANALYSIS.md) - Complete analysis

### Troubleshooting

**Workflow not running?**
- Check that the workflow file is in `.github/workflows/` directory
- Verify the repository has Actions enabled
- Check workflow permissions in repository settings

**Need to adjust schedule?**
Edit the cron expression in `monitor-qt-vstools.yml`:
```yaml
- cron: '0 9 1 * *'  # Minute Hour Day Month DayOfWeek
```

**Want to change version threshold?**
Modify the version check logic in the "Check Qt VS Tools version" step.

### Permissions Required

The workflow requires:
- `issues: write` - To create issues
- `contents: read` - To checkout repository

These are configured in the workflow file.

---

*For questions or issues with this workflow, refer to the Visual Studio upgrade documentation or contact the development team.*
