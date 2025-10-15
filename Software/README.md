# Project North Star: Software

This Unity Package contains the Unity Assets, Scenes, and Prewarping systems necessary to build Unity applications with the Project North Star headset.

## Compatibility 

**These assets require a multi device compatible service to display hands. A recent Gemini (v5) or Hyperion (V6) release should work**

These assets support more recent versions of the UnityPlugin, which should be imported as a package [Ultraleap Unity Modules](https://github.com/ultraleap/UnityPlugin). Prior version(s) bundled an older snapshot of the plugin within this unitypackage. This package has been tested with Unity 2022. Note: the legacy Ultraleap UnityPlugin package is required in addition to the tracking package. [Version 6 of the plugin should include this package] (https://github.com/ultraleap/UnityPlugin/releases/tag/com.ultraleap.tracking%2F6.15.1). Version 7 is not supported at present, due to breaking changes (namespaces). V7 is also not bundled with the legacy package and removes features such as the Interaction Engine in favour of Physical Hands.

Note: the calibration bars feature was not tested as part of this update.

[![North Star Starting Scene](/Software/imgs/UnityNorthStarRig.png)](https://github.com/leapmotion/ProjectNorthStar/tree/master/Software)

## Getting Started:
  - Make sure your North Star AR Headset is plugged in
  - In Windows 'display settings' make sure the headset is showing at the correct resolution (2880x1600) and is to the right of the main monitor.
  - Create a new Project in Unity 2022.3 LTS
  - Import "LeapAR.unitypackage"
  - Import the UnitypPlugin package (e.g. V6.15.1) using the package manager or by downloading the appropriate .unitypackage. You will need the Tracking package and the Ultraleap Tracking 6.0 Legacy package.
  - Navigate to `LeapMotion/North Star/Scenes/NorthStar.unity`
  - Click on the `ARCameraRig` game object and look for the `WindowOffsetManager` component
  - Here, you can adjust the X and Y Shift that should be applied to the Unity Game View for it to appear on the North Star's display
  - When you're satisfied with the placement; press "Move Game View To Headset"
  - With the Game View on the Headset, you should be able to preview your experience in play mode!
  
  Key Code Shortcuts in `NorthStar.unity` (in the Editor with the Game View in focus and playing)
  - `C` to Toggle Visibility of Calibration Bars


## Calibrating your Headset

We have included a pre-built version of the internal calibration tool.  We can make no guarantees about the accuracy of the process in DIY environments; this pipeline is built from multiple stages, each with multiple points of failure.  Included in the .zip file are a python script for calibrating the calibration cameras, a checkerboard .pdf to be used with that, and Windows-based Calibrator exe, and a readme describing how to execute the entire process.

## Click for friendlier video overview:

[![](http://img.youtube.com/vi/twyUk7MtiHo/0.jpg)](http://www.youtube.com/watch?v=twyUk7MtiHo "Calibration Overview")