# Validation Checklist for Visual Studio Upgrade

**Date:** January 2, 2026  
**Status:** Ready for Testing  
**Prerequisite:** Visual Studio 2022 17.12 or later installed

---

## Overview

All code changes have been completed. This checklist provides steps to validate the Visual Studio upgrade on your development machine.

---

## Prerequisites

Before starting validation, ensure you have:

- [ ] Visual Studio 2022 version 17.12 or later installed
- [ ] Qt 6.10.0 installed (msvc2022_64)
- [ ] Qt VS Tools extension installed (version 3.2.0 rev 47 minimum, 3.4.1+ recommended)
- [ ] Visual Leak Detector 2.5.12 installed
- [ ] vcpkg configured properly

---

## Phase 4: Validation Steps

### Step 1: Verify Project Files Open Correctly

1. **Open the solution in Visual Studio 2022:**
   ```
   Open: DeepSkyStacker.VS2022.sln
   ```

2. **Expected Results:**
   - [ ] Solution opens without upgrade prompts
   - [ ] No errors in Error List
   - [ ] All projects load successfully
   - [ ] Solution Explorer shows all projects

3. **Verify Toolset Configuration:**
   - Right-click on DeepSkyStacker project → Properties
   - Check Configuration Properties → General
   - Verify:
     - [ ] Platform Toolset = v143
     - [ ] Windows SDK Version = 10.0
     - [ ] C++ Language Standard = C++23

---

### Step 2: Build Debug Configuration

1. **Select Debug x64 configuration**
   - Configuration Manager → Active solution configuration: Debug
   - Platform: x64

2. **Clean solution:**
   ```
   Build → Clean Solution
   ```

3. **Build solution:**
   ```
   Build → Build Solution
   ```

4. **Expected Results:**
   - [ ] Build completes without errors
   - [ ] Qt MOC generation succeeds
   - [ ] All projects compile successfully
   - [ ] Output shows: "Build: X succeeded, 0 failed"
   - [ ] No new warnings introduced (compared to baseline)

5. **Check Output Folders:**
   ```
   x64/Debug/DeepSkyStacker.exe exists
   x64/Debug/DeepSkyStackerCL.exe exists
   x64/Debug/DeepSkyStackerLive.exe exists
   ```

---

### Step 3: Build Release Configuration

1. **Select Release x64 configuration**
   - Configuration Manager → Active solution configuration: Release
   - Platform: x64

2. **Clean solution:**
   ```
   Build → Clean Solution
   ```

3. **Build solution:**
   ```
   Build → Build Solution
   ```

4. **Expected Results:**
   - [ ] Build completes without errors
   - [ ] Qt MOC generation succeeds
   - [ ] All projects compile successfully
   - [ ] Output shows: "Build: X succeeded, 0 failed"
   - [ ] No new warnings introduced (compared to baseline)

5. **Check Output Folders:**
   ```
   x64/Release/DeepSkyStacker.exe exists
   x64/Release/DeepSkyStackerCL.exe exists
   x64/Release/DeepSkyStackerLive.exe exists
   ```

---

### Step 4: Verify Qt Integration

1. **Test Qt Designer Integration:**
   - Open any .ui file (e.g., DeepSkyStacker/ui/AboutDlg.ui)
   - Double-click to open in Qt Designer
   - Expected Results:
     - [ ] Qt Designer launches successfully
     - [ ] UI file displays correctly
     - [ ] Can edit and save changes

2. **Verify MOC Files Generation:**
   - Check that .moc files are generated during build
   - Look in intermediate directories (x64/Debug/ or x64/Release/)
   - Expected Results:
     - [ ] moc_*.cpp files are generated
     - [ ] No MOC-related build errors

3. **Test Qt Version Switching (Optional):**
   - Tools → Qt VS Tools → Qt Versions
   - Verify 6.10.0_msvc2022_64 is configured
   - Expected Results:
     - [ ] Qt version is correctly configured
     - [ ] Path points to correct Qt installation

