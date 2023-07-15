# Building for macOS and Apple Silicon

You will still want and probably need the pre-requisites anyway.  This document is a work in progress. If you see anything which is incorrect, could be made clearer, os needs updating, please let me know.

I know a lot of poeople are going to say, why not just get Brew and go from there, well you could, but I am very old-school and have been building Open Source before there weer package managers. Package managers are a blessing and a curse. I also had a bad expperience with Brew and the other package manager which managed Open Source and other source distributed installations.

* Advantages of Package Managers

  * Package managers are great becayse they deal with dependencies
  * Package managers can help keep you up to date
  * Package managers everything is pre-configured
  * Package managers automatically give you updates

* Disadvantages of Package Managers

  * Package managers are great becayse they deal with dependencies
  * Package managers can help keep you up to date
  * Package managers everything is pre-configured
  * Package managers automatically give you updates

## Pre-requisites

You're going to need a few things before you start

1. iTerm2

      Make this the first thing you get before you do anything else.

      If you spend any time on the command line, you must get this, unless you just love Terminal.app. There are far too many configuration options to go into here, but play around with it. PROTIP: Set it up for tabbed windows.

1. XCode

      You will need a compiler. It would be nice if GCC ran one macOS without going through lots of gyrations, but Xcode works well enough.

      Go tio the App Store and get a copy of Xcode You're going to need a few things before you just begin.

      If you **ONLY** want the command line tools and not the complete Xcode IDE, you will want just the command line tools for Xcode by running the xcode-select. Open up iTerm2 you downloaded and installed earlier or Terminal.app in /AApplications/Utilities and get the command line tools for XCode like this:

      ```bash
      xcode-select --install
      ```

1. VSCode

      Yeah, I know two IDE's but theer are lots of plugins for VSCode and unless you are developing macOS or other Apple apps, this is a great IDE. I like it because there is a Vi/Vim mode for it so, if you know the keystrokes for Vi, you're good to go. An integrated terminal means you can run shell scripts while in the IDE. There are millions of options, settings, and plugins for this and finding the right ones may be difficult, but if youare doing any work with AI, then chances are you will want this fo rthe Jupyter Notebooks support alone. It also connects very seamlesssly GitHub.

## Get Conda (Miniconda)

A word of caution during all this, some libraries are critical you get the proper compiled version rather than the version whcih just comes down with pip or conda. Pay attention to this as some extensions for oobabooga habe uninstalled perfectly fine versiond of my libraries and downgraded them due to dependecies.  If this happens you could lose performance and be hunting down what happened. This has happened to me for NumPy and llama-cpp.  My intenet here is to try to pay attention to the libraries as much as possible going into the construction of the environment for running and managing LLMs using oobabooga. I'm trying to catch as many gotcha's as possible.

One potentiam method os avoiding conflicts, downgrades and other headaches is to use the "--dry-run" argument which will just tell you what it's planning to do, without actually doing it. The output can be long ad you can miss things. As an added measure I clone my virtual environments (venv), then switch to th enew one before I do anything which could potentially be fatal to my current venv.

```bash
   cd
   mkdir tmp
   cd tmp
   curl -sL https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOS-amd64.sh > "Miniconda3.sh"
   sh Miniconda.sh
```

After installing the miniconda and conda, you should update conda
using th e following:

```bash
conda update -n base -c defaults conda
exec bash -l
```

* Creatwe a new venv using Python 3.10.

```bash
conda create -n textgen0 python==3.10
exec bash -l
conda activate textgen0
```

-- This gives us a clean environment to returbn to as a base. I tend to clone
my conda venv's so it's easy to roll back soething evil whicih has taken a
wrecking ball to a small part of my environment.  It saves me time to be able
to roll back to a prior known environment and move forward again. These
venv's can be quite nice for rolling back to a known configuration and I
recommend my technique of cloning your good venv then activating it and
applying any changes to that. Many packages or updates hit multiple python
modules at one time and this is an easy way to roll back and then move forward
creating a new venv cloned from the previous then new items are instaleld into
that venv. When it is working, then clone that one and activate it, then do
the next round of updates or changes.  At any point, venv's may be removed
completely and even renamed.  So, you can take your final venv, if you are
happy with it, and rename it back to th ebase for your application. I will try
to do this as I go along in this installation taking venv chackpoints which I
can roll back the previous.

Cloning a venv can also help you quickly determine if a compile, or soe other
module gives any performance advantage, but I can explain soe mof these
techniques at another time.

Make sure you ahve CMake installed.  You can find it here:
            <https://cmake.org/download/>

It's pretty easy to install CMake and you will need it for later steps like
   llama-cpp, llama-cpp-python and other modules..

Download the lastest source bversion of CMake.  You do not want to use the
packages as they are universal binaries and you might accidently end up
building something with x86_64 architecture.  I haven't tried this to verify,
but why take a chance?

The steps are pretty simple:

```bash
# untar the tar file, it will create a cmake_source directory.
# cd cmake_source
./bootstrap
# I do this on the 12 core M2 Max and it flys.
make -j24
make test
make install
```

This creates 24 compile jobs.  I havbe 12 cores on my MBP, so I use 2
time cores, this works rather well and builds quickly. Make should
parallelize as much as it can based on dependencies.

## OpenBLAS

Building OpenBLAS is optional, but it can be built for incorporation into your
own build of NumPy. If you wsant to get down into th enuts and bolts, then go
ahead and build your own NumPy, it's actually quite easy, but installing using
conda will being down a number of other lobraries and Python modules, so
you're probably best using conda to do this.  I will show how to build
OponBLAS and NumPy using your library for OpenBLAS.

