# oobabooga macOS Apple Silicon Quick Start for the Impatient

Make sure Xcode at the minimum is installed.

```bash
# Install Miniconda MAke sure it is arm64
curl  https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh
sh miniconda.sh
# After sucessful install of Conda you may remove miniconda.sh
rm miniconda.sh

# Get a new login Shell
exec bash -l

# Update Conda if necessary
conda update -n base -c defaults conda

# Create a new Conda environment with Python 3.10
conda create -n python3.10 python=3.10

# Activate the new environment
conda activate python3.10

# Clone and build CMake from source
git clone https://github.com/Kitware/CMake.git
cd cmake
./bootstrap
make -j24
make install
cd ..

# Clone, build, and install OpenBLAS **OPENBLAS THROUGH NUMPY WILL CHANGE SOON, NEW INFOiRMATION**
git clone https://github.com/xianyi/OpenBLAS
cd OpenBLAS
cmake .
make -j24
make install


git clone https://github.com/unixwzrd/text-generation-webui-macos.git

# git clone https://github.com/oobabooga/text-generation-webui.git

# Install the Python modules listed in oobabooga's requirements.txt file
pip install -r requirements.txt

# Uninstall any existing version of llama-cpp-python
pip uninstall -y llama-cpp-python

### NOTE: if you aren't converting your GGML files over to th enew GGUF
### format, use version 0.1.78 of the llama-cpp-python.
#
# If it's been installed bt oobaboogs, do this for 0.1.78.  
 NPY_BLAS_ORDER='accelerate' NPY_LAPACK_ORDER='accelerate' \
   CMAKE_ARGS='-DLLAMA_METAL=on' FORCE_CMAKE=1 \
   pip install --force-reinstall --no-cache --no-binary :all: --compile llama-cpp-python==0.1.78

### NOTE: If you have or weill be converting your GGML files to GGUF format, use this.
#
 # This will get the latest which qill no longer work with GGML files until you convert them.
 NPY_BLAS_ORDER='accelerate' NPY_LAPACK_ORDER='accelerate' \
   CMAKE_ARGS='-DLLAMA_METAL=on' FORCE_CMAKE=1 \
   pip install --force-reinstall --no-cache --no-binary :all: --compile llama-cpp-python


# It may already be installed from a previous step, but we will need PyTorch re-installed.
# PyTorch, torchvision, and torchaudio from the PyTorch Conda channel
conda install pytorch torchvision torchaudio -c pytorch
```

Add the --upgrade flag to upgrade any of the pip or conda commands if necessary.

Remember to create clones of your Conda environments at various stages to easily roll back to a previous state if something goes wrong.
