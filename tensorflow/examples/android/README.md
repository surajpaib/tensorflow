# TensorFlow Custom Object Detector Demo using TF Detect

This folder contains an example application utilizing Object Detection using SSD ( via TensorFlow Object Detection API ) TensorFlow for Android
devices.

## Description

The demos in this folder are designed to give straightforward samples of using
TensorFlow in mobile applications.

Inference is done using the [TensorFlow Android Inference
Interface](../../../tensorflow/contrib/android), which may be built separately
if you want a standalone library to drop into your existing application. Object
tracking and efficient YUV -> RGB conversion are handled by
`libtensorflow_demo.so`.

A device running Android 5.0 (API 21) or higher is required to run the demo due
to the use of the camera2 API, although the native libraries themselves can run
on API >= 14 devices.

## Modules:

1. [TF Classify](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/android/src/org/tensorflow/demo/ClassifierActivity.java):
        Uses the [Google Inception](https://arxiv.org/abs/1409.4842)
        model to classify camera frames in real-time, displaying the top results
        in an overlay on the camera image.
2. [TF Detect](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/android/src/org/tensorflow/demo/DetectorActivity.java): [ We will be using this module for the demo]
        Demonstrates an SSD-Mobilenet model trained using the
        [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection/)
        introduced in [Speed/accuracy trade-offs for modern convolutional object detectors](https://arxiv.org/abs/1611.10012) to
        localize and track objects (from 80 categories) in the camera preview
        in real-time.
3. [TF Stylize](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/android/src/org/tensorflow/demo/StylizeActivity.java):
        Uses a model based on [A Learned Representation For Artistic
        Style](https://arxiv.org/abs/1610.07629) to restyle the camera preview
        image to that of a number of different artists.
4.  [TF
    Speech](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/android/src/org/tensorflow/demo/SpeechActivity.java):
    Runs a simple speech recognition model built by the [audio training
    tutorial](https://www.tensorflow.org/versions/master/tutorials/audio_recognition). Listens
    for a small set of words, and highlights them in the UI when they are
    recognized.

<img src="sample_images/classify1.jpg" width="30%"><img src="sample_images/stylize1.jpg" width="30%"><img src="sample_images/detect1.jpg" width="30%">

## Prebuilt App for testing Detection:

If you just want the fastest path to trying the demo, you may download the
nightly build
[here](https://ci.tensorflow.org/view/Nightly/job/nightly-android/). Expand the
"View" and then the "out" folders under "Last Successful Artifacts" to find
tensorflow_demo.apk.

## Running the Application

Once the app is installed it can be started via the  "TF Detect" icon, which has an orange TensorFlow logo as
their icon.

While running the activities, pressing the volume keys on your device will
toggle debug visualizations on/off, rendering additional info to the screen that
may be useful for development purposes.

## Building the Demo with TensorFlow using Bazel

Pick your preferred approach below. At the moment, we have full support for
Bazel, and partial support for gradle, cmake, make, and Android Studio.

[ Bazel needs to be used for TF Detect to work on Custom Models ]

As a first step for all build types, clone the TensorFlow repo with:

```
git clone --recurse-submodules https://github.com/tensorflow/tensorflow.git
```

Note that `--recurse-submodules` is necessary to prevent some issues with
protobuf compilation.

### Bazel

NOTE: Bazel does not currently support building for Android on Windows. Full
support for gradle/cmake builds is coming soon, but in the meantime we suggest
that Windows users download the [prebuilt
binaries](https://ci.tensorflow.org/view/Nightly/job/nightly-android/) instead.

##### Install Bazel and Android Prerequisites

Bazel is the primary build system for TensorFlow. To build with Bazel, it and
the Android NDK and SDK must be installed on your system.

1.  Install the latest version of Bazel as per the instructions [on the Bazel
    website](https://bazel.build/versions/master/docs/install.html).
2.  The Android NDK is required to build the native (C/C++) TensorFlow code. The
    current recommended version is 14b, which may be found
    [here](https://developer.android.com/ndk/downloads/older_releases.html#ndk-14b-downloads).
3.  The Android SDK and build tools may be obtained
    [here](https://developer.android.com/tools/revisions/build-tools.html), or
    alternatively as part of [Android
    Studio](https://developer.android.com/studio/index.html). Build tools API >=
    23 is required to build the TF Android demo (though it will run on API >= 21
    devices).

##### Edit WORKSPACE

The Android entries in
[`<workspace_root>/WORKSPACE`](../../../WORKSPACE#L19-L36) must be uncommented
with the paths filled in appropriately depending on where you installed the NDK
and SDK. Otherwise an error such as: "The external label
'//external:android/sdk' is not bound to anything" will be reported.

Also edit the API levels for the SDK in WORKSPACE to the highest level you have
installed in your SDK. This must be >= 23 (this is completely independent of the
API level of the demo, which is defined in AndroidManifest.xml). The NDK API
level may remain at 14.

##### Install Model Files (optional)

Download your models and place them in the [assets](assets) folder
Your model needs to be a .pb file ( Protobuf file). This .pb can be generated by exporting the inference graph. Training Details are mentioned below.
Along with this a .txt file that contains labels is needed. The standard format for this .txt file is

```
???
label1
label2
..
```

Where the labels are the name of objects trained for in an order corresponding to the .pbtxt file. ( See Training Process for more details )


##### Build

After editing your WORKSPACE file to update the SDK/NDK configuration, you may
build the APK. Run this from your workspace root:

```bash
bazel build -c opt //tensorflow/examples/android:tensorflow_demo
```

##### Install

Make sure that adb debugging is enabled on your Android 5.0 (API 21) or later
device, then after building use the following command from your workspace root
to install the APK:

```bash
adb install -r bazel-bin/tensorflow/examples/android/tensorflow_demo.apk
```

### Android Studio with Bazel

Android Studio may be used to build the demo in conjunction with Bazel. First,
make sure that you can build with Bazel following the above directions. Then,
look at [build.gradle](build.gradle) and make sure that the path to Bazel
matches that of your system.

At this point you can add the tensorflow/examples/android directory as a new
Android Studio project. Click through installing all the Gradle extensions it
requests, and you should be able to have Android Studio build the demo like any
other application (it will call out to Bazel to build the native code with the
NDK).

### Training Process

The model can be trained using the following instructions, to generate the .pb file that we need for our custom model.

(Training Instructions)[https://pythonprogramming.net/introduction-use-tensorflow-object-detection-api-tutorial/]
While running through the above demo, ensure that steps only to build the model are used