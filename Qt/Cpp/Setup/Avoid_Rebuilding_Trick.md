<!-- omit in toc -->
# Avoid Rebuilding Trick

<!-- omit in toc -->
## Table of Contents

- [Qt Resource System](#qt-resource-system)
- [The little trick](#the-little-trick)
  - [CMake variable](#cmake-variable)
  - [Ifdef](#ifdef)
- [Resources](#resources)

## Qt Resource System

When working with Qt, we can use the **Qt Resource System** (or not).

Here is a brief explanation of what the Qt Resource System is (taken from [this article](https://www.kdab.com/embedding-qml-why-where-and-how/) of KDAB).
```
Standard Qt resources—icons, bitmaps, audio files, translation tables, etc—can be compressed and linked to the executable, allowing you to create a distribution with fewer external dependencies. Not only is this simpler to manage for versioning and installation, but it provides a measure of confidence that your application will always have the resources it depends on to run properly.
```

If you want to build an application for Android, for instance, you must use the Qt Resource System.

The only "problem" is, while developing, each time you make a modification in one of the QML files, you need to build again the app (and actually, it is quite annoying).

## The little trick

Here is the little trick I have found to avoid this rebuilding process.

:warning: I have absolutely no idea of the fiability of this method.

### CMake variable

Add this specific variable in the CMakeList file (of course, I use CMake and not QMake).

```
if(INSTANT_RELOAD)
    message("Instant Reload is enabled.")
    add_compile_definitions(INSTANT_RELOAD)
endif()
```

Then, when you call your CMake command, just add this variable:

```bash
cmake .. -DINSTANT_RELOAD=True
```

This will create a new definition accessible from the C++ code.

:warning: If you called the CMake command once with the variable enabled, the CMakeCache will know it even if you relaunch the command without the variable. To avoid this, you must call it with *False* or clean up the whole CMakeCache directory.

### Ifdef

When we instantiate the QML engine (or every time we use *qrc* I guess), we just check the variable with a *ifdef* statement:

```cpp
#ifdef INSTANT_RELOAD
    engine.addImportPath(QStringLiteral("src/qml")); // For QML Plugins
    const QUrl url(QStringLiteral("src/qml/App.qml")); // Avoid use of RCC during development
#else
    engine.addImportPath(QStringLiteral("qrc:///src/qml")); // For QML Plugins
    const QUrl url(QStringLiteral("qrc:///src/qml/App.qml"));
#endif
```

Like this, we tell the C++ code to use the RCC only when we do not want the *Instant Reloading*.

Actually, I do not really understand yet how it really works, but it seems to make the job so, that is okay for now! :sweat_smile:

## Resources

- [Qt Resource System](https://doc.qt.io/qt-5/resources.html)
- [Embedding QML](https://www.kdab.com/embedding-qml-why-where-and-how/)
- [Live Reloading](https://qml.guide/live-reloading-hot-reloading-qml/)