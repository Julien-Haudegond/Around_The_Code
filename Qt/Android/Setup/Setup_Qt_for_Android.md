<!-- omit in toc -->
# Setup Qt for Android

<!-- omit in toc -->
## Table of Contents

- [Why?](#why)
- [Pre-requisites](#pre-requisites)
  - [Java](#java)
  - [Android Studio](#android-studio)
  - [Qt](#qt)
  - [Setup Qt Creator for Android](#setup-qt-creator-for-android)
- [Create a Project](#create-a-project)
- [Run](#run)
- [Avoid using Qt Creator?](#avoid-using-qt-creator)
- [Resources](#resources)

## Why?

Qt has a very very very huge amount of documentation so, what is the purpose of this Markdown file? Actually, even with Qt's documentation and with some tutorials I found online, setup the Android environment with Qt was not so easy and I encountered several issues. That is why I want to share this experience.

This is written for Windows.

## Pre-requisites

### Java

Installing Java on Windows is like a nightmare! Why is it so complicated? Why Oracle needs my personal data? Address, society name, phone number... NO.

So, I wanted to find a workaround. I thought OpenJDK was also available on Windows, but Qt needs JDK version 8 to work and, I do not know if I am dumb or what, but I did not succeed to install Oracle OpenJDK version 8 on my Windows computer. So, I took a look at [AdoptOpenJDK](https://adoptopenjdk.net/) and it was surprisingly easy to install OpenJDK in its version 8. Thank you very much guys! :sunglasses:

### Android Studio

Qt says we can only download and use the Android SDK command-line tools. I tried but I did not succeed to make it work well (maybe because of other problems I discovered later, I do not know). Anyway, I decided to install [Android Studio](https://developer.android.com/studio) instead.

Then, in the SDK Manager:
- I installed several SDK Platforms, for different Android versions.
- I installed the SDK Tools. I had no idea which ones were important or not so I installed everything. :sweat_smile: I just did not install *CMake* because I already had it and *Android Emulator Hypervisor Driver for AMD Processors* because I run on an Intel processor.

:warning: You can check the *Show Package Details* to have a better overview of what is installed. Thanks to forum threads, I discovered later there are some *Android SDK Build-Tools* versions which do not work with Qt. I currently only have 28.0.3 and 29.0.2 versions installed on my computer.

### Qt

To be able to compile for Android, the Android kit should be installed via the *Qt Maintenance Tool*.

### Setup Qt Creator for Android

Inside *Tools/Options.../Devices/Android*, we have to set up some things:
- Java JDK path if not already found.
- Android SDK path + Android NDK path if not already found.
- Android OpenSSL settings if you want.

Then, we can create an AVD (Android Virtual Device) to emulate applications on it.

## Create a Project

To create a project for Android, let us click on *New Project*:
- Qt Quick Application (Empty)
- Location of the project
- Build system: CMake (for me, because I prefer CMake over QMake)
- Version of Qt which has the Android kits
- Kits (you can change the build directories for each kit (I recommand it)):
  - Desktop Qt 5.15.0 MSVC2019 64bit
  - Android for armeabi-v7a, arm64-v8a, x86, x86_64
- Version control

Inside the *Projects* tab, we can configure the build settings.
- If not already done, the first thing to do is to change the build directory. Because it is way too long (and it can be the cause of building errors).
- Then, we may need to change the architecture ABI in function of the target device.
- In the *Build Steps* panel, we can change the Android build SDK to decide the compatible version of Android.

## Run

Let us build and run the application on the AVD we previously created. And... surprise, it does NOT work! At least, for me, it does not work and I did not find how to resolve the problem yet. 

However, I found a workaround I do not understand myself. If I uncomment the Android if-statement in the *CMakeLists.txt* and launch the build again, it still does not work. BUT, if I comment back those lines and relaunch the build, it works! Why? I have absolutely no idea but, you know... :sweat_smile:

## Avoid using Qt Creator?

Okay, so, I am not a big fan of Qt Creator. And I do not really want to code inside this IDE. I think I will follow this workflow:
- Create a CMake project inside of Qt Creator.
- Open it in VS Code and configure VS Code to be able to build Desktop version.
- Writing code.
- Go back to Qt Creator just for building Android version.

We will see how it goes!

## Resources

Very very helpful: a lot more information is available here.

- [Getting Started with Qt for Android](https://doc.qt.io/qt-5/android-getting-started.html)
- [Android: Java vs Qt QML](https://evileg.com/en/post/328/)
- [Qt for Android](https://retifrav.github.io/blog/2017/12/28/qt-for-android/)
- [Getting Qt Creator working for Android Development](https://joshuatz.com/posts/2019/getting-qt-creator-working-for-android-development-first-steps/)
- [Getting Started with Qt Widgets in Android](https://hub.packtpub.com/getting-started-with-qt-widgets-in-android-video/)
- [Ma première application Android avec Qt](https://imikado.developpez.com/tutoriels/qml-javascript/ma-premiere-application-1/)
- [Qt for Android: Setup, Build, Deploy](https://www.youtube.com/watch?v=w2RRgRGHsDA)

Problems solutions:

- [What's the problem?](https://forum.qt.io/topic/101322/what-s-the-problem-android-compile-error/3)
- [Exception while generating APK](https://stackoverflow.com/questions/55491542/processexception-occures-in-qtcreator-while-generating-apk)
- [Known issues](https://wiki.qt.io/Qt_for_Android_known_issues)
