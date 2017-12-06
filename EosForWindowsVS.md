We have made several attempts with Windows C++ compilers, but to no avail. The main problem is that, while the EOS code is branched with the `WIN32` flag, unfortunately it's been done inconsistently. Also, Windows *Clang* compiler behaves differently than its Unix counterpart - it seems to be more restrictive.

However, we have found it possible now, with the newest editions of MS Visual Studio combined with the [Windows Subsystem for Linux](https://msdn.microsoft.com/en-us/commandline/wsl/about) to have EOS code compiled and debugged with Visual Studio. 

# Prerequisites

You need to be running [Windows 10, version 1703, also known as the Creators Update](https://docs.microsoft.com/en-us/windows/whats-new/whats-new-windows-10-version-1703). To verify your Windows version open the *Settings Panel* and then navigate to `System > About`.

If you have an earlier version of Windows 10 and for some reasons don't want to upgrade to version 1703, you might still be able to give it a try, provided you manage to upgrade your *Windows Subsystem for Linux* from Ubuntu 14 to Ubuntu 16.

*Windows Subsystem for Linux* can be only installed on the system drive. If you're running low on your disk space (extra 3-4 GB will be needed), you might want to expand your system partition using tools available [here](https://www.partition-tool.com/).

# Tooling up

First we will enable *Windows Subsystem for Linux* and then we will access the Bash command line interface from within *Visual Studio*. 

### Windows Subsystem for Linux

The official *Windows Subsystem for Linux* installation guide is to be found [here](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide). However, for your convenience, we provide the following simplified procedure:

1. Open *PowerShell* in the administrator mode (right-click and choose `Run as Administrator`) and execute this command: 
```
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```
2. Restart your computer when prompted.

3. Turn on the *Developer Mode*:
    -- Open the *Settings Panel* and navigate to `Update & Security > For developers`.
    -- Select the *Developer Mode* radio button.
    -- If prompted, restart your computer again.

4. Enter `bash` into the *Windows PowerShell* prompt. After you have accepted the license agreement, a Ubuntu user-mode image will be downloaded and extracted to `%localappdata%\lxss\`. You will be prompted for setting up a Linux user name and password. When the process is finished, a shortcut named *Bash on Ubuntu on Windows* will be added to your *Start Menu*.

5. Once you are inside the Linux shell, make sure you are running Ubuntu 16:
```
lsb_release -a
```
Note that you can copy anything from here with the usual method, `Ctr+C`, for example, and copy it to the *Windows PowerShell* prompt by clicking the right mouse. Similarly, you can mark anything in the *Windows PowerShell* (dragging with the left mouse), and then copy it by clicking the right mouse.

7. And finally update & upgrade Ubuntu:
```
sudo apt update
sudo apt full-upgrade
sudo apt install -y build-essential
```

### Redoing the Ubuntu installation

The following step is optional. Use it only if, for some reasons, you need to start over and redo the Ubuntu installation.

1. Open Windows command prompt and run:
```
lxrun /uninstall /full
lxrun /install
```

2. Run `bash` to launch a new Ubuntu shell and inside it run update & upgrade:
```
sudo apt update
sudo apt full-upgrade
sudo apt install git
```

### Visual Studio

We use a [preview version of VS](#https://www.visualstudio.com/vs/preview/), perhaps a newly updated regular version would do, as well. Download *Visual Studio Installer*, check *Desktop development with C++* and *Linux Development with C++* workloads, at least, and, get *Visual Studio* installed.

# Setting up

EOS repository is to be placed in a folder accessible for both, Windows and Ubuntu systems. 

Visual Studio controls Ubuntu remotely via an SSH link, and hence, it can invoke *cmake* processed down in the Ubuntu. So, the code can be compiled with the linux compiler.

Debugging is achieved due to the linux compiler, fitted with a specialized SSH server.

Not any *cmake* installation is good for our purpose, especially the one default for the current *Ubuntu 17.xx* is enough: we need v3.8.0, at least.

Following procedure prepares conditions for the set-up. We have learnt them from Marc Goodner [there](#https://blogs.msdn.microsoft.com/vcblog/2017/02/08/targeting-windows-subsystem-for-linux-from-visual-studio/) and [there](#https://blogs.msdn.microsoft.com/vcblog/2017/08/25/visual-c-for-linux-development-with-cmake/#buildcmake).

```
sudo apt install gdb
sudo apt install -y gdbserver
sudo apt install -y openssh-server
sudo nano /etc/ssh/sshd_config
```
Scroll down the “PasswordAuthentication” setting and make sure it’s set to “yes”. Hit CTRL + X to exit, then Y to save.

Now generate SSH keys for the SSH instance:
```
$ sudo ssh-keygen -A
```
Start SSH before connecting from Visual Studio:
```
$ sudo service ssh start
```
**Note: You will need to do this every time you start your first Bash console. As a precaution, WSL currently tears-down all Linux processes when you close your last Bash console!.**

Now you can connect to the Windows Subsystem for Linux from Visual Studio by going to Tools > Options > Cross Platform > Connection Manager. Click add and enter “localhost” for the hostname and your WSL user/password.

WSL Ubuntu comes with installed *CMake* of version 3.5x, and considers it the newest, hence, no better installation can be done automatically; install a newer *CMake* manually:
```
sudo apt install cmake
cd /usr/local/src
sudo git clone https://github.com/Kitware/CMake.git
cd CMake
cd CMake
sudo git checkout tags/v3.9.0
sudo mkdir out
cd out
sudo cmake ../   
sudo make install
```
The above will build and install the release of CMake to `/usr/local/bin`. Verify the version is >= 3.8 and that server mode is enabled.
```
cmake --version
# cmake version 3.9.0

cmake -E capabilities
# {...
# ,"serverMode":true,
# ...}
```

# Clone EOS repository and install its dependencies 

Create a workspace location for EOS on the Windows file system. In this guide we will be using `X:\Workspaces\EOS` but obviously it's up to you to choose your own location.

On the Linux file system, the above location will be mapped as `/mnt/x/Workspaces/EOS`. The location you have chosen will be mapped accordingly. 

NOTE: use lower case for the name of the drive, in our case it's `/mnt/x/`.

All the following commands are to be run in the *Windows PowerShell* .

1. Define EOSIO_INSTALL_DIR system variable, and save it:
```
export EOSIO_INSTALL_DIR=/mnt/x/Workspaces/EOS/eos
echo "export EOSIO_INSTALL_DIR=${EOSIO_INSTALL_DIR}"  >> ~/.profile
```
NOTE: make sure to replace `x/Workspaces/EOS` with the appropriate path that matches the workspace location you have chosen on your computer.

4. Clone the source code from the EOS repository:
```
cd ${EOSIO_INSTALL_DIR}/../
git clone https://github.com/eosio/eos --recursive
```
5. Now you are ready to proceed with installation of EOS dependencies. This step can take several hours, depending on your computer's power, and will require you to intermittently confirm some actions and also to supply the `sudo` password. 
```
cd ${EOSIO_INSTALL_DIR} && \
. ${WORK_DIR}/scripts/install_dependencies.sh
```
NOTE: As this process requires downloading a lot of files from various sources, it might fail. In this case just try running this step again.

# Compile EOS

Start *Visual Studio*.

1. File > Open > Folder... > X:\Workspaces\EOS\eos.
2. Tools > Options > Cross Platform > Connection Manager. If there is not a host named `localhost`with its user name as in the bash prompt, click add and enter “localhost” for the hostname and your WSL user/password.
3. CMake > Change CMake Settings > CMakeLists.txt. It opens *CmakeSettings.json. There are blocks of definitions there: *x86-Debug*, ect. Add the following entry:
```
{
    "name": "Linux-Debug",
    "generator": "Unix Makefiles",
    "remoteMachineName": "localhost",
    "configurationType": "Debug",
    "remoteCMakeListsRoot": "/mnt/e/Workspaces/EOS/eos", ///// adjust it
    "cmakeExecutable": "/usr/local/bin/cmake",
    "buildRoot": "E:\\Workspaces\\EOS\\eos\\buildVS",  ///// adjust it
    "remoteBuildRoot": "/mnt/e/Workspaces/EOS/eos/buildVS", ///// adjust it
    "remoteCopySources": false,
    "remoteCopySourcesOutputVerbosity": "Normal",
    "remoteCopySourcesConcurrentCopies": "10",
    "cmakeCommandArgs": "-DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_COMPILER=/usr/bin/g++ -DWASM_LLVM_CONFIG=${HOME}/opt/wasm/bin/llvm-config -DBINARYEN_BIN=${HOME}/opt/binaryen/bin -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib -DBOOST_ROOT=${HOME}/opt/boost_1_64_0"
    "buildCommandArgs": "",
    "ctestCommandArgs": "",
    "inheritEnvironments": [ "linux-x64" ]
},

```
The *Visual Studio* did exactly the same what can be achieved with the *Windows PowerShell* (or with the *Windows Command Prompt*), having active linux `bash`:
```
cd ${EOSIO_INSTALL_DIR}
mkdir buildVS
cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_COMPILER=/usr/bin/g++ -DWASM_LLVM_CONFIG=${HOME}/opt/wasm/bin/llvm-config -DBINARYEN_BIN=${HOME}/opt/binaryen/bin -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib -DBOOST_ROOT=${HOME}/opt/boost_1_64_0 ../

make
```


# Run the executables

If no errors have occurred, you should have the EOS code on your Windows system compiled, resulting with multiple libraries and executables. Executables are placed in the `$EOSIO_INSTALL_DIR}/build/programs` folder:

- `eosd` - a server-side blockchain node component,
- `eosc` - a command line interface to interact with the blockchain,
- `eos-walletd` - an EOS wallet,
- `launcher` - an application for network composing and deployment, more information is available [here](https://github.com/EOSIO/eos/blob/master/testnet.md).

At this stage you should be able to run tests described in `eos/README.md`. 

For completeness, let's try if EOS can be started:
```
cd ${EOSIO_INSTALL_DIR}/build/programs/eosd && ./eosd
```
At this stage it should exit with an error complaining about the `genesis.json` file not being defined. In case it does not exit with this error, you can always close it with `ctrl + C`.

# Produce Blocks

Figure out the path for your `genesis.json` file. In our case it's `/mnt/x/Workspaces/EOS/eos/build/genesis.json`, yours will probably be different, depending on the workspace location you have chosen.

Edit the `config.ini` file (in our case it's located here: `X:\Workspaces\EOS\eos\build\programs\eosd\data-dir\config.ini`) using *WordPad* (or other text editor of your choice), locate the `enable-stale-production` entry and make it `enable-stale-production = true`.

Then append the following content in `config.ini`:
```
genesis-json = /mnt/x/Workspaces/EOS/eos/build/genesis.json

producer-name = inita
producer-name = initb
producer-name = initc
producer-name = initd
producer-name = inite
producer-name = initf
producer-name = initg
producer-name = inith
producer-name = initi
producer-name = initj
producer-name = initk
producer-name = initl
producer-name = initm
producer-name = initn
producer-name = initq
producer-name = initr
producer-name = inits
producer-name = initt
producer-name = initu

plugin = eosio::producer_plugin
plugin = eosio::chain_api_plugin
plugin = eosio::wallet_api_plugin
plugin = eosio::account_history_api_plugin
plugin = eosio::http_plugin 
```
NOTE: Make sure to set the proper value for the `genesis.json` path - most probably your path will be different than the one quoted above.

At this stage, when you run `eosd` again, it should start block production:
```
cd ${EOSIO_INSTALL_DIR}/build/programs/eosd && ./eosd
```

This is what you should see in your console if everything works OK:
```
232901ms thread-0   chain_plugin.cpp:80           plugin_initialize    ] initializing chain plugin
232902ms thread-0   producer_plugin.cpp:159       plugin_initialize    ] Public Key: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
232903ms thread-0   http_plugin.cpp:132           plugin_initialize    ] host: 127.0.0.1 port: 8888
232905ms thread-0   http_plugin.cpp:135           plugin_initialize    ] configured http to listen on 127.0.0.1:8888
...
232998ms thread-0   producer_plugin.cpp:170       plugin_startup       ] producer plugin:  plugin_startup() begin
232999ms thread-0   producer_plugin.cpp:175       plugin_startup       ] Launching block production for 19 producers.

