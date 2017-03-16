title: "Using Native OpenCV Code in Android Projects"
date:  2016-03-15 12:00:00
tags:
 - Android
 - OpenCV
---
There are many ways to integrate the OpenCV library into Android projects. The simplest way is to invoke the OpenCV Manager APIs provided by the *OpenCV Manager*, an individual app containing the actual library, which is published and maintained by the official OpenCV dev group and must be installed on the device before running apps that require OpenCV. This using model is the easiest one to be implemented among all the methods but it seems to be the worst one for the end users because they need to install another “useless” app on their phones.

A better way than using the OpenCV Manager is to import the OpenCV4Android distribution into the projects. Generated library files can be found in the SDK and the download link can be found directly on the OpenCV [home page][1]. You can find the libraries in the *sdk/native/libs* directory. Copy them into your *src/main/jniLibs* directory and you can use the OpenCV Java APIs. Writing OpenCV codes in Java seems to be attracting because of the familiar language features like garbage collection and the not-so-bad performance as the computations are performed on the native level via JNI.

However, both the two methods above have the same drawback that, when you’re performing too much API calls to the OpenCV, the extra cost of JNI calls will sorely slow down the computations. Besides, the Java APIs are not the same as the native C++ APIs of OpenCV. For example, you cannot find an alternative in the Java APIs to the Vec class used widely in the C++ side, so when you are “translating” some C++ OpenCV codes into Java, you might easily get confused by the differences of the two sets of APIs.

- - -

Thus when you require the maximum performance or you need 100% API support or if you’ve got some native C++ OpenCV code that you want to use directly in your project, the only way is to import the native OpenCV library into your projects.

Writing native code means you need to set up [NDK][2] for your development environment. Remember to add the path to NDK into your PATH environment variable. I’m assuming that you’ve already installed and configured the NDK properly. If not, you may find an official guide provided [here][3].

## Prepare the OpenCV Library

First of all, you need to prepare the OpenCV library. Download the OpenCV for Android SDK from the OpenCV [home page][4]. Extract the zip file to somewhere you like. Go to the *sdk/native/jni* directory, and you can find many *.mk* and *.cmake* files here along with the C++ header files in *include*. Create a directory named *jni* in the *app/src/main* directory of your Android project (replace app with your module’s name if you are importing the OpenCV library into another module), and then copy all the files and directories from OpenCV’s *sdk/native/jni* to your newly created jni directory.

