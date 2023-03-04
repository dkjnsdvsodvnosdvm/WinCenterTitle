# WinCenterTitle
WinCenterTitle is a simple tool that allows you to center align the text in Windows 10 titlebars, the same way it was in Windows 8, 8.1, or even 3.1.

Note：Application requires active Internet connection when downloading symbol files for uDWM.dll

## Installation

* Download archive from [releases](https://github.com/dkjnsdvsodvnosdvm/WinCenterTitle/releases)
* Unpack to folder of your choice (preconfigured for *C:\WinCenterTitle*)
* Edit *WinCenterTitle.xml* and edit line 40, which contains path to application, according to your system
* Register scheduled task which will enable the tweak at startup: run *install.bat* as administrator. This is one way of running the application using the SYSTEM account, which is required in order to be able to hook into DWM.
* VERY IMPORTANT: Hooking DWM requires special permissions for the folder that contains the application and the library. Make sure folder owner is set to *Administrator**s*** and that the *Users* group is added as well, with default permissions (Read & execute, List folder contents, and Read). Otherwise, the application will terminate with error code -6, being unable to hook the library into Desktop Window Manager. Also, in that folder, make sure you have a folder called "symbols", with Full Control permissions for *Users* group.
* Reboot system or manually run the created scheduled task. This will center title bars for all windows. You can close or rerun the application from Task Scheduler, allowing you to toggle between center aligned and left aligned (default) title bars. No DWM restart is required and the library should leave DWM in a clean state when unloading.

## To do

* Hook window activation and optionally hide icon in title bar (could be achieved with [SetWindowThemeAttribute](https://docs.microsoft.com/en-us/windows/win32/api/uxtheme/nf-uxtheme-setwindowthemeattribute))
* Add option to center between window margins, as opposed to space between icon and window controls
* User settings, system tray icon, configuration GUI etc (unlikely)

## How it works

The application injects into DWM and hooks *DrawText* method that is used to compute the area, and draw the title bar text for the window. By doing so, we achieve the effect with least amount of overhead, as the method is called by DWM only when the window is an unskinned (no custom client side decoration aka no [DwmExtendFrameIntoClientArea](https://docs.microsoft.com/en-us/windows/win32/api/dwmapi/nf-dwmapi-dwmextendframeintoclientarea)). After injecting, the application monitors DWM and reinjects should it crash or reload by user action.

This means that all regular windows will have their title bar text centered. Custom skinned windows require more attention; in particular, heavy skinned ones are impossible to modify unless hacks specific to the application are employed - think Visual Studio, Visual Studio Code (fortunately, it is already centered), Office (already centered as well). However, a special class of windows are left left-aligned by this tool, but we can do more regarding them: ribbon windows - think File Explorer, Paint, WordPad etc. These respect a property (CONTENTALIGNMENT) specified in the theme style file (the default is aero.msstyles). To patch this, a rather complicated process is required:

1. Enable custom themes on your system by patching uxtheme. For this, I used UltraUXThemePatcher, but the tool does not matter.
2. Stop WinCenterTitle if running, and then kill dwm in order to get a fresh new process.
3. Make a copy of aero folder and aero.theme from C:\Windows\Resources\Themes in the same folder. I called mine test.theme and folder test.
4. Rename aero.msstyles to test.msstyles from aero folder, and aero.msstyles.mui to test.msstyles.mui from aero\en-US folder. These two steps may require you to take ownership of the files.
5. Download [msstyleEditor](https://github.com/nptr/msstyleEditor) and open the test.msstyles file in it. In the left, browse for *Window\CAPTION*. On the right, click + near *Common* and choose CONTENTAALIGNMENT from the list. Set it to Right (there is currently a bug in msstyleEditor and Centered is labeled Right and vice versa; so, when you see Right, it means Center). Repeat this for *Window\MAXCAPTION* on the left (credits: https://www.sevenforums.com/customization/186924-center-titlebar-text.html). Make sure to save the newly edited file at C:\Windows\Resources\Themes\test\test.msstyle.
6. Edit C:\Windows\Resources\Themes\test.theme, and make sure Path in VisualStyles section reads *%ResourceDir%\Themes\test\test.msstyles*. Also, change DisplayName in Theme section to whatever you like, I changed mine to *@%SystemRoot%\System32\themeui.dll,-2014* which will name the theme *Windows Basic*.
7. Apply the new theme from Personalization\Themes in Settings. Notice how the title bar text is centered in Explorer. There is one issue though: the title bars are now colored, instead of default white/black. This is because DWM contains a check for the name of the msstyles file. If it is called *aero.msstyles*, title bars will be painted white/black, depending on whether you are on light/dark theme in Settings. If the msstyles file has any other name (like in our case), the title bar will have whatever color the msstyles file specifies, or in the case of this renamed aero.msstyles, it will use your accent color no matter if you have 'Show accent color on the following surfaces: Title bars and window borders' in Personalization\Colors in Settings checked or not. To override this behavour, WinCenterTitle has a small code block that identifies the in memory flag of DWM that holds the status of whether the theme is called aero.msstyles and overrides its value to 1, meaning that the theme is called aero.msstyles, despite its actual name, and forcing DWM to render the title bars using the default behavior. So, to complete this guide, start WinCenterTitle now and observe how the title bars will have centered text for all windows, including ribbon applications (File Explorer, WordPad, Paint etc). After aprox. 10 seconds, the title bars will also revert to their default white/black apparance, if you had them this way previously.

### Overview

![image](https://github.com/dkjnsdvsodvnosdvm/WinCenterTitle/blob/glass/AeroGlass.png)

## License

Hooking is done using the excellent [funchook](https://github.com/kubo/funchook) library (GPLv2 with linking exception), which in turn is powered by the [diStorm3](https://github.com/gdabah/distorm/) (3-clause BSD) disassembler. Thus, I am offering this under GNU General Public License Version 2.0, which I believe is compatible.

## Compiling

The following prerequisites are necessary in order to compile this project:

* Microsoft C/C++ Optimizing Compiler - this can be obtained by installing either of these packages:

  * Visual Studio - this is a fully featured IDE; you'll need to check "C/C++ application development role" when installing. If you do not require the full suite, use the package bellow.
  * Build Tools for Visual Studio - this just installs the compiler, which you'll be able to use from the command line, or from other applications like CMake

  Download either of those [here](http://go.microsoft.com/fwlink/p/?LinkId=840931). The guide assumes you have installed either Visual Studio 2019, either Build Tools for Visual Studio 2019.

* [CMake](https://cmake.org/) - for easier usage, make sure to have it added to PATH during installation
* Git - you can use [Git for Windows](https://git-scm.com/download/win), or git command via the Windows Subsystem for Linux.

Steps:

1. Clone git repo along with all submodules

   ```
   git clone --recursive https://github.com/dkjnsdvsodvnosdvm/WinCenterTitle
   ```

   If "git" is not found as a command, type its full path, or have its folder added to PATH, or open Git command window in the respective folder if using Git for Windows.

2. Compile funchook

   ```
   cd libs
   cd funchook
   md build
   cd build
   cmake -G "Visual Studio 16 2019" -A x64 ..
   cmake --build . --config Release
   ```

   If "cmake" is not found as a command, type its full path, or have its folder added to PATH.

   Type "Win32" instead of "x64" above, if compiling for x86. The command above works for x64.

3. Compile WinCenterTitle

   * Double click the WinCenterTitle.sln to open the solution in Visual Studio. Choose Release and your processor architecture in the toolbar. Press F6 to compile.

   * Open an "x86 Native Tools Command Prompt for VS 2019" (for x86), or "x64 Native Tools Command Prompt for VS 2019" (for x64) (search that in Start), go to folder containing solution file and type:

     * For x86:

       ```
       msbuild WinCenterTitle.sln /property:Configuration=Release /property:Platform=x86
       ```

     * For x64:

       ```
       msbuild WinCenterTitle.sln /property:Configuration=Release /property:Platform=x64
       ```

   The resulting exe and dll will be in "Release" folder (if you chose x86), or "x64\Release" (if you chose x64) in the folder containing the solution file.

That's it. later, if you want to recompile, make sure to update the repository and the submodules first:

```
git pull
git submodule update --init --recursive
```