*******************************
*                             *
*   ------ NEW CHAIN ------   *
*   -   Welcome to EOS!   -   *
*   -----------------------   *
*                             *
*******************************

Your genesis seems to have an old timestamp
Please consider using the --genesis-timestamp option to give your genesis a recent timestamp

233012ms thread-0   producer_plugin.cpp:185       plugin_startup       ] producer plugin:  plugin_startup() end
233012ms thread-0   http_plugin.cpp:147           plugin_startup       ] start processing http thread
...
233059ms thread-0   http_plugin.cpp:224           add_handler          ] add api url: /v1/account_history/get_transaction
233060ms thread-0   http_plugin.cpp:224           add_handler          ] add api url: /v1/account_history/get_transactions
237003ms thread-0   chain_controller.cpp:235      _push_block          ] initq #1 @2017-09-29T16:03:57  | 0 trx, 0 pending, exectime_ms=0
237005ms thread-0   producer_plugin.cpp:233       block_production_loo ] initq generated block #1 @ 2017-09-29T16:03:57 with 0 trxs  0 pending
240003ms thread-0   chain_controller.cpp:235      _push_block          ] initc #2 @2017-09-29T16:04:00  | 0 trx, 0 pending, exectime_ms=0
240004ms thread-0   producer_plugin.cpp:233       block_production_loo ] initc generated block #2 @ 2017-09-29T16:04:00 with 0 trxs  0 pending
243003ms thread-0   chain_controller.cpp:235      _push_block          ] initd #3 @2017-09-29T16:04:03  | 0 trx, 0 pending, exectime_ms=0
```
NOTE: It might happen that `eosd` hangs and fails to produce blocks at the first attempt. In this case just exit the process using `ctrl + C`, then wait a bit and try again.

# Update the Source Code

In order to update the source code from the official repository and recompile it, run the following commands:
```
cd ${EOSIO_INSTALL_DIR}
git pull
rm -r build && mkdir build && cd build
export BOOST_ROOT=${HOME}/opt/boost_1_64_0
cmake -DCMAKE_BUILD_TYPE=Debug \
    -DCMAKE_C_COMPILER=clang-4.0 \
    -DCMAKE_CXX_COMPILER=clang++-4.0 \
    -DWASM_LLVM_CONFIG=${HOME}/opt/wasm/bin/llvm-config \
    -DBINARYEN_BIN=${HOME}/opt/binaryen/bin \
    -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl \
    -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib \
    ../ && make
```