After that, copy *3rdparty* and *libs* two folders from OpenCV’s *sdk/native/* to *app/main/*. These directories contain the libraries that are required to build the native OpenCV code.

Finally, you should import the OpenCV Java code as a module into your project. Select *File* \> *New* \> *Import Module* in the Android Studio’s menu bar, browse the location of the *sdk/java* directory of the extracted OpenCV library, and type in a name you’d like to use (e.g., *open-cv*). You’ll need to use the Java interfaces of the OpenCV Android SDK when interacting with the native C++ code and share types like Mat from OpenCV. Remember to open the module settings of your *app* module and add the *open-cv* module as a dependency.

## Get the C++ Code Ready

After importing the OpenCV library into your project, it’s time to get use of your C++ code. In order to use native C++ code in your Android project, you need to make a NDK module and build it into a library file to be included in the APK files (a NDK module is different from a module in an Android project, which is just a logical project containing a bunch of code and configuration files).

Create two files named *Android.mk* and *Application.mk* in the *jni* directory, which is required for the Android NDK’s `ndk-build` tool to build the JNI project. You can find more about the two *.mk* files at [here][5] and [there][6]. Basically you can write them like this:

> You can find the explanations of the code at the [Android Development with OpenCV][7] tutorial.

*Android.mk*

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

OPENCV_INSTALL_MODULES:=on
OPENCV_CAMERA_MODULES:=on
OPENCV_LIB_TYPE := STATIC

include $(LOCAL_PATH)/OpenCV.mk

LOCAL_MODULE := CvExample
LOCAL_SRC_FILES := com_example_yourapp_CvUtil.cpp cv.cpp
LOCAL_C_INCLUDES := $(LOCAL_PATH)/include
LOCAL_LDLIBS += -lm -llog

include $(BUILD_SHARED_LIBRARY)
```

*Application.mk*

```
APP_STL := gnustl_static
APP_CPPFLAGS := -frtti -fexceptions
APP_ABI := all
APP_PLATFORM := android-15
```

Some key points you need to pay attention to when writing your own .mk files:

 1. `CLEAR_VARS` clears many `LOCAL_XXX` variables for you, which must be placed above all the other `include`s and variable declarations.
 2. Setting `OPENCV_INSTALL_MODULES` to `on` will copy necessary OpenCV dynamic libraries to the *libs* directory and include them into APK files. `OPENCV_CAMERA_MODULES` is used to enable native OpenCV camera related libraries.
 3. `LOCAL_MODULE` defines the name that will be used to name your compiled native library. For example, if you set `LOCAL_MODULE` to `CvExample`, then the compile library will be named as *libCvExample.so*.
 4. `LOCAL_SRC_FILES` includes the C++ programs that you write. In the example above the first file *com\_example\_yourapp\_CvUtil.cpp* is a C++ class generated by JNI, which will be introduced later. The second one is the C++ file that contains your native OpenCV code.
 5. `APP_STL` in *Application.mk* chooses a runtime to provide Standard C++ Library support for your native code. You can find a list of runtimes and the differences between them [here][8].
 6. `APP_PLATFORM` defines the target Android version to build the native library. You must install the corresponding Android platform and build tools in the Android SDK Manager before building your native code.

In order to use the JNI module you’ve created, you should clear the default JNI source directories in *build.gradle* of your *app* module. Add a `sourceSets` part into `android` in the file and write something like:

```
android {
    ...
    
    sourceSets {
        main {
            jni.srcDirs = []
        }
    }
}
```

With the NDK project configured, you can now copy your OpenCV C++ programs into the *jni* directory. But in order to call the native C++ methods from your Java code, you have to use *Java Native Interface* (JNI) which acts as a bridge between Java and C++. You can find the official introduction and guide to the JNI [here][9]. Here I’ll provide a simple example of creating a JNI.

### 1. Write a Java class.

The Java class should contain the methods that you would like to call “from the C++ code” by which I mean the Java methods declarations are connected through the JNI with the C++ methods definitions.

Let’s suppose that you need to send a `Mat` with a float threshold value from the Java code to your native C++ code. Then you should generate some data from the `Mat` and the threshold and return it back to the Java code. To help you understand how the complex data types should be dealt with in JNI, I’m assuming the return type is an ArrayList of (two) float arrays which both contain 10 elements, complex enough. You can define the Java class like this:

```java
package com.example.yourapp;

import android.util.Log;

import org.opencv.android.OpenCVLoader;
import org.opencv.core.Mat;

import java.util.ArrayList;

public class CvUtil {
    static {
        if (!OpenCVLoader.initDebug()) {
            Log.d("Error", "Unable to load OpenCV");
        } else {
            System.loadLibrary("CvExample");
        }
    }
    
    public static ArrayList<float[]> processMat(Mat mat, float threshold) {
        return processMat(mat.getNativeObjAddr(), threshold);
    }
    
    public static native ArrayList<float[]> processMat(long matAddr, float threshold);
}
```

The `static` part of this class loads the OpenCV library and the library that your C++ code will be built into. The `OpenCVLoader#initDebug` method loads the OpenCV library in debugging mode, which will load the lib instead of using the OpenCV Manager APIs. `CvExample` is what we named the NDK module in the *Android.mk* file.
 
Two methods named `processMat` are declared. The first method is the one that should be called, which will convert the Mat into a native object address and pass it to the native code. The second one with the `native` modifier has its method body defined in the C++ code.

### 2. Generate a JNI C++ class.

After creating the Java class, you can generate a JNI C++ class with the `javah` command in your terminal. For example:

```
$ cd your_app_project/app/src/main/jni
$ javah -jni -d ./ -cp ~/Library/Android/sdk/platforms/android-15/android.jar:../../../../open-cv/src/main/java:../java com.example.yourapp.CvUtil
```

> Read more about the javah tool at [Oracle’s Java Documetation][10].

The `-cp` argument defines the classpaths from which `javah` should find the classes, separated by a colon in Unix or a semicolon in Windows. Here the first path points to an *android.jar* of the version same to the one you specified in *Application.mk*. The second one is the *java* folder of your *open-cv* module in your Android project, providing references to the OpenCV Java classes and interfaces. The last path is where your Java files is located.

After executing the command, a header file named *com\_example\_yourapp\_CvUtil.h* should be generated in the *jni* folder. The content should be like:

```c++
/\* DO NOT EDIT THIS FILE - it is machine generated \*/
# include \<jni.h\>
/\* Header for class com\_example\_yourapp\_CvUtil \*/

