<!-- omit in toc -->
# Setup Visual Studio Code for C++ Development

<!-- omit in toc -->
## Table of Contents

- [Why?](#why)
- [Pre-requisites](#pre-requisites)
  - [Linux](#linux)
  - [Windows](#windows)
- [Extensions for VS Code](#extensions-for-vs-code)
- [Makefile](#makefile)
  - [Linux](#linux-1)
  - [Windows](#windows-1)
- [VS Code Tasks](#vs-code-tasks)
  - [CMake Task](#cmake-task)
  - [Configure cmd and Visual Studio's environment variables](#configure-cmd-and-visual-studios-environment-variables)
  - [Make Task](#make-task)
- [Launcher](#launcher)
- [Snippets](#snippets)
- [Ressources](#ressources)

## Why?

The first question is: why doing this instead of using an IDE such as Visual Studio? I have two personal answers:
- When I work on Linux, I do not have access to Visual Studio. So, being allowed to use at least VS Code is very helpful.
- When I work with QML files, Visual Studio is not responsive enough. There is a very great extension under VS Code and that is why I chose this workflow (I will talk about it in another "How-to" but I do not really like Qt Creator IDE).

I am writing this Markdown file to make such a summary of everything I have learned on the Internet. I found a lot of ressources and I mixed them to get one workflow which, I guess, is pretty convenient for me. So, this is more like a reminder and if it can be helpful for anyone, that is cool! Ressources are at the bottom.

## Pre-requisites

### Linux

- C/C++ compiler (gcc, g++)
- CMake
- VS Code, of course

### Windows

- Visual Studio IDE (free version): we need its compiler **MSVC**
- CMake
- VS Code, of course

I am writing this Markdown file under Windows 10 with Visual Studio 2019 Community installed on my computer. So everything I will do will be specific to Windows (but it is quite the same on Linux, with less problems :wink:).

## Extensions for VS Code

There is some very pratical extensions for VS Code:
- [C/C++ by Microsoft](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)
- [CMake by twxs](https://marketplace.visualstudio.com/items?itemName=twxs.cmake) (optional but very handy when working with CMakeLists files)
- [CMake Tools by Microsoft](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools) (optional: I do not use it, personally)

## Makefile

Here, we are going to use CMake to generate a Makefile and not a Visual Studio solution (because, we do not work inside of Visual Studio)! Like this, manipulations are (almost) the same under Linux and Windows.

### Linux

The basic bash commands to generate a Makefile and build the project with it are:
```bash
cmake <path/to/CMakeLists.txt> # generate the Makefile
make # build the project thanks to the Makefile
```

### Windows

The basic commands to generate a Makefile and build the project with it are:
```bash
cmake <path/to/CMakeLists.txt> -G "NMake Makefiles"
nmake
```

:warning: Those commands will not work inside the Powershell or the *cmd* terminal. It is because they do not have access to Visual Studio's environment variables. To run those commands, you will need to run them from a Visual Studio terminal (I use *x64 Native Tools Command Prompt for VS 2019*). But fortunately, there is another way to run those commands from the basic *cmd* terminal. We will see that in a minute because VS Code has no access to Visual Studio's terminals.

## VS Code Tasks

:grey_exclamation: The first thing to do is to open the project folder with VS Code.

To automate the process of generating the Makefile and building the project, we can describe some tasks to VS Code and run those tasks whenever we want.

Inside the *Terminal* menu, you can click on *Configure Tasks...*, choose *Create tasks.json file from template* and then *Others*. It will give you a simple example of task.

### CMake Task

Let us create a new task to generate the Makefile and replace the template one. Something like this (remember: this command is relative to Windows):

```json
{
    "version": "2.0.0",
    "tasks": [
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
                "-DCMAKE_BUILD_TYPE=Debug"
            ]
        }
    ]
}
```

A little breakdown is in order:
- `version` : tasks version.
- `tasks` : array containing the different tasks (described inside an object).

A task itself is composed by:
- `label` : name of the task.
- `type` : can be a shell command or a process to execute.
- `options` : object with several attributes like `cwd`, `env` or `shell` which override default ones.
  - `cwd` : current working directory - the folder inside which the command is executed. In this case, we want to execute this command inside a *build* folder placed inside the workspace root folder (folder in which VS Code was opened).
- `command` : the command to execute. 
- `args` : 
  - `..` : here, we want to call *cmake* on the parent folder (so on workspace root folder). 
  - `-G` : to specify the build system generator. 
  - `"NMake Makefiles"` : the build system generator to use. Actually, it does not seem to work when we place it on the same line as the `-G` flag, so we place it just below.
  - `-DCMAKE_BUILD_TYPE=Debug` : this will make a debug build version.

:pencil: It is important to use double quotes around *NMake Makefiles* because simple quotes will not work with the *cmd* terminal.

Now, to run the task, just go to *Terminal*, *Run Task...* and select this one. If everything is right, it should not work. Do you remember when I talked about the fact Powershell cannot access to Visual Studio's environment variables? That is the problem here! There is several ways to deal with it.

- Starting VS Code from a *x64 Native Tools Command Prompt for VS 2019* terminal and like this, VS Code will know the specific environment variables.
- Call a specific batch file used to set those environment variables.
  - By default, the integrated shell is Powershell and Powershell cannot call a batch file. The workaround is to install a module in Powershell to allow us to call a batch file. But I do not want to install this external module so, let us take the other option!
  - We can configure our task to tell VS Code to use the *cmd* terminal instead of Powershell and to call this batch file inside it just before executing the command! Let us see how to do this.

### Configure cmd and Visual Studio's environment variables

We can add options to tasks. Either globally or for specific tasks. Here, we will add this configuration for all the tasks to avoid useless repetitions. We can place it juste before the `tasks` property, for instance.

```json
"options": {
    "shell": {
        "executable": "C:\\Windows\\System32\\cmd.exe",
        "args": [
            "/d", "/c", "call \"C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Auxiliary/Build/vcvars64.bat\" && "
        ]
    }
}
```

Here, the purpose is to set the *cmd* terminal as the default one for the tasks instead of Powershell: like this, we can call batch files. In the arguments, we specify two important flags `/d` and `/c`. Just after that, we call the batch file used to set all the environment variables. For me, this file is *vcvars64.bat*. This one is a specification of another batch file in the same directory called *vcvarsall.bat*. To know which one you should call, you can open your *x64 Native Tools Command Prompt for VS 2019* and see what batch file is called at the beginning.

Because of Windows, and I absolutely have no idea why, `C:\\Windows\\System32\\cmd.exe` needs to be written with double backslashes like this. Simple slashes do not work and executing the task fails.

### Make Task

Let us add another task to build the project now.

```json
{
    "label": "Make",
    "type": "shell",
    "options": {
        "cwd": "${workspaceRoot}/build"
    },
    "command": "nmake",
    "group": {
        "kind": "build",
        "isDefault": true
    }
}
```

Here, we tell VS Code to use this task as the default build task. Like this, we can easily call it with `Ctrl+Shift+B`.

So, the final *tasks.json* should look like this:

```json
{
    "version": "2.0.0",
    "options": {
        "shell": {
            "executable": "C:\\Windows\\System32\\cmd.exe",
            "args": [
                "/d", "/c", "call \"C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Auxiliary/Build/vcvars64.bat\" && "
            ]
        }
    },
    "tasks": [
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
                "-DCMAKE_BUILD_TYPE=Debug"
            ]
        },
        {
            "label": "Make",
            "type": "shell",
            "options": {
                "cwd": "${workspaceRoot}/build"
            },
            "command": "nmake",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

## Launcher

Once built, we can launch the project! For that, let us create a *launch.json* file in the *.vscode* folder.

Inside of the *Run* menu, select *Add Configuration...* and take the *C++ (Windows)* option. The template is quite good, we only have to adjust a few settings.

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
            "environment": [],
            "externalConsole": true
        }
    ]
}
```

Because we specified the flag `-DCMAKE_BUILD_TYPE=Debug`, if CMakeLists.txt takes it in count for the output path, the executable will be inside *build/Debug*. I set `externalConsole` to *true* because I prefer.

Now, you can launch the program in debug mode or not with `F5` or `Ctrl+F5`. You can also create another task to generate the release version of the program!

## Snippets

Now, we can make some templates using snippets! Actually, it is not possible to call snippets from *tasks.json* and *launch.json*. So the workaround is to add an underscore at the beginning of the file's name, for instance (just long enough to call the snippet and then, give it back its name).
Creating snippets for JSON can be really tedious and hard to read because snippets are written in JSON. It is a bit of JSON-ception, and the results are quite difficult to maintain but it can be cool anyway.

Here is an example for the tasks:

```json
{
    "tasks-build-cpp-template": {
        "scope": "json",
        "prefix": "tasks-build-cpp-template",
        "body": [
            "{",
                "\t\"version\": \"2.0.0\",",
                "\t\"options\": {",
                    "\t\t\"shell\": {",
                        "\t\t\t\"executable\": \"C:\\\\\\Windows\\\\\\System32\\\\\\cmd.exe\",",
                        "\t\t\t\"args\": [",
                            "\t\t\t\t\"/d\", \"/c\", \"call \\\"C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/VC/Auxiliary/Build/vcvars64.bat\\\" && \"",
                        "\t\t\t]",
                    "\t\t}",
                "\t},",
                "\t\"tasks\": [",
                    "\t\t{",
                        "\t\t\t\"label\": \"CMake Debug\",",
                        "\t\t\t\"type\": \"shell\",",
                        "\t\t\t\"options\": {",
                            "\t\t\t\t\"cwd\": \"\\${workspaceRoot}/build\"",
                        "\t\t\t},",
                        "\t\t\t\"command\": \"cmake\",",
                        "\t\t\t\"args\": [",
                            "\t\t\t\t\"..\",",
                            "\t\t\t\t\"-G\",",
                            "\t\t\t\t\"\\\"NMake Makefiles\\\"\",",
                            "\t\t\t\t\"-DCMAKE_BUILD_TYPE=Debug\"",
                        "\t\t\t]",
                    "\t\t},",
                    "\t\t{",
                        "\t\t\t\"label\": \"Make\",",
                        "\t\t\t\"type\": \"shell\",",
                        "\t\t\t\"options\": {",
                            "\t\t\t\t\"cwd\": \"\\${workspaceRoot}/build\"",
                        "\t\t\t},",
                        "\t\t\t\"command\": \"nmake\",",
                        "\t\t\t\"group\": {",
                            "\t\t\t\t\"kind\": \"build\",",
                            "\t\t\t\t\"isDefault\": true",
                        "\t\t\t}",
                    "\t\t}",
                    "\t]",
            "}"
        ],
        "description" : "Create a CPP Build task."
    }
}
```

As I said, it is quite difficult to write without an extension (I do not know if one exists or not for that). `\t` are used to mark tabulations and because we have sometimes double quotes inside double quotes, the escape characters are literally everywhere. :sweat_smile:

## Ressources

- [Compile C++ with VS Code and NMake](https://www.40tude.fr/compile-cpp-code-with-vscode-cmake-nmake/)
- [Using VS Code for Qt Apps](https://www.kdab.com/using-visual-studio-code-for-qt-apps-pt-1/)
- [Batch files and VS Code Tasks](https://www.reddit.com/r/raylib/comments/9napwf/wrote_batch_file_tasksjson_on_vscode_for_windows/)
- [Tasks in VS Code](https://code.visualstudio.com/docs/editor/tasks)
- [Configure VS Code for C++](https://code.visualstudio.com/docs/cpp/config-msvc)

And so many other forum threads!