---

### Step 5: Test Projects Validation

1. **Build Test Projects:**
   - Build TestZTrace project (x64 Debug)
   - Build TestZPtr project (x64 Debug)

2. **Expected Results:**
   - [ ] TestZTrace builds successfully with v143 toolset
   - [ ] TestZPtr builds successfully with v143 toolset
   - [ ] No errors related to Windows SDK 10.0
   - [ ] No C++ standard issues

3. **Run Tests (if applicable):**
   ```
   x64/Debug/TestZTrace.exe
   x64/Debug/TestZPtr.exe
   ```
   - [ ] Tests run without crashes
   - [ ] Test results match baseline (if tests are functional)

---

### Step 6: IntelliSense and IDE Features

1. **Test IntelliSense:**
   - Open a .cpp file
   - Type code and check IntelliSense suggestions
   - Expected Results:
     - [ ] IntelliSense works properly
     - [ ] C++23 features are recognized
     - [ ] No excessive squiggles or false errors

2. **Test Navigation:**
   - Right-click on function → Go To Definition
   - Use Ctrl+, (Quick Open)
   - Expected Results:
     - [ ] Navigation works correctly
     - [ ] Project indexing completes successfully

---

### Step 7: Debugging (Optional but Recommended)

1. **Set a breakpoint in DeepSkyStacker main application**

2. **Start debugging (F5)**

3. **Expected Results:**
   - [ ] Application launches in debugger
   - [ ] Breakpoint hits correctly
   - [ ] Can step through code
   - [ ] Variable inspection works
   - [ ] Call stack displays correctly

---

## Validation Summary

After completing all steps, mark the overall status:

- [ ] **PASS:** All validation steps completed successfully
- [ ] **FAIL:** Issues found (document issues below)

---

## Issues Found (if any)

If validation fails, document issues here:

| Step | Issue Description | Severity | Status |
|------|-------------------|----------|--------|
| Example: Step 2 | Build error in DeepSkyStacker.cpp line 123 | High | Open |
|      |                   |          |        |
|      |                   |          |        |

---

## Next Steps

### If Validation PASSES:
1. Mark Phase 4 as complete in PR
2. Merge PR to main branch
3. Update team documentation
4. Roll out to development team
5. Consider testing Qt VS Tools 3.4.1 (see UPGRADE_IMPLEMENTATION_NOTES.md)

### If Validation FAILS:
1. Document all issues in the table above
2. Report issues in PR comments with:
   - Exact error messages
   - Build output logs
   - Steps to reproduce
3. Assign for investigation and fixes
4. Re-run validation after fixes

---

## Additional Resources

- **UPGRADE_PATH_PLAN.md:** Comprehensive upgrade plan with rollback procedures
- **UPGRADE_IMPLEMENTATION_NOTES.md:** Details of changes made and testing protocols
- **VISUAL_STUDIO_UPGRADE_ANALYSIS.md:** Technical analysis and rationale
- **BLOCKING_FACTORS.md:** Constraint analysis and mitigation strategies

---

## Rollback Procedure (If Needed)

If critical issues are found, you can rollback the changes:

1. **Revert to previous commit:**
   ```bash
   git revert HEAD~2..HEAD
   ```

2. **Or manually revert toolset changes:**
   - Edit all .vcxproj files
   - Change `v143` back to `v145` (if reverting to VS2026 Preview)
   - Or change to `v141` (if reverting to VS2017 for test projects)
   - Change `ToolsVersion="Current"` back to `ToolsVersion="15.0"`

3. **Rebuild solution:**
   ```
   Build → Clean Solution
   Build → Build Solution
   ```

See UPGRADE_IMPLEMENTATION_NOTES.md for detailed rollback instructions.

---

**Validation Status:** 🟡 PENDING  
**Last Updated:** January 2, 2026  
**Validator:** _______________  
**Date Validated:** _______________
