# oobabooga macOS Apple Silicon Quick Start for the Impatient

Make sure Xcode at the minimum is installed.

If you are really in a rush and feeling brave, copy all of these lines into a text file and edit the uncomment line for version for the type of install you want. Uncomment the lines you wish to use and paste them in one at time into a terminal session of your choice. Use the script created as a template for your start script.

These instructions have been tested with a non-admin, plain user, so they should work for most everyone, but do let me know if something doesn't and I'll fix it. Typos and copy and paste sometimes have a away for going wrong.

## DO NOT JUST COPY AND PASTE UNLESS YOU HAVE READ AND UNDERSTAND THE INSTRUCTIONS - YOU MAY NEED TO CHANGE THEM FOR YOUR SYSTEM

## 15 Sep 2024 - updated instructions, you may need to update your CMake, I did.

This has ben updated with a few new items, like CMake, installing in the user's home directory.

```bash
#!/bin/bash
## These instructions assume you are using the Bash shell. I also sugget getting a copy
## of iTerm2, it will make your life better, iut is much better than the default terminal
## on macOS.
## 
## If you are using zsh, do this first, do it even if you are running bash,
## it will not hurt anything.

## This will give you a login shell with bash.
exec bash -l

cd "${HOME}"

umask 022

### Choose a target directory for everything to be put into, I'm using "${HOME}/projects/ai-projects" You
### may use whatever you wish. This must be exported because we will exec a new login shell later.
export TARGET_DIR="${HOME}/projects/ai-projects"

mkdir -p "${TARGET_DIR}"
cd "${TARGET_DIR}"

# This will add to your path and DYLD_LIBRARY_PATH if they aren't already seyt up.
# export PATH=${HOME}/local/bin
# export DYLD_LIBRARY_PATH=${HOME}/local/lib:$DYLD_LIBRARY_PATH

### Be sure to add ${HOME}/local/bin to your path  **Add to your .profile, .bashrc, etc...**
export PATH=${HOME}/local/bin:${PATH}

### Thwe following Sed line will add it permanantly to your .bashrc if it's not already there.
sed -i.bak '
  /export PATH=/ {
    h; s|$|:${HOME}/local/bin|
  }
  ${
    x; /./ { x; q0 }
    x; s|.*|export PATH=${HOME}/local/bin:\$PATH|; h
  }
  /export DYLD_LIBRARY_PATH=/ {
    h; s|$|:${HOME}/local/lib|
  }
  ${
    x; /./ { x; q0 }
    x; s|.*|export DYLD_LIBRARY_PATH=${HOME}/local/lib:\$SYLD_LIBRARY_PATH|; h
  }
' ~/.bashrc && source ~/.bashrc

## Install Miniconda

#### We will set this heer and it will be used later when we source .bashrc later.
echo 'export MACOS_LLAMA_ENV="macOS-llama-env"' >> ~/.bashrc

### Download the miniconda installer
curl  https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh

### Run the installer in non-destructive mode in order to preserve any existing installation.
sh miniconda.sh -b -u
. "${HOME}/miniconda3/bin/activate"

conda init $(basename "${SHELL}")
conda update -n base -c defaults conda -y
 
#### Get a new login shell no that conda is activated to your shell profile.
exec bash -l

umask 022

#### Just in case your startup login environment scripts do some thing like change to another directory.
#### Get back into teh target directory for teh build.
cd "${TARGET_DIR}"

#### Set the name of the VENV to whatever you wish it to be. This will be used later when the procedure
#### creates a script for sourcing in the Conda environment and activating the one set here when you installed.
#### Create the base Python 3.10 and the llama-env VENV.
conda create -n ${MACOS_LLAMA_ENV} python=3.10 -y
conda activate ${MACOS_LLAMA_ENV}

## Build and install CMake

### Clone the CMake repository, build, and install CMake
git clone https://github.com/Kitware/CMake.git
cd CMake
git checkout tags/v3.30.2
mkdir build
cd build

### This will configure the installation of cmake to be in your home directory under local, rather than /usr/local
../bootstrap --prefix=${HOME}/local
make -j
make -j test
make install

### Verify the installation
which cmake       # Should say $HOME/local/bin
### Verify you are running cmake z3.29.3
cmake --version
cd  "${TARGET_DIR}"


## Get my oobabooga and checkout macOS-test branch
git clone https://github.com/unixwzrd/text-generation-webui-macos.git textgen-macOS
cd textgen-macOS
git checkout macOS-dev
pip install -r requirements.txt

## llamacpp-python
CMAKE_ARGS="-DLLAMA_METAL=on" \
FORCE_CMAKE=1 \
PATH=/usr/local/bin:$PATH \
pip install llama-cpp-python --force-reinstall --no-cache --no-binary :all: --compile --no-deps --no-build-isolation

## Pip install from daily build
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu --no-deps --force-reinstall

## NumPy Rebuild with Pip
CFLAGS="-I/System/Library/Frameworks/vecLib.framework/Headers -Wl,-framework -Wl,Accelerate -framework Accelerate" \
pip install numpy==1.26.* --force-reinstall --no-deps --no-cache --no-binary :all: --no-build-isolation --compile -Csetup-args=-Dblas=accelerate -Csetup-args=-Dlapack=accelerate -Csetup-args=-Duse-ilp64=true

## CTransformers
export CFLAGS="-I/System/Library/Frameworks/vecLib.framework/Headers -Wl,-framework -Wl,Accelerate -framework Accelerate"
export CT_METAL=1
pip install ctransformers --no-binary :all: --no-deps --no-build-isolation --compile --force-reinstall

### Unset all the stuff we set while building.
unset CMAKE_ARGS FORCE_CMAKE CFLAGS CT_METAL


## This will create a startup script whcih shoudl be clickable in finder.

### Set the startup options you wish to use

# Add any startup options you wich to this here:
START_OPTIONS=
#START_OPTIONS="--verbose "
#START_OPTIONS="--verbose --listen"

cat <<_EOT_ > start-webui.sh
#!/bin/bash

# >>> conda initialize >>>
__conda_setup="$('${HOME}/miniconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
  eval "$__conda_setup"
else
  if [ -f "${HOME}/miniconda3/etc/profile.d/conda.sh" ]; then
    . "${HOME}/miniconda3/etc/profile.d/conda.sh"
  else
    export PATH="${HOME}/miniconda3/bin:$PATH"
  fi
fi
unset __conda_setup
# <<< conda initialize <<<

cd "${TARGET_DIR}/textgen-macOS"

conda activate ${MACOS_LLAMA_ENV}

python server.py ${START_OPTIONS}
_EOT_


chmod +x start-webui.sh
```

## Starting the Web UI

### This will create the file in the current directory which is displayed here feel free to move it

````bash
${PWD}
```

### Feel free to move it to another location.

```bash
./start-webui.sh
```
