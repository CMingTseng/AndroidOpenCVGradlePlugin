# AndroidOpenCVGradlePlugin

Gradle Plugin that will automate retrieving the Android OpenCV SDK and
linking it to the project, making it easy to include OpenCV into
Android applications.

## Usage

Currently the plugin is not yet published on public repositories. To
use the plugin, it requires to be compiled and published locally on the
machine for projects to be able to resolve and use it:

```
git clone https://github.com/ahasbini/AndroidOpenCVGradlePlugin.git
cd AndroidOpenCVGradlePlugin

# Either (on Windows):
gradlew.bat :plugin:publishToMavenLocal
# or (on *nix):
./gradlew :plugin:publishToMavenLocal
```

Once commands above complete to succession, the plugin is now located
in the local Maven repository of the machine (```.m2```) under
```com\ahasbini\tools\android-opencv-gradle-plugin```.

For usage in an Android Project, the below changes are needed:

1. In the project ```build.gradle``` (at the root directory of the
  project folder), modify the ```repositories``` and ```dependencies```
  block as below:

```
buildscript {
    repositories {
        mavenLocal() // At the beginning of the block
        // ... google() or jcenter() others
        maven { // At the end of the block and after google()
           url 'https://repo.gradle.org/gradle/libs-releases'
        }
    }
    dependencies {
        // ... the Android plugin and other classpath definitions
        classpath 'com.ahasbini.tools:android-opencv-gradle-plugin:0.0.+'
    }
}
```

2. In the ```app``` module (or application/library module that you're
  developing) ```build.gradle``` file, add the
  ```android-opencv-gradle-plugin``` plugin and the ```androidOpenCV```
  as below:

```
// apply plugin: 'com.android.application' or other Android plugin
apply plugin: 'com.ahasbini.android-opencv-gradle-plugin' // After the Android plugin

// ...

andoird {
    // ...
}

androidOpenCV { // After the android block

    // Required: Version of OpenCV to be used in the project
    version '3.3.0'

    // Optional: Custom url for downloading the
    // opencv-xxx-android-sdk.zip file located at
    // https://sourceforge.net/projects/opencvlibrary/files/opencv-android
    url 'https://sourceforge.net/projects/opencvlibrary/files/opencv-android/3.3.0/opencv-3.3.0-android-sdk.zip/download'
}

// ...
```

3. Optional: If the project did not contain any C++ code (usually
   located in ```jni``` or ```cpp``` folders under
   ```{project_app_module}/src/main/``` ), perform the below changes:

   * Add the below in ```app``` module (or application/library module
   that you're developing) ```build.gradle``` file:

    ```
    andoird {
        // ...
        externalNativeBuild {
            cmake {
                path "CMakeLists.txt"
            }
        }
    }
    ```

   * Create the ```CMakeLists.txt``` in ```app``` module (or
   application/library module that you're developing) directory and
   check the
   [Android Guides for NDK](https://developer.android.com/ndk/guides)
   or the [sample](sample) for more info.

4. Do a Gradle Sync
   <img src="https://developer.android.com/studio/images/buttons/toolbar-sync-gradle.png" width="16px" height="16px"/>,
   refresh linked C++ projects (**Build > Refresh Linked C++
   Projects**) and compile to make sure the integration was successful.

## Underlying Logic & Implementation

TL;DR The plugin downloads the ```opencv-xxx-android-sdk.zip```,
extracts the files, compiles the Java sources into AARs, and links them
along with JNI binaries into the project using ```dependencies``` and
```externalNativeBuild``` configurations.

In detail, below are the steps it carries our (primarily in this order)
which can be found mostly in
[AndroidOpenCVGradlePlugin.java](plugin/src/main/java/com/ahasbini/tools/androidopencv/AndroidOpenCVGradlePlugin.java):

 - Set the OpenCV JNI directory and arguments of the
 ```externalNativeBuild``` in the ```android``` block.
 - Extract the requested version of OpenCV from the ```androidOpenCV```
 block.
 - Check if existing or download the ```opencv-xxx-android-sdk.zip```
 into the directory ```{user_home}/.androidopencv/{version}``` using
 the url template
 ```"https://sourceforge.net/projects/opencvlibrary/files/" + version + "/opencv-" + version + "-android-sdk.zip"```
 or the value of ```url``` in ```androidOpenCV``` block.
 - Check if existing or extract the downloaded zip file.
 - Check if existing or copy the JNI directories (path used in the
 first step) into ```{project_module_directory}/build/androidopencv```.
 - Compile AAR binaries from Java source and place outputs (debug and
 release builds) in
 ```{user_home}/.androidopencv/{version}/build-cache``` using the
 [Gradle Tooling API](https://docs.gradle.org/current/userguide/embedding.html).
 - Add ```flatDir``` repository with
 ```{user_home}/.androidopencv/{version}/build-cache/outputs``` path
 and add dependencies ```debugImplementation``` and
 ```releaseImplementation``` with the AARs to project
 ```dependencies```.

## Contributing & Future Plans

As this is still under development, testing and not yet published, feel
free to share your contributions to the project with finding issues,
code improvements and/or feature additions and requests. Below is a
brief list of things (TODOs) that are in plan for the project:

 - [ ] Add Android NDK version checking and configuration for proper
 linking with compiled binaries of OpenCV
 - [ ] Add plugin ```clean``` task
 - [ ] Add more tests and assertions in test cases
 - [ ] CI/CD integration