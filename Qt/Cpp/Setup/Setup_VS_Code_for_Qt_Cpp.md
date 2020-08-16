# Setup VS Code for Qt C++

## Table of Contents
- [Why?](#why)
- [Pre-requisites](#pre-requisites)
- [Change Tasks](#change-tasks)
- [Change Launcher](#change-launcher)
- [C++ Configuration](#c-configuration)
- [Ressources](#ressources)

## Why?

Qt is an amazing framework for building applications. It is fully supported by the Qt IDE, called *Qt Creator*. The problem is, I personally, hate this IDE. I do not really know why but I really do not like coding inside of Qt Creator. That is why I used either Visual Studio or Visual Studio Code (which has a powerful extension for QML-support).

A simple workflow would be to create the CMake project inside of Qt Creator and then, move to VS Code.

This Markdown file may have some Windows specificities because I am currently running on Windows (I do not know how it goes on Linux or OSX).

## Pre-requisites

- Qt installed on the computer (with Qt Creator)
- [Setup VS Code for C++](https://github.com/Julien-Haudegond/Around_The_Code/blob/master/Cpp/Setup/Setup_VS_Code_for_Cpp.md)

## Change Tasks

To be able to generate the Makefile, CMake will need the path to Qt installation. Let us add it inside the `args` property.

Inside *tasks.json*:

```json
{
    "label": "CMake Debug",
    "type": "shell",
    "options": {
        "cwd": "${workspaceRoot}/build"
    },
    "command": "cmake",
    "args": [
        "..",
        "-G",
        "\"NMake Makefiles\"",
        "-DCMAKE_BUILD_TYPE=Debug",
        "-DCMAKE_PREFIX_PATH=\"C:/Qt/5.15.0/msvc2019_64\""
    ]
}
```

## Change Launcher

We also need to add the Qt DLLs to the path. Specifying the *bin* and the *plugins/platforms* directories seems to work fine.

Inside *launch.json*:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(Windows) Launch",
            "type": "cppvsdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/Debug/executable.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [
                {
                    "name": "PATH",
                    "value": "C:/Qt/5.15.0/msvc2019_64/bin;C:/Qt/5.15.0/msvc2019_64/plugins/platforms"
                }
            ],
            "externalConsole": true
        }
    ]
}
```

## C++ Configuration

Press `F1` or `Ctrl+Shift+P` to open the commands and type *C/C++: Edit Configurations (JSON)*. There, we can add Qt includes inside `includePath` property. Like this, VS Code will know Qt and propose auto-completion.

```json
{
    "includePath": [
        "${workspaceFolder}/**",
        "C:/Qt/5.15.0/msvc2019_64/include/**"
    ]
}
```

## Ressources

- [Using VS Code for Qt Apps](https://www.kdab.com/using-visual-studio-code-for-qt-apps-pt-1/)
- [VS Code CMake Qt](https://retifrav.github.io/blog/2019/05/11/vscode-cmake-qt/)

And so many other forum threads!
