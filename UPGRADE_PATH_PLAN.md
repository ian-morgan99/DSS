# DeepSkyStacker Visual Studio Upgrade Path

**Quick Reference Guide**  
**Date:** December 28, 2025

---

## 🎯 Executive Summary

**Current State:** Visual Studio 2022 17.5 with Qt VS Tools 3.2.0 (intentionally held back)  
**Recommendation:** Upgrade VS to 17.12.x, update project metadata, **DO NOT** upgrade Qt VS Tools

---

## ⚡ Quick Start: What to Do Now

### ✅ SAFE TO DO (Recommended)

1. **Upgrade Visual Studio to 17.12.x**
   - Risk: Very Low ✅
   - Benefit: Latest C++23 features, bug fixes, security patches
   - Time: 1-2 hours

2. **Update project file ToolsVersion**
   - Change `ToolsVersion="15.0"` to `ToolsVersion="Current"`
   - Risk: Very Low ✅
   - Benefit: Future-proofing
   - Time: 30 minutes

3. **Update solution file version marker**
   - Update `VisualStudioVersion = 17.5.33424.131` to `17.12.0.0`
   - Risk: Very Low ✅
   - Benefit: Accurate tracking
   - Time: 5 minutes

### ❌ DO NOT DO

1. **DO NOT upgrade Qt VS Tools**
   - Keep version 3.2.0 (rev 47)
   - Version 3.3+ has critical build-breaking bugs
   - Wait for community confirmation of fixes

2. **DO NOT remove multi-Qt version support**
   - Required for supporting multiple release versions

---

## 📋 Phase 1: Immediate Upgrades (DO THIS NOW)

### Step 1: Upgrade Visual Studio 2022

**Prerequisites:**
- Qt VS Tools 3.2.0 auto-update is DISABLED (verify in Tools → Options → Extensions)
- Clean workspace (commit/stash all changes)
- Backup current environment

**Steps:**
1. Open Visual Studio Installer
2. Update Visual Studio 2022 to version 17.12.x (latest stable)
3. **IMPORTANT:** After update, verify Qt VS Tools is still 3.2.0 (rev 47)
4. If Qt VS Tools was upgraded accidentally:
   - Uninstall Qt VS Tools
   - Reinstall version 3.2.0 rev 47 from Qt website
   - Disable auto-updates

**Testing:**
```bash
# Build all configurations
msbuild DeepSkyStacker.VS2022.sln /p:Configuration=Debug /p:Platform=x64
msbuild DeepSkyStacker.VS2022.sln /p:Configuration=Release /p:Platform=x64

# Verify Qt Designer still works
# Open any .ui file and verify Qt Designer launches
```

**Success Criteria:**
- All projects build without errors
- Qt Designer opens .ui files
- No new warnings introduced
- Binary output compares identical (for Release builds)

---

### Step 2: Update ToolsVersion in Project Files

**Why:** ToolsVersion="15.0" is from VS2017. Updating to "Current" is best practice.

**Files to Update:**
```
DeepSkyStacker/DeepSkyStacker.vcxproj
DeepSkyStackerCL/DeepSkyStackerCL.vcxproj
DeepSkyStackerKernel/DeepSkyStackerKernel.vcxproj
DeepSkyStackerLive/DeepSkyStackerLive.vcxproj
DeepSkyStackerTest/DeepSkyStackerTest.vcxproj
ZClass/ZClass.vcxproj
ZClass/TestZTrace/TestZTrace.vcxproj
ZClass/TestZPtr/TestZPtr.vcxproj
```

**Find:**
```xml
<Project DefaultTargets="Build" ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

**Replace with:**
```xml
<Project DefaultTargets="Build" ToolsVersion="Current" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

**Script Option:**
```bash
# Linux/macOS/WSL
find . -name "*.vcxproj" -exec sed -i 's/ToolsVersion="15.0"/ToolsVersion="Current"/g' {} +

# PowerShell (Windows)
Get-ChildItem -Recurse -Filter *.vcxproj | ForEach-Object {
    (Get-Content $_.FullName) -replace 'ToolsVersion="15.0"', 'ToolsVersion="Current"' | 
    Set-Content $_.FullName
}
```

**Testing:**
```bash
# Clean and rebuild
msbuild DeepSkyStacker.VS2022.sln /t:Clean
msbuild DeepSkyStacker.VS2022.sln /p:Configuration=Release /p:Platform=x64
```

**Success Criteria:**
- No new MSBuild errors or warnings
- All projects build successfully
- Qt integration still works

---

### Step 3: Update Solution File Visual Studio Version

**File:** `DeepSkyStacker.VS2022.sln`

**Find:**
```
VisualStudioVersion = 17.5.33424.131
```

**Replace with:**
```
VisualStudioVersion = 17.12.0.0
```

**Note:** Use the actual version number of your installed VS2022. Find it with:
```bash
# PowerShell
& "C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\devenv.exe" /?
```

**Testing:**
- Open solution in Visual Studio
- Verify no "solution needs upgrade" prompts
- Build solution

---

