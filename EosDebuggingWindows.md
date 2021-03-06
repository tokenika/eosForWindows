# Debugging EOS on a Windows platform

Recently, we have shown how to install, compile and run EOS code on a Windows platform. This can be done by combining the [Windows Subsystem for Linux](https://msdn.microsoft.com/en-us/commandline/wsl/about) with the [*Visual Studio Code*](https://code.visualstudio.com/). Now, if EOS is compiled, it can be debugged. *Visual Studio Code* is ready for this purpose.

## Setting up
1. Install Ubuntu debugger:
```
sudo apt install gdb
```
2. In the vscode *EXPLORER* do: File > Open Folder ... <br>
browse where you keep the contents of the EOS repository.
3. Crt+Shift+P ... C/Cpp Edit Configurations <br>
opens *c_cpp_properties.json* file. Replace the contents of this file with the following text:
```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "/usr/include",
                "/usr/local/include",
                "/usr/local/include/openssl",
                "${HOME}/opt/boost_1_64_0/include/",
                "${EOSIO_INSTALL_DIR}"
            ],
            "defines": [],
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "/usr/include",
                    "/usr/local/include",
                    "/usr/local/include/openssl",
                    "${HOME}/opt/boost_1_64_0/include",    
                    "${EOSIO_INSTALL_DIR}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        },
        {
            "name": "Win32",
            "includePath": [
                "${localappdata}/lxss/rootfs/usr/include",
                "${localappdata}/lxss/rootfs/usr/local/include",
                "${localappdata}/lxss/rootfs/usr/include/openssl",
                "${localappdata}/lxss/home/cartman/opt/boost_1_64_0/include",
                "${EOSIO_INSTALL_DIR_WIN}"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE"
            ],
            "intelliSenseMode": "msvc-x64",
            "browse": {
                "path": [
                    "${localappdata}/lxss/rootfs/usr/include",
                    "${localappdata}/lxss/rootfs/usr/local/include",
                    "${localappdata}/lxss/rootfs/usr/include/openssl",
                    "${localappdata}/lxss/home/cartman/opt/boost_1_64_0/include",
                    "${EOSIO_INSTALL_DIR_WIN}"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 3
}
```
4. Set new environment variables that are used in the properties file. Note that `HOME` is the linux `HOME`, and `EOSIO_INSTALL_DIR` has been set prior to the compilation of EOS. (In our environment `HOME=/home/cartman`, `EOSIO_INSTALL_DIR=/mnt/e/Workspaces/EOS/eos`.) You can set Windows system variable `EOSIO_INSTALL_DIR_WIN` (where in the *Windows* file system is EOS), or you can replace them with literals. In our environment  `EOSIO_INSTALL_DIR_WIN=E:/Workspaces/EOS/eos`.

5. Click on the Configure gear icon on the *Debug View* top bar and VS Code will generate a launch.json file under your workspace's .vscode folder.
```
{
        "version": "0.2.0",
        "configurations": [
            {
                "name": "(gdb) Bash on Windows Launch",
                "type": "cppdbg",
                "request": "launch",
                "program": "${env:EOSIO_INSTALL_DIR}/build/programs/eosd/eosd",
                "args": [],
                "stopAtEntry": false,
                "cwd": "${env:EOSIO_INSTALL_DIR}",
                "environment": [],
                "externalConsole": true,
                "pipeTransport": {
                    "debuggerPath": "/usr/bin/gdb",
                    "pipeProgram": "C:/Windows/sysnative/bash.exe",  
                    "pipeArgs": ["-c"],
                    "pipeCwd": ""
                },
                "windows": {
                    "MIMode": "gdb",
                    "setupCommands": [
                    {
                        "description": "Enable pretty-printing for gdb",
                        "text": "-enable-pretty-printing",
                        "ignoreFailures": true
                    }
                    ]
                },
                "sourceFileMap": 
                {
                    "/mnt/e": "e:\\"
                }
            }
        ]
    }
```
6. Set new environment variable `EOSIO_INSTALL_DIR` (or replace it with a literal) that is used in the launch configuration file. It has the same value as the corresponding Ubuntu variable but it has to be available for *Windows*. (In our environment `EOSIO_INSTALL_DIR=/mnt/e/Workspaces/EOS/eos`.)

7. Open `programs\eosd\main.cpp` source file. Toggle a breakpoint somewhere close to the beginning of the code.

## Debug

Debug > Start Debugging