# ifndef \_Included\_com\_example\_yourapp\_CvUtil
# define \_Included\_com\_example\_yourapp\_CvUtil
# ifdef \_\_cplusplus
extern "C" {
# endif
/\*
 * Class:     com\_example\_yourapp\_CvUtil
 * Method:    processMat
 * Signature: (JF)Ljava/util/ArrayList;
 \*/
JNIEXPORT jobject JNICALL Java\_com\_example\_yourapp\_CvUtil\_processMat
  (JNIEnv \*, jclass, jlong, jfloat);

# ifdef \_\_cplusplus
}
# endif
# endif
```

The `CvUtil#processMat` method is converted into a C++ function declaration. Be careful with the signature in the method’s comment. A signature is by what a method can be identified. You can see the method arguments are represented as `(JF)`, `J` for `long` and `F` for `float`. The return type is represented as `Ljava/util/ArrayList`, and the `L` stands for “fully qualified class”, which means it is a Java class referred to by the following qualifier `java/util/ArrayList`. Now that the header is generated, you need to implement the function in a separate *.cpp* file.

> Read more about the types and signatures in JNI at [JNI Types and Data Structures][11].

### 3. Provide your C++ code.

Create a file next to the *.h* file generated named *com\_example\_yourapp\_CvUtil.cpp*. Now you can implement the function with the C++ OpenCV program you’ve already got — just include it into this C++ code and call the functions from that one. Here I will only show how to parse the data provided by the function call and how to return data back to Java.
There are four arguments with the generated function declaration. The first one is a pointer to JNIEnv which is a environmental object of the JNI providing many useful functions (sounds like Context in Android). The second argument type is jclass, which is a reference to a class in Java. Here it is the class of which the method belongs to. The following two arguments are what we will pass from the Java caller to this C++ function, in which `jlong` and `jfloat` can be viewed as a C++ wrapper for Java’s primitive types `long` and `float`. Let’s name the four arguments like the code below:

```c++
JNIEXPORT jobject JNICALL Java\_com\_example\_yourapp\_CvUtil\_processMat(
  JNIEnv \* env, jclass cls, jlong mat\_addr, jfloat threshold) {
```

The `mat_addr` is a native object address as mentioned above, which means you can directly create a `Mat` from this address like:

```c++
cv::Mat *image = (cv::Mat *) image_addr;
```

Now you can process the `Mat` and prepare to return some data. Here’s how an `ArrayList<float[]>` object is created:

```c++
// Find the Java class of ArrayList
jclass arrayListClass = (\*env).FindClass(“java/util/ArrayList”);
// Create an instance by finding the method ID of the constructor
jobject arrayList = (*env).NewObject(arrayListClass, (*env).GetMethodID(arrayListClass, “<init>”, “()V”));

// Find the class of float
jclass floatClass = env-\>FindClass(“F”);
// Let’s assume a float array contains 10 elements
jsize size = 10;
// Create arrays
jobjectArray array1 = env-\>NewObjectArray(size, floatClass, NULL);
jobjectArray array2 = env-\>NewObjectArray(size, floatClass, NULL);
```

