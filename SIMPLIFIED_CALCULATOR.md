# Simplified Scientific Calculator Implementation

This document describes the modifications made to create a simplified version of the Windows Calculator that focuses solely on the Scientific mode.

## Changes Implemented

### 1. Scientific Mode as Default
- Modified `ApplicationViewModel.cs` to default to `ViewMode.Scientific` instead of `ViewMode.Standard`
- Updated `MainPage.xaml.cs` to start in Scientific mode by default
- Changed fallback mode from Standard to Scientific in error recovery

### 2. Navigation Menu Simplification
- Modified `NavCategory.cpp` to filter out all modes except Scientific
- Updated `CreateMenuOptions()` to only include the Calculator group (not Converters)
- Hidden Standard, Programmer, Date Calculator, and all Converter modes from the navigation menu

### 3. Always on Top Persistence
- Added state persistence for "Always on Top" mode
- State is saved to `LocalSettings` on app suspension
- State is restored on app startup in `RestoreAlwaysOnTopState()`
- The existing Always on Top functionality (pin button) now persists between sessions

### 4. Keyboard Shortcut (Ctrl+Shift+C)
- Added `RegisterQuickLaunchHotkey()` method to register Ctrl+Shift+C shortcut
- Implemented `OnQuickLaunchHotkeyInvoked()` handler for the shortcut
- Added `ToggleWindowVisibility()` method

## UWP Platform Limitations

This is a Universal Windows Platform (UWP) application, which has some inherent limitations:

### Global Hotkeys
**Limitation**: UWP apps run in a sandboxed environment and cannot register true system-wide global hotkeys that work when the app doesn't have focus.

**What We Can Do**: 
- The Ctrl+Shift+C keyboard accelerator works when the Calculator app has focus
- The keyboard shortcut can be used to trigger window operations within the app

**What We Cannot Do**:
- Register a Windows-level hotkey that works when the app is minimized or when other apps have focus
- This would require either:
  - Converting to a Desktop Bridge (Centennial) app with full trust capabilities
  - Implementing a separate Win32 service/background task
  - Using Windows App SDK (WinUI 3) instead of UWP

### System Tray Functionality
**Limitation**: UWP apps cannot create system tray icons or minimize to the system tray in the traditional sense.

**What We Can Do**:
- Handle app suspension properly to save state
- The app can continue running in the background when minimized to taskbar
- State is preserved when the app is resumed

**What We Cannot Do**:
- Create a system tray icon
- Intercept window close to prevent app termination
- Truly "hide" the app from the taskbar
- This would require:
  - Desktop Bridge with full trust capabilities
  - Win32 APIs for system tray (NotifyIcon)
  - Windows App SDK with Desktop-specific features

## Alternative Solutions for Full Functionality

To implement true global hotkeys and system tray functionality, consider:

1. **Windows App SDK (WinUI 3)**: Migrate to Windows App SDK which provides better desktop integration while maintaining modern UI
2. **Desktop Bridge**: Package as a Desktop Bridge app with full trust to access Win32 APIs
3. **Hybrid Approach**: Keep the UWP calculator but add a small Win32 companion app for global hotkey registration and tray icon management

## Testing the Implementation

1. Launch the calculator - it should open directly in Scientific mode
2. No other modes should appear in the navigation menu
3. Use the pin button in the title bar to enable Always on Top mode
4. Close and restart the app - Always on Top state should be preserved
5. Press Ctrl+Shift+C while the app has focus - this triggers the quick launch handler

## Files Modified

- `src/CalcViewModel/Common/NavCategory.cpp` - Navigation menu filtering
- `src/Calculator.ManagedViewModels/ApplicationViewModel.cs` - Default mode changes
- `src/Calculator/Views/MainPage.xaml.cs` - State persistence and keyboard shortcuts

## Known Issues

1. Ctrl+Shift+C only works when the app has focus (UWP limitation)
2. No system tray icon due to UWP sandbox restrictions
3. Cannot prevent window close to minimize to tray (UWP limitation)

## Future Enhancements

If migrating to Windows App SDK or Desktop Bridge becomes an option:
- Implement true global hotkey using Win32 RegisterHotKey API
- Add system tray icon using NotifyIcon
- Intercept window close events to minimize to tray
- Add context menu to system tray icon for quick access
