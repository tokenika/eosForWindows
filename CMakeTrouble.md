CMake.Quick Start:

CMakeTools extension was unable to initialize: [cmake-server] Failed to activate protocol version: "CMAKE_HOME_DIRECTORY" is set but incompatible with configured source directory value. [See output window for more details]

CMAKE_HOME_DIRECTORY:INTERNAL=/mnt/e/Workspaces/EOS/eos

## [CMake Tools Helper](#https://marketplace.visualstudio.com/items?itemName=maddouri.cmake-tools-helper)
This extension helps to bridge a gap between *C/C++* and *CMakeTools*.
CMake Tools Helper enables *C/C++* to automatically know the information parsed by CMake Tools (such as include directories and defines) and use it to provide auto-completion, go to definition, etc.

```
Loading CMake Tools from e:\Workspaces\EOS\eos\.vscode\.cmaketools.json
Detected available environment "Visual Studio Enterprise 2017 - x86
Detected available environment "Visual Studio Enterprise 2017 - amd64
Started new CMake Server instance with PID 15656
-- CMake Error: The current CMakeCache.txt directory e:/Workspaces/EOS/eos/build/CMakeCache.txt is different than the directory /mnt/e/Workspaces/EOS/eos/build where CMakeCache.txt was created. This may result in binaries being created in the wrong place. If you are not sure, reedit the CMakeCache.txt
```