## 📋 Phase 2: Test Project Modernization (OPTIONAL)

**Status:** Medium priority, can be deferred

### Issue

Two test projects still use v141 (VS2017) toolset:
- `ZClass/TestZTrace/TestZTrace.vcxproj`
- `ZClass/TestZPtr/TestZPtr.vcxproj`

### Assessment Questions

Before modernizing, answer:
1. Are these tests actively maintained?
2. Are they run regularly?
3. Do they cover critical functionality?
4. Do they pass on current codebase?

### If YES to above: Modernize

**Changes needed in each project file:**

1. **Update Platform Toolset:**
   ```xml
   <!-- FROM -->
   <PlatformToolset>v141</PlatformToolset>
   
   <!-- TO -->
   <PlatformToolset>v143</PlatformToolset>
   ```

2. **Update Windows SDK:**
   ```xml
   <!-- FROM -->
   <WindowsTargetPlatformVersion>8.1</WindowsTargetPlatformVersion>
   
   <!-- TO -->
   <WindowsTargetPlatformVersion>10.0</WindowsTargetPlatformVersion>
   ```

3. **Add C++ Standard (if not present):**
   ```xml
   <ClCompile>
     <LanguageStandard>stdcpp23</LanguageStandard>
     <!-- ... other settings ... -->
   </ClCompile>
   ```

**Testing:**
```bash
# Build test projects
msbuild ZClass/TestZTrace/TestZTrace.vcxproj /p:Configuration=Debug /p:Platform=x64
msbuild ZClass/TestZPtr/TestZPtr.vcxproj /p:Configuration=Debug /p:Platform=x64

# Run tests
.\x64\Debug\TestZTrace.exe
.\x64\Debug\TestZPtr.exe
```

### If NO to above: Document and Mark

**Option A:** Remove from solution
- Comment out in .sln file or remove ProjectReference

**Option B:** Mark as "do not build"
- In Solution Configuration Manager, uncheck Build checkbox

**Option C:** Document as legacy
- Add comment in project file noting it's unmaintained but kept for reference

---

## 🔍 Phase 3: Qt VS Tools Monitoring (ONGOING)

**Status:** BLOCKED until Qt releases stable version

### Monitoring Checklist

- [ ] Subscribe to Qt VS Tools GitHub releases: https://github.com/qt-labs/vstools/releases
- [ ] Check issue tracker monthly: https://github.com/qt-labs/vstools/issues
- [ ] Monitor Qt Blog for announcements: https://www.qt.io/blog
- [ ] Search Stack Overflow for "Qt VS Tools 3.3" monthly

### Known Issues to Watch For

Track these specific bugs:
1. **MSB4044 Error:** `ReadLinesFromFile` task missing "File" parameter
2. **Multi-version Qt conflicts:** Switching between Qt versions fails
3. **Linker errors:** Unresolved `_main` references
4. **Designer integration:** Qt Designer won't open in VS

### When to Attempt Upgrade

**Green Lights (ALL must be true):**
- ✅ No open critical/high severity bugs in Qt VS Tools GitHub
- ✅ Community confirmation of multi-Qt version stability
- ✅ At least 2 months since last bug report for known issues
- ✅ Positive feedback on Qt forums from similar multi-version setups

**Testing Protocol:**

1. **Isolated Test Environment:**
   - Create new VM or separate machine
   - Install VS2022 (latest)
   - Install ALL Qt versions used: 6.4.0, 6.5.1, 6.6.1, 6.8.0, 6.10.0
   - Install Qt VS Tools (new version)
   - Configure Qt installations in VS Tools

2. **Clone and Test:**
   ```bash
   git clone https://github.com/deepskystacker/DSS.git
   cd DSS
   git checkout main  # or development branch
   ```

3. **Build Test:**
   ```bash
   # Try building with each Qt version configuration
   # Verify no MSB4044 or other Qt-related errors
   msbuild DeepSkyStacker.VS2022.sln /p:Configuration=Release /p:Platform=x64
   ```

4. **Integration Test:**
   - Open .ui files in Qt Designer
   - Verify MOC generation works
   - Switch between Qt versions
   - Test on a feature branch with real development work

5. **Decision:**
   - If ALL tests pass: Create upgrade plan for team
   - If ANY test fails: Document issues, report to Qt, continue on 3.2.0

---

## 📊 Version Tracking

### Current Versions
```yaml
Visual Studio: 17.5.33424.131
Target VS:     17.12.x (latest)
Status:        UPGRADE RECOMMENDED ✅

ToolsVersion:  15.0
Target:        Current
Status:        UPDATE RECOMMENDED ✅

Platform Toolset:
  - Main Projects: v143 (Current ✅)
  - Test Projects: v141 (Optional upgrade ⚠️)

Qt VS Tools:   3.2.0 rev 47
Target:        STAY ON 3.2.0 ❌
Status:        DO NOT UPGRADE

Qt Versions:   6.4.0, 6.5.1, 6.6.1, 6.8.0, 6.10.0
Status:        MAINTAIN ALL VERSIONS ✅

C++ Standard:  C++23 (stdcpp23)
Status:        CURRENT ✅
```