Now we’ve created two float arrays, how can we put values into these arrays? Can we directly put C++ `float` values into Java `float` arrays? Absolutely not. Although JNI provides methods like `JNIEnv#SetObjectArrayElement` which can set the value of an element in a `jobjectArray`, you just cannot use the C++ data types here. You must use `java.lang.Float` instead here, because the `Float` class has a constructor so that we can put the value into it using the method ID of the constructor.

```c++
// Find the class of Float
jclass FloatClass = env-\>FindClass(“java/lang/Float”);
// And the method ID of constructor of the Float class
jmethodID FloatInit = env-\>GetMethodID(FloatClass, “<init>”, “(F)V”);
for (int i = 0; i \< 10; i += 1) {
    jsize index = i;
    // Create instances of Float
    jobject value1 = env->NewObject(FloatClass, FloatInit, xxx);
    jobject value2 = env->NewObject(FloatClass, FloatInit, xxx);
    // Put the Float values into float arrays
    env->SetObjectArrayElement(array1, index, value1);
    env->SetObjectArrayElement(array2, index, value2);
}

// Now let’s put the float arrays into the ArrayList
jmethodID addElementMethod = env-\>GetMethodID(arrayListClass, “add”, “(Ljava/lang/Object;)Z”);
env-\>CallBooleanMethod(arrayList, addElementMethod, array1);
env-\>CallBooleanMethod(arrayList, addElementMethod, array2);

return arrayList;
```

Huge work to be done for a simple task, isn’t it?

## Build Your Native Library

Android doesn’t provide the ability to run the C++ code directly on your phone. Instead, you should build your native module into a library for your app to use.

The `ndk-build` tool from the NDK is used to build your native module. You can use `ndk-build` in your terminal, but there’s a more convenient way of doing this. You can write some tasks in your *build.gradle* to ask gradle to build it for you every time you compile your project. The task is created like:

```
task ndkBuild(type: Exec, description: ‘Compile JNI source via NDK’) {
    println(‘executing ndkBuild’)
    def ndkDir = project.android.ndkDirectory
    commandLine “$ndkDir/ndk-build”,
        ‘NDK_PROJECT_PATH=build/intermediates/ndk’,
        ‘APP_BUILD_SCRIPT=src/main/jni/Android.mk’,
        ‘NDK_APPLICATION_MK=src/main/jni/Application.mk’,
        ‘V=1’
}
```

Set the `ndkBuild` task as a dependency of gradle’s `JavaCompile` tasks:

```
tasks.withType(JavaCompile) {
    compileTask -> compileTask.dependsOn ndkBuild
}
```

Build your project and you will find the generated library files at *app/build/intermediates/ndk/libs* folder of your project. You need to copy them to your *app/src/main/jniLibs* directory. Also, you should copy *libopencv\_java3.so* from OpenCV’s *sdk/native/libs* into *jniLibs* too. Now you can call the method you declared in the `CvUtil` class and thus make use of your native OpenCV programs.

Whenever you build your project, your JNI module will be built automatically. If you’ve made any changes to the native code, don’t forget to copy the latest libs into your `app/src/main/jniLibs` directory.

[1]:  http://opencv.org/
[2]:  https://developer.android.com/ndk/index.html
[3]:  https://developer.android.com/ndk/guides/index.html
[4]:  http://opencv.org/
[5]:  https://developer.android.com/ndk/guides/android_mk.html
[6]:  https://developer.android.com/ndk/guides/application_mk.html
[7]:  http://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/dev_with_OCV_on_Android.html#native-c
[8]:  https://developer.android.com/ndk/guides/cpp-support.html
[9]:  https://docs.oracle.com/javase/7/docs/technotes/guides/jni/
[10]: https://docs.oracle.com/javase/8/docs/technotes/tools/unix/javah.html
[11]: https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/types.html#wp9502
