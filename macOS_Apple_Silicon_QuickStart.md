# oobabooga macOS Apple Silicon Quick Start for the Impatient

Make sure Xcode at the minimum is installed.

If you are really in a rush and feeling brave, copy all of these lines into a text file and edit the uncomment line for version for the type of install you want. Uncomment the lines you wish to use and paste them in one at time into a terminal session of your choice. Use the script created as a template for your start script.

**DO NOT JUST COPY AND PASTE UNLESS YOU HAVE READ AND UNDERSTAND THE INSTRUCTIONS**

## 17 Sep 2023 - lots changed in the past 24 hours or so.  NumPy no longer builds on Metal/ MPS like it did in these instructions, and there have been issues with llama-cpp-python, I've bumped the version up to 0.2.6 which is the latest working version. I need to validate and verify everything.

Latest != Greatest, Latest + Greatest != Best, Stable == None

```bash
#!/bin/bash

# Install Miniconda MAke sure it is arm64
mkdir tmp
cd tmp
curl  https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh

# Do a non-destructive Conda install whcih will preserve existing VENV's
sh miniconda.sh -b -u
cd ..

# Activate the conda environment.
. ${HOME}/miniconda3/bin/activate

# Initialize Conda which will add initialization functions to your shell's profile.
conda init $(basename ${SHELL})

# Update your Conda environment with the latest updates to th ebase environment
conda update -n base -c defaults conda -y

# Grab a new login shell - this will work for any shell aand you wil enter back in the
# tmp directory we just created..
exec $( basename ${SHELL}) -l

# Update Conda if necessary
conda update -n base -c defaults conda

# Create a new Conda environment with Python 3.10
conda create -n python3.10 python=3.10

# Activate the new environment
conda activate python3.10

### Clone and build CMake from source
### NOTE: This will install in your /usr/local directory tree.
git clone https://github.com/Kitware/CMake.git
cd cmake
./bootstrap
make -j24
make install
cd ..

### Clone, build, and install OpenBLAS **OPENBLAS THROUGH NUMPY WILL CHANGE SOON, NEW INFOiRMATION**
### NOTE: This will install in your /usr/local directory tree.
### UN-COMMENT the next six lines to build th eoptional OpenBLAS
# git clone https://github.com/xianyi/OpenBLAS
# mkdir -p OpenBLAS/build
# cd OpenBLAS/build
# cmake .
# make -j24
# make install
# cw ../..

# Make a checkpoint VENV for rollback
onda activate python3.10
conda create --clone python3.10 -n webui.00.base

# It may already be installed from a previous step, but we will need PyTorch re-installed.
# PyTorch, torchvision, and torchaudio from the PyTorch Conda channel
conda create --clone webui.00.base -n webui.01.torch
conda activate webui.02.torch-oldllama
conda install pytorch torchvision torchaudio -c pytorch


# Create a new VENV for rollback before installing th erequirements fo rthe webui.
conda create --clone webui.01.pytorch -n webui.02.oobabase
conda deactivate
conda activate webui.02.oobabase 

### Get the macOS repo or see the commented lines and use them to get the development or original.
git clone https://github.com/unixwzrd/text-generation-webui-macos.git webui-macOS
cd webui-macOS
pip install -r requirements.txt
cd ..

# Get the TEST version
# conda create --clone python3.10 -n webui.00.ooba-macOS-dev
# conda activate webui.00.ooba-macOS-dev
### macOS development
# git clone -b test --single-branch https://github.com/unixwzrd/text-generation-webui-macos.git webui-macOS-dev
# cd webui-macOS
# pip install -r requirements.txt
# cd ..

# Get the DEV version
# conda create --clone python3.10 -n webui.00.ooba-macOS-dev
# conda activate webui.00.ooba-macOS-dev
### macOS development
# git clone -b dev --single-branch https://github.com/unixwzrd/text-generation-webui-macos.git webui-macOS-dev
# cd webui-macOS
# pip install -r requirements.txt
# cd ..

### UN-COMMENT THE NEXT FOUR LINES IF YOU WANT TO USE THE ORIGINAL OOBABOOGA
# conda create --clone python3.10 -n webui.00.oobabase
# conda activate webui.00.oobabase
### oobabooga original
# git clone https://github.com/oobabooga/text-generation-webui.git webui
# cd webui
# pip install -r requirements.txt
# cd ..

# Install the Python modules listed in oobabooga's requirements.txt file

### NOTE: if you aren't converting your GGML files over to th enew GGUF
### format, use version 0.1.78 of the llama-cpp-python.
#
conda create --clone webui.02.oobabase -n webui.03.oldllama
conda activate webui.03.oldllama
# If it's been installed by oobaboogs, do this for 0.1.78.  
NPY_BLAS_ORDER='accelerate' NPY_LAPACK_ORDER='accelerate' \
    CMAKE_ARGS='-DLLAMA_METAL=on' FORCE_CMAKE=1 \
    pip install --force-reinstall --no-cache --no-binary :all: --compile llama-cpp-python==0.1.78

### NOTE: If you have or weill be converting your GGML files to GGUF format, use this.
###
### This will get the latest which will no longer work with GGML files until you convert them.
conda create --clone webui.02.oobabase -n webui.03.llama-new
conda activate webui.03.newllama
NPY_BLAS_ORDER='accelerate' NPY_LAPACK_ORDER='accelerate' \
    CMAKE_ARGS='-DLLAMA_METAL=on' FORCE_CMAKE=1 \
    pip install --force-reinstall --no-cache --no-binary :all: --compile llama-cpp-python==0.2.6


# As a last step, you may want to clone this VENV in order to protect it from having
# packages accidently dropped into it.
#
# If you used the old LLAMA-CPP-PYTHON, user this line
conda craete --clone webui.03.torch-oldllama -n webui.04.final-ggnml

# If you used the NEW LLAMA-CPP-PYTHON, user this line
conda craete --clone webui.03.torch-newllama -n webui.04.final-gguf

# Pick whcih one of these you wish to make your preferred VENV.
# PREFERRED_VENV=webui.03.final-ggml
PREFERRED_VENV=webui.04.final-gguf

# Add any startup options you wich to this here:
START_OPTIONS="--chat"
#START_OPTIONS="--chat --verbose "
#START_OPTIONS="--chat --verbose --listen"

# This assumes you followed the instructions abobe for installing teh Conda or upgrading
# the Conda oackage manager in your homme directory.  If you changed it to someting else,
# you'll need to make appropriate changes here.
cat <<_EOT_
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

conda activate ${PREFERRED_VENV}

python server.py ${START_OPTIONS}
_EOT_ > start-webui.sh
chmod +x start-webui.sh

# Cleanup any unneeded VENV's once we are happy with th ebuild and everytihng is running
# smoothly
conda uninstall --all -n webui.00.ooba-macOS
conda uninstall --all -n webui.01.ooba-llama-0.1.78
conda uninstall --all -n webui.01.ooba-llama-new
conda uninstall --all -n webui.02.ooba-torch-oldllama
conda uninstall --all -n webui.02.ooba-torch-newllama
```

Add the --upgrade flag to upgrade any of the pip or conda commands if necessary.

Remember to create clones of your Conda environments at various stages to easily roll back to a previous state if something goes wrong.