---

## 🚀 Implementation Timeline

### Week 1: Preparation
- [ ] Review this document with team
- [ ] Verify all team members have auto-update disabled for Qt VS Tools
- [ ] Create backup/rollback plan
- [ ] Document current working state (versions, build times, etc.)
- [ ] Select pilot developer for initial testing

### Week 2: Pilot Testing
- [ ] Pilot developer upgrades VS to 17.12.x
- [ ] Update project files (ToolsVersion, solution version)
- [ ] Test all development workflows for 1 week
- [ ] Document any issues or unexpected behavior
- [ ] Measure build times (should be same or better)

### Week 3: Partial Rollout
- [ ] If pilot successful, upgrade 50% of team
- [ ] All upgraded developers test for 1 week
- [ ] Collect feedback
- [ ] Address any issues discovered

### Week 4: Complete Rollout
- [ ] Upgrade remaining team members
- [ ] Update CI/CD build servers (if applicable)
- [ ] Update README.md with new recommended versions
- [ ] Update onboarding documentation
- [ ] Close out upgrade project

---

## 🛡️ Rollback Plan

### If Issues Occur

**Symptom: Build failures after VS upgrade**
1. Verify Qt VS Tools is still 3.2.0 rev 47
2. If upgraded accidentally, reinstall 3.2.0 rev 47
3. Clean solution: `msbuild /t:Clean`
4. Delete `x64/` output folders
5. Rebuild

**Symptom: Qt Designer doesn't open**
1. Reinstall Qt VS Tools 3.2.0 rev 47
2. Reconfigure Qt installations in Tools → Qt VS Tools → Qt Versions
3. Restart Visual Studio

**Symptom: Cannot open solution**
1. Revert .sln file: `git checkout DeepSkyStacker.VS2022.sln`
2. Revert .vcxproj files if ToolsVersion was changed
3. Rebuild

**Complete Rollback:**
```bash
# Revert all project changes
git reset --hard HEAD

# Uninstall VS 17.12.x
# Reinstall VS 17.5.x from Visual Studio Installer
# Verify Qt VS Tools is 3.2.0 rev 47
```

---

## ✅ Testing Checklist

After any changes, verify:

### Build Tests
- [ ] Debug x64 builds without errors
- [ ] Release x64 builds without errors
- [ ] All projects in solution build
- [ ] No new compiler warnings
- [ ] Qt MOC runs successfully
- [ ] UI files compile correctly

### Functionality Tests
- [ ] Qt Designer opens .ui files
- [ ] Can switch between Qt versions
- [ ] Visual Leak Detector still works (Debug builds)
- [ ] vcpkg dependencies resolve correctly
- [ ] Can create installer packages

### Performance Tests
- [ ] Build times comparable or better
- [ ] IntelliSense response time acceptable
- [ ] Solution load time acceptable

### Integration Tests
- [ ] Git integration works
- [ ] Debugging works (breakpoints, watch, etc.)
- [ ] NuGet/vcpkg integration works
- [ ] External tools still accessible

---

## 📞 Support Resources

### If You Encounter Issues

**Qt VS Tools Problems:**
- GitHub Issues: https://github.com/qt-labs/vstools/issues
- Qt Forum: https://forum.qt.io/category/28/qt-for-visual-studio
- Stack Overflow: Tag `qt-vs-tools`

**Visual Studio Problems:**
- Microsoft Docs: https://learn.microsoft.com/en-us/visualstudio/
- Stack Overflow: Tag `visual-studio-2022`
- Visual Studio Developer Community: https://developercommunity.visualstudio.com/

**vcpkg Problems:**
- GitHub: https://github.com/microsoft/vcpkg
- Documentation: https://vcpkg.io/

---

## 📝 Change Log

Track changes made during upgrade:

| Date | Component | From | To | Result | Notes |
|------|-----------|------|----|----|-------|
| YYYY-MM-DD | Visual Studio | 17.5 | 17.12.x | ✅ | Pilot machine |
| YYYY-MM-DD | ToolsVersion | 15.0 | Current | ✅ | All projects |
| YYYY-MM-DD | Solution Version | 17.5.33424.131 | 17.12.0.0 | ✅ | .sln file |
| YYYY-MM-DD | VS Rollout | - | 50% | ✅ | Half team |
| YYYY-MM-DD | VS Rollout | - | 100% | ✅ | Full team |

---

## 🎓 Key Takeaways

1. **Visual Studio version can be safely upgraded** - v143 toolset works across VS2022 versions
2. **Qt VS Tools 3.2.0 is critical** - Do not upgrade until community confirms stability
3. **ToolsVersion is mostly cosmetic** - MSBuild uses latest regardless
4. **C++23 support is already good** - VS2022 v143 supports most C++23 features
5. **Test incrementally** - Pilot → Partial → Full rollout minimizes risk
6. **Document everything** - Helps with troubleshooting and future upgrades

---

**Document Version:** 1.0  
**Created:** December 28, 2025  
**Review:** After each phase completion or when Qt VS Tools releases new stable version
