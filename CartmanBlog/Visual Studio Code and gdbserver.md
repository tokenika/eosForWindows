Debugging C/C++ Programs Remotely Using Visual Studio Code and gdbserver
https://medium.com/@spe_/debugging-c-c-programs-remotely-using-visual-studio-code-and-gdbserver-559d3434fb78



With VS project properties > General > Remote Build Machine > localhost (username=cartman, port=22, authentication=Password)
The minimum version of CMake required on your Linux machine is 3.8, and it must also support server mode. 

Download the Visual Studio 2017 Preview, install the Linux C++ Workload, and try our CMake support for Linux with your code. Let us know how it is or isn’t working for you. Your feedback matters.
https://www.visualstudio.com/thank-you-downloading-visual-studio/?ch=pre&sku=Community&rel=15



## Visual Studio and WSL

Targeting the Windows Subsystem for Linux from Visual Studio
https://blogs.msdn.microsoft.com/vcblog/2017/02/08/targeting-windows-subsystem-for-linux-from-visual-studio/
Start SSH before connecting from Visual Studio:
$ sudo service ssh start
Note: You will need to do this every time you start your first Bash console. As a precaution, WSL currently tears-down all Linux processes when you close your last Bash console!.

Linux development with C++ in Visual Studio
//blogs.msdn.microsoft.com/vcblog/2017/04/11/linux-development-with-c-in-visual-studio/

Bring your existing C++ Linux projects to Visual Studio
https://blogs.msdn.microsoft.com/vcblog/2017/04/14/bring-your-existing-c-linux-projects-to-visual-studio/

Bring your C++ code to Visual Studio
https://blogs.msdn.microsoft.com/vcblog/2017/04/14/bring-your-cpp-code-to-visual-studio/

Visual C++ for Linux Development with CMake
https://blogs.msdn.microsoft.com/vcblog/2017/08/25/visual-c-for-linux-development-with-cmake/#buildcmake


### Install the newest cmake:
https://blogs.msdn.microsoft.com/vcblog/2017/08/25/visual-c-for-linux-development-with-cmake/#buildcmake

sudo apt update
cartman@southpark:/usr/local/src$ sudo apt install -y git cmake
sudo git clone https://github.com/Kitware/CMake.git
cd CMake
sudo git checkout tags/v3.9.0
sudo mkdir out
cd out
sudo cmake ../