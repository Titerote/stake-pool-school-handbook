# Installing Cardano-node

## Latest version 1.25.1

## Prerequisites

Set up your platform:

You will need:

* An x86 host \(AMD or Intel\), Virtual Machine or AWS instance with at least 2 cores, 4GB of RAM and at least 10GB of free disk space;
* A recent version of Linux, not Windows or MacOS – this will help us isolate any issues that arise.
* Make sure you are on a network that is not firewalled. In particular, we will be using TCP/IP port 3000 and 3001 by default to establish connections with other nodes, so this will need to be open.

## Install dependencies

We need the following packages and tools on our Linux system to download the source code and build it:

* the version control system `git`,
* the `gcc` C-compiler,
* C++ support for `gcc`,
* developer libraries for the arbitrary precision library `gmp`,
* developer libraries for the compression library `zlib`,
* developer libraries for `systemd`,
* developer libraries for `ncurses`,
* `ncurses` compatibility libraries,
* the Haskell build tool `cabal`,
* the GHC Haskell compiler.

If we are using an AWS instance running Amazon Linux AMI 2 \(see the [AWS walk-through](https://github.com/carloslodelar/SPO/tree/baec64ba9efba39d4b60b7824fb4d7b962f2c3e7/getting-started/000_AWS.md) for how to get such an instance up and running\)or another CentOS/RHEL based system, we can install these dependencies as follows:

```text
sudo yum update -y
sudo yum install git gcc gcc-c++ tmux gmp-devel make tar wget -y
sudo yum install zlib-devel libtool autoconf -y
sudo yum install systemd-devel ncurses-devel ncurses-compat-libs -y
```

For Debian/Ubuntu use the following instead:

```text
sudo apt-get update -y
sudo apt-get install build-essential pkg-config libffi-dev libgmp-dev -y
sudo apt-get install libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev -y
sudo apt-get install make g++ tmux git jq wget libncursesw5 libtool autoconf -y
```

If you are using a different flavor of Linux, you will need to use the package manager suitable for your platform instead of `yum` or `apt-get`, and the names of the packages you need to install might differ.

## Download, unpack, install and update Cabal:

```text
wget https://downloads.haskell.org/~cabal/cabal-install-3.2.0.0/cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
tar -xf cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz
rm cabal-install-3.2.0.0-x86_64-unknown-linux.tar.xz cabal.sig
mkdir -p ~/.local/bin
mv cabal ~/.local/bin/
```

Verify that .local/bin is in your PATH

```text
echo $PATH
```

If .local/bin is not in the PATH, you need to add the following line to your `.bashrc`file

Navigate to your home folder:

```text
cd
```

Open your .bashrc file with nano text editor

```text
nano .bashrc
```

Go to the bottom of the file and add the following lines

```text
export PATH="~/.local/bin:$PATH"
```

You need to restart your server or source your .bashrc file

```text
source .bashrc
```

Update cabal

```text
cabal update
```

Above instructions install Cabal version `3.2.0.0`. You can check the version by typing

```text
cabal --version
```

## Download and install GHC:

For Debian/Ubuntu systems:
```text
wget https://downloads.haskell.org/~ghc/8.10.2/ghc-8.10.2-x86_64-deb9-linux.tar.xz
tar -xf ghc-8.10.2-x86_64-deb9-linux.tar.xz
rm ghc-8.10.2-x86_64-deb9-linux.tar.xz
cd ghc-8.10.2
./configure
sudo make install
cd ..
```
For CentOS/RHEL systems:
```text
wget https://downloads.haskell.org/~ghc/8.10.2/ghc-8.10.2-x86_64-centos7-linux.tar.xz
tar -xf ghc-8.10.2-x86_64-centos7-linux.tar.xz
rm ghc-8.10.2-x86_64-centos7-linux.tar.xz
cd ghc-8.10.2
./configure
sudo make install
cd ..
```
Alternatively, the ghcup tool can be used to install and set several versions of GHC:

```text
curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh
ghcup upgrade
ghcup install <VERSION>
ghcup set <VERSION>
```
<VERSION> here could be for example 8.10.2

You can check that your default GHC version has been properly set:

```text
ghc --version
```

## Install Libsodium

```text
export LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
export PKG_CONFIG_PATH="/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH"

git clone https://github.com/input-output-hk/libsodium
cd libsodium
git checkout 66f017f1
./autogen.sh
./configure
make
sudo make install

```

## Download the source code for cardano-node

```text
cd
git clone https://github.com/input-output-hk/cardano-node.git
```

This creates the folder `cardano-node` and downloads the latest source code.

After the download has finished, we can check its content by

```text
ls cardano-node
```

We change our working directory to the downloaded source code folder:

```text
cd cardano-node
```

For reproducible builds, we should check out a specific release, a specific "tag". For the Mary Testnet, we will use tag `1.25.1`, which we can check out as follows:

```text
git fetch --all --tags
git tag
git checkout tags/1.25.1
```

## Build and install the node

Now we build and install the node with `cabal`, which will take a few minutes the first time you do a build. Later builds will be much faster, because everything that does not change will be cached.

```text
cabal clean
cabal update
cabal build all
```

Now we can copy the executables files to the .local/bin directory

```text
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-node-1.25.1/x/cardano-node/build/cardano-node/cardano-node ~/.local/bin/
```

```text
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-cli-1.25.1/x/cardano-cli/build/cardano-cli/cardano-cli ~/.local/bin/
```

```text
cardano-cli --version
```

{% hint style="info" %}
If you need to update to a newer version follow the steps below:
{% endhint %}
```text
cd cardano-node
git fetch --all --tags
git tag
git checkout tags/<the-tag-you-want>
cabal update
cabal build cardano-node cardano-cli
```
_Note:_ This is a good time to backup your current binaries (in case you have to revert to an earlier version). Something like this will work:
```text
cd ~/.local/bin
mv cardano-cli cardano-cli-backup
mv cardano-node cardano-node-backup
```
Now copy your newly built binaries to the appropriate directory, with:
```text
cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-node-<NEW VERSION>/x/cardano-node/build/cardano-node/cardano-node ~/.local/bin/

cp -p dist-newstyle/build/x86_64-linux/ghc-8.10.2/cardano-cli-<NEW VERSION>/x/cardano-cli/build/cardano-cli/cardano-cli ~/.local/bin/
```

_Note:_ It might be necessary to delete the `db`-folder \(the database-folder\) before running an updated version of the node.



{% hint style="info" %}
QUESTIONS AND FEEDBACK

If you have any questions and suggestions while taking the lessons please feel free to ask in the [forum](https://forum.cardano.org/c/staking-delegation/setup-a-stake-pool/158) and we will respond as soon as possible.
{% endhint %}