OpenBLAS i(BLAS) and LAPACK - Basic Linear Algebra and Linear Algebra Pack
These were originally built in FORTRAN, and you may find them on netlib.org.
I didn't have  the GNU Fortran compiler available and the BNU compoiler
collection has been a problem on macOS for quite a while since they changed
their binary formats and use LLVM.  You muct use their C compiler, but it
works pretty well.  Make sure you have Xcode and teh Xcode command line
utilities loaded. (Actually before you begin any of this.)

Download the openblas repo and build it, then install.

```bash
git clone <https://github.com/xianyi/OpenBLAS>
cd OpenBLAS
cmake .
make
make test
make install
```

## Now for NumPy

first thing I do is create a clone of my baseline to this point, just in case
something goes wrong, like other modules being updated or for some reason we
want to roll this back. This one is fairly easy, since pip only installs one
module for NumPy, however using conda, will install several modules,
presumably as dependencies though pip doesn't. One thing which is included
seperately in the conda install is the OpenBLAS libraries.

```bash
conda create --clone textgen0 -n textgen1
conda deactivate
conda activate textgen1
pip install --no-cache --no-binary :all: --compile numpy
```

You can check to see if this worked by thr following:

```bash
python
import numpy
numpy.show_config()
```

```
(textgen1) [unixwzrd@thorazine projects]$ python
Python 3.10.0 (default, Mar  3 2022, 03:54:28) [Clang 12.0.0 ] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import numpy
>>> numpy.show_config()
blas_armpl_info:
  NOT AVAILABLE
blas_mkl_info:
  NOT AVAILABLE
blas_ssl2_info:
  NOT AVAILABLE
blis_info:
  NOT AVAILABLE
openblas_info:
    libraries = ['openblas', 'openblas']
    library_dirs = ['/usr/local/lib']
    language = c
    define_macros = [('HAVE_CBLAS', None)]
blas_opt_info:
    libraries = ['openblas', 'openblas']
    library_dirs = ['/usr/local/lib']
    language = c
    define_macros = [('HAVE_CBLAS', None)]
lapack_armpl_info:
  NOT AVAILABLE
lapack_mkl_info:
  NOT AVAILABLE
lapack_ssl2_info:
  NOT AVAILABLE
openblas_lapack_info:
    libraries = ['openblas', 'openblas']
    library_dirs = ['/usr/local/lib']
    language = c
    define_macros = [('HAVE_CBLAS', None)]
lapack_opt_info:
    libraries = ['openblas', 'openblas']
    library_dirs = ['/usr/local/lib']
    language = c
    define_macros = [('HAVE_CBLAS', None)]
Supported SIMD extensions in this NumPy install:
    baseline = NEON,NEON_FP16,NEON_VFPV4,ASIMD
    found = ASIMDHP,ASIMDDP
    not found = ASIMDFHM
>>>
```

Clone that textgen0 and put the numerical items necessary conda install of
the NumPy and PyTorch modules and their dependencies.  The big difference
between pip and conda is that OpenBLAS will be pout in your conda venv,
whereas the compilation and installation of OpenBLAS and the forced recopile
of NumPy by pip will use the library in /usr/local/lib instead of your
conda/miniconda repository.

conda create --clone textgen0 -n textgen1

conda install numpy

-- PyTorch and Friends

First create a checkpoint before either installing using pip or conda.

conda create --clone textgen1 -n textgen2

Let's now install PyTorch or Torch and see what happens by using either pip,
conda or the instructions on the PyTorch site.  PyTorch says there are two
ways to install PyTorch/Torch.  One using pip and the other with conda. They
are slightly different builds.

Method 1 is with pip.

pip install torch torchvision torchaudio

Method 2 is woith conda.

conda install pytorch torchvision torchaudio -c pytorch

These may have to be done another time as well.

Now those are complete, let's make another checkpoint and istall everything
else.

conda create ==clone textgen2 -n textgen3

Change into th eoobabooga text-generation-webui directory and load the
requirements. Due to th estructure of the file, it must be done with pip.

pip install -r requirements.txt

Now, at this point, we have everything we need to run the basid server with no
extensions.  However, we should have a look at the llama-cpp and
llama-cpp-python as we may need to build them ourselves.

torch doesn't seem to be loaded correctly

# Llama for macOS and MPS

pip uninstall -y llama-cpp-python
CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install llama-cpp-python --no-cache-dir

# PyTorch for macOS and MPS

This seems to be the best method rathetr than using pip to install, th ecollection seems more up to date and more comprehensive than PyPi.

''
conda install pytorch torchvision torchaudio -c pytorch
''

# Misc ToDo's

* Build Linux/macOS Installer for ARM64 and x86_64
  * Keep Windows batch, WSL separated from these. Windows file system and *nix don't play nice together very well.
  * This is a work in process.
    * Do it all in shell scripts with sne envirnment variables
    * .env
    * .env.local
    * .env.default

* Keep existing or download new Minicondo
  * Prevent a new installation of Miniconda from burining down existing Miniconda installs.

* New venv on update
  * Copy repo to backup or tar
  * Clone repo
  * Create new venv for backup

* support for building in and retreiving extensions

* Voice alterations and config to keep ElevenLabs voice
  * Save voice name and ID
  * Save Voice parameters
  * Clear cache and mp3 files somehow

* Increase Context size to 16k or make it a parameter
  * Fix context truncation calculkation
  * Generate tokens in advance and store them along side history
  * Add token_count and
