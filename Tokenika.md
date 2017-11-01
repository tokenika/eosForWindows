# Installing EOS on a remote Linux computer

## Tooling up

### Enhanced terminal for Windows

Download [MobaXterm](#http://mobaxterm.mobatek.net/download.html). It is much easer than *PuTTY*: it can open GUI applications.

## Set up a workspace

1. Upgrade Ubuntu:
```bash
sudo apt update
sudo apt full-upgrade
sudo apt install nedit
```
2. Make a workspace for EOS. Let it be `~/Workspaces/EOS`, for example.

3. Make environment variables, defining the workspace:
```bash
WORKSPACE_DIR=~/Workspaces/EOS

export EOSIO_INSTALL_DIR=${WORKSPACE_DIR}/eos && \
echo "export EOSIO_INSTALL_DIR=${EOSIO_INSTALL_DIR}"  >> ~/.profile
```
4. Clean install Ubuntu

```bash
cd ${WORKSPACE_DIR} && git clone https://github.com/eosio/eos --recursive

cd ${EOSIO_INSTALL_DIR}
./build.sh ubuntu full
```

## All is ready now

Now, you have the EOS code in your *Windows 10* computer, compiled resulting with  libraries and executables. Executables are placed in `${EOSIO_INSTALL_DIR}/build/programs` folder:
* eosd - server-side blockchain node component
* eosc - command line interface to interact with the blockchain
* eos-walletd - EOS wallet
* launcher - application for nodes network composing and deployment; [more on launcher](https://github.com/EOSIO/eos/blob/master/testnet.md)

Now, you can do tests described in eos/README.md. For completeness, let us prove that eos can be started.

```bash 
cd ${EOSIO_INSTALL_DIR}/build/programs/eosd && ./eosd
```
If *eosd* does not exit with an error, close it immediately with <kbd>Ctrl-C</kbd>.

*EOS* uses a configuration file named `config.ini`, that needs a reference to a *genesis* file:

```bash
locate /build/genesis.json
    /home/jakub/Workspaces/EOS/eos/genesis.json
```

Open *nedit* GUI editor with the configuration file ...

```bash
nedit `locate build/programs/eosd/data-dir/config.ini`
```

... and edit it, appending the following text, setting the proper value for the `genesis-json` path, and commenting out the original `enable-stale-production` definition:

```
genesis-json = /home/jakub/Workspaces/EOS/eos/genesis.json
enable-stale-production = true

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

plugin = eos::producer_plugin
plugin = eos::chain_api_plugin
plugin = eos::wallet_api_plugin
plugin = eos::account_history_api_plugin
plugin = eos::http_plugin
```

## Update EOS, if needed

```bash
cd ${EOSIO_INSTALL_DIR}
git pull --recursive

rm -r build && mkdir build && cd build

cmake -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_C_COMPILER=clang-4.0 \
    -DCMAKE_CXX_COMPILER=clang++-4.0 \
    -DWASM_LLVM_CONFIG=${HOME}/opt/wasm/bin/llvm-config \
    -DBINARYEN_BIN=${HOME}/opt/binaryen/bin \
    -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl \
    -DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib \
    -DBOOST_ROOT=${HOME}/opt/boost_1_64_0 ../ 
make
```