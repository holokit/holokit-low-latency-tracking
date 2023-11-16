# HoloKit Low Latency Tracking

HoloKit uses ARKit's spatial tracking feature to get the 6DoF (Degrees of Freedom) pose of the user's head. ARKit, designed for iOS devices, is primarily tailored for screen-based AR experiences, where real-time camera background images are displayed on the screen. To ensure stability, the mobile device introduces a delay in the camera video stream, enhancing the integration of real and virtual content. However, in the case of HoloKit, an optical-see-through (OST) headset, this delay causes a misalignment between virtual content and the real world, especially noticeable when the user moves their head.

The objective of the HoloKit low latency tracking algorithm is to reduce latency by using the device's raw accelerometer and gyroscope data to refine ARKit's output pose data. [Aryzon SDK](https://github.com/Aryzon/unity-sdk)'s approach addresses the limitations of [Google Cardboard](https://github.com/googlevr/cardboard), which relies solely on device accelerometer and gyroscope data for real-time 3DoF pose calculations. Aryzon SDK's approach enhanced this by integrating ARKit's 6DoF pose data with Google Cardboard's 3DoF data, creating a fused 6DoF pose with lower latency than ARKit's standalone output. Following this model, we adapted and packed the necessary native Objective-C++ code from [Aryzon's modified Google Cardboard repository](https://github.com/Aryzon/cardboard/tree/main). Additionally, we wrote a bridge file to facilitate two-way marshalling between unmanaged (Objective-C++) and managed (C#) codebases.

## Native Marshalling Interface

The file `HoloKitLowLatencyTracking/unity_c_bridge.cc` comprises native marshalling interface functions accessible from the C# side. Each function serves a specific purpose in the low latency tracking system:

- `HoloInteractiveHoloKit_LowLatencyTracking_init`: Initializes the native side of the low latency tracking system.

- `HoloInteractiveHoloKit_LowLatencyTracking_initHeadTracker`: Activates the head tracker subsystem of the low latency tracking system, typically called after the system's initialization.

- `HoloInteractiveHoloKit_LowLatencyTracking_pauseHeadTracker`: Temporarily pauses the head tracker subsystem.

- `HoloInteractiveHoloKit_LowLatencyTracking_resumeHeadTracker`: Resumes operation of the head tracker subsystem.

- `HoloInteractiveHoloKit_LowLatencyTracking_addSixDoFData`: Incorporates new ARKit 6DoF pose data into the system whenever ARKit outputs it.

- `HoloInteractiveHoloKit_LowLatencyTracking_getHeadTrackerPose`: Retrieves the latest predicted head pose of the user.

- `HoloInteractiveHoloKit_LowLatencyTracking_delete`: Releases the native pointer associated with the low latency tracking system.

## How `LowLatencyTrackingManager` Script Works

The `[LowLatencyTrackingManager](https://github.com/holoi/holokit-unity-sdk/blob/main/Runtime/LowLatencyTrackingManager.cs)` script from HoloKit Unity SDK serves as the central component for managing low latency tracking on the C# side.

Its `Start()` function executes three critical steps:

- It sets up the `ARCameraManager.frameReceived` callback, indicating when a new 6DoF pose is calculated by ARKit. This data is subsequently fed into the native low latency tracking system.

- It establishes the `Application.onBeforeRender` callback. This is used to fetch the estimated 6DoF pose from the native low latency tracking system right before rendering, ensuring the camera pose is updated in real-time.

- It initializes the native side of the low latency tracking system, setting the groundwork for the tracking process.

By handling these callbacks and the native system initialization, `LowLatencyTrackingManager` ensures that the camera pose is always synchronized with the latest pose data from ARKit, minimizing latency and enhancing the AR experience.
