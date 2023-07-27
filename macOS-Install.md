# Apple Silicon Support for oobabooga text-generation-webui

This guide provides instructions on how to build and run the oobabooga text-generation-webui on macOS, specifically on Apple Silicon.

While this is primarily for oobabooga users at the moment, many of the Python ibraries and packages used heer may also be used for Data Analytics, Machine Learning and other purposes.

I will likely turn this into information on acceleratuon using the Apple Silicon GPU.

## TL;DR

1. **Python**: Install Python 3.10 using Miniconda. Create a virtual environment and install pip.
1. **CMake**: Install CMake from source to avoid potential issues with universal binaries. This is used for building other software.
1. **OpenBLAS**: Build and install OpenBLAS, a library for linear algebra operations. This can optionally be used when building NumPy.
1. **NumPy**: Install NumPy in two ways:
   - Build it from source, ensuring it uses the OpenBLAS library you built.
   - Install it quickly using Conda or Pip.
1. **PyTorch**: Install PyTorch using either Conda or Pip. PyTorch is a key package for machine learning applications.
1. **oobabooga Base**: Clone the oobabooga GitHub repository and install the Python modules listed in its requirements.txt file.
1. **Llama for macOS and MPS**: Uninstall any existing version of llama-cpp-python, then reinstall it with specific CMake arguments to enable Metal support.
1. **PyTorch for macOS and MPS**: Install PyTorch, torchvision, and torchaudio from the PyTorch Conda channel.

Cheeck out [oobabooga macOS Apple Silicon Quick Start for the Impatient](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS_Apple_Silicon_QuickStart.md) for the short method without explanations.

Throughout the process, you're advised to create clones of your Conda environments at various stages. This allows you to easily roll back to a previous state if something goes wrong.

Please note that the guide is incomplete and is expected to be continued.

## Table of Contents

- [Apple Silicon Support for oobabooga text-generation-webui](#apple-silicon-support-for-oobabooga-text-generation-webui)
  - [TL;DR](#tldr)
  - [Table of Contents](#table-of-contents)
  - [Building for macOS and Apple Silicon](#building-for-macos-and-apple-silicon)
  - [Pre-requisites](#pre-requisites)
  - [Get Conda (Miniconda)](#get-conda-miniconda)
  - [CMake](#cmake)
  - [NumPy - Everything or Quickly](#numpy---everything-or-quickly)
    - [NymPy Build Everything - OpenBLAS](#nympy-build-everything---openblas)
    - [NumPy](#numpy)
    - [NumPy Quicker - Use Conda or Pip](#numpy-quicker---use-conda-or-pip)
  - [PyTorch](#pytorch)
  - [oobabooga Base - Everything Else](#oobabooga-base---everything-else)
    - [Clone The oobabooga GitHub Repository](#clone-the-oobabooga-github-repository)
    - [Install oobabooga Requirements](#install-oobabooga-requirements)
  - [Llama for macOS and MPS](#llama-for-macos-and-mps)
    - [Buiklding llama-cpp-pythin from source](#buiklding-llama-cpp-pythin-from-source)
  - [Pandas](#pandas)
  - [PyTorch for macOS and MPS](#pytorch-for-macos-and-mps)
  - [Where We Are](#where-we-are)
  - [Extensions](#extensions)
  - [NOTE THIS IS INCOMPLETE- To be continued](#note-this-is-incomplete--to-be-continued)

This guide is quite comprehensive and covers everything from getting the necessary prerequisites to building and installing all the required components. It also includes a section on how to clone and install the oobabooga repository and its requirements. The guide is still a work in progress and will be updated with more information in the future.

## Building for macOS and Apple Silicon

You will likely need the pre-requisites regardless. This document is a work in progress. If you notice anything incorrect, unclear, or outdated, please let me know.

Many people might suggest using Brew, but I am old-school and have been building Open Source before package managers existed. Package managers are both a blessing and a curse. I've had bad experiences with Brew and other package managers that manage Open Source and other source-distributed installations.

- Advantages of Package Managers:

  - Package managers handle dependencies.
  - Package managers help keep your system up to date.
  - With package managers, everything is pre-configured.
  - Package managers automatically provide updates.

- Disadvantages of Package Managers:

  - Package managers handle dependencies, which can sometimes lead to unwanted changes.
  - Package managers keep your system up to date, but sometimes you might want to stick with a specific version.
  - With package managers, everything is pre-configured, which can limit customization.
  - Package managers automatically provide updates, which can sometimes break things.

These points illustrate why package managers can be both good and not so good. If you want maximum control over your environment, build it yourself, document it, write some scripts to help automate the process, and figure out something that works for you. However, building everything yourself comes at a cost: it's time-consuming, you need to keep things up to date (though version inconsistencies can still exist either way), and you need to know what you're doing to debug odd problems during a build.

Building is sometimes the best option, such as when you need special options built in or want to use a version other than what is typically distributed. There are many ways to do this, but I'm going to present one method. While it may not be the best, could probably be improved upon, or there's always another way, this is what we're going to do and hopefully, it's simple enough for anyone to follow the directions. In fact, as I am updating this file, I have completely torn down my build environment (making a backup) and am going to follow the steps through here to validate.

I did it the long way so I could ensure I had the proper versions of libraries and modules which, for many reasons, get overlaid, reverted, or uninstalled, and a different version gets installed from a different repository. Some repositories are better in sync than others, but I tried going to the source for these things. I mention using the --dry-run argument, but sometimes the output is difficult to sift through. I will also explain setting up virtual environments or venv using Conda.

## Pre-requisites

Before you begin, there are a few things you'll need.

1. **iTerm2**

    This should be the first thing you download.

    Download iTerm2 here: <https://iterm2.com/downloads.html>

    If you spend any time on the command line, this is a must-have, unless you're content with Terminal.app. There are many configuration options to explore. PROTIP: Set it up for tabbed windows.

    **IMPORTANT NOTE:** This is a universal application. Before you run it, find the application where you installed it, "Right Click" on it, select "Get Info", and ensure that "Open using Rosetta" is not checked. If it is, iTerm will think it's running on an Intel machine, which can cause problems during software builds.

2. **XCode**

    You'll need a compiler. While it would be ideal if GCC ran on macOS, Xcode is a sufficient alternative.

    You can download Xcode from the App Store.

    **IMPORTANT NOTE:** If you ONLY want the command line tools and not the complete Xcode IDE, you can get just the command line tools for Xcode by running the xcode-select command. Open up the iTerm2 you downloaded and installed earlier or Terminal.app in /Applications/Utilities and get the command line tools for XCode like this:

      ```bash
      xcode-select --install
      ```

3. **VSCode**

    Yes, two IDE's, but there are many plugins for VSCode. Unless you're developing macOS or other Apple apps, this is a great IDE. I like it because there's a Vi/Vim mode for it. If you're familiar with the keystrokes for Vi, you're good to go. An integrated terminal means you can run a command line while in the IDE, and it can even do ssh tunneling so you can develop on aremote machine apearing as though everything was local. There are many options, settings, and plugins for this, and finding the right ones may be challenging. However, if you're working with AI or Data Science, you'll likely want this for the Jupyter Notebooks support alone. It also integrates seamlessly with GitHub.

    Make sure you get the "Apple Silicon" zip file. The universal and the Intel version caused me problems when I migrated from my Intel Mac to Apple Silicon because it would run in Rosetta, and everything on the system would report that it was running on Intel when using the terminal. Universal could possibly run inside Rosetta as there is an option on some applications, like iTerm, where when you open the "Get Info" for the application, there is an option to run it using Rosetta. Make sure you don't have the universal build and this box is unchecked.

    Download VSCode here: <https://code.visualstudio.com/Download>

    Unzip the file wherever your downloads are and copy the application to your preferred location.

4. **GNU Coreutils (Optional, but a matter of taste)**

    This isn't strictly a requirement, but more of a personal preference. You might want to use something like Brew to install this. The problem is that the new ls command that comes with macOS displays directory listings in color, just like GNU ls which comes with most Linux distributions. However, the colors and configuration of your colors for macOS is not compatible with GNU ls, is extremely difficult to configure using the scant information in the man page, but the bottom line is the colors they chose for things gives me a headache, so any screenshots of my terminal will be done using GNU ls for directory listings.

    My theory is they are trying to actively discourage people from using the command line.

Apologies for the confusion. Here's the edited version:

## Get Conda (Miniconda)

**NOTE:** If Conda is already installed on your machine, skip this step.

During this process, be cautious as some libraries require the properly compiled version rather than the version that comes with pip or conda. This is important because some extensions for oobabooga may uninstall perfectly fine versions of libraries and downgrade them due to dependencies. This can lead to performance loss and troubleshooting issues. This has happened to me with NumPy and llama.cpp. My goal here is to pay close attention to the libraries during the construction of the environment for running and managing LLMs using oobabooga. I aim to catch as many potential issues as possible.

One way to avoid conflicts, downgrades, and other issues is to use the "--dry-run" argument. This will show you what it plans to do without actually doing it. The output can be lengthy and you might miss things. As an extra precaution, I clone my virtual environments (venv), then switch to the new one before making any potentially harmful changes.

```bash
   cd
   mkdir tmp
   cd tmp
   curl  https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh
   sh miniconda.sh
   # Replace the shell name below with your preferred shell. The -l switch gives you a login shell and, contrary to what you may heard, you don't have to log out or exit the terminal. Simply exec the shell and it will reload your environment variables with the additional Conda ones set. This also works in Linux and most other Unix-like POSIX operating systems.
   exec bash -l
```

After installing miniconda and conda, you may get a message indicating it needs updating. It's a good idea to go ahead and update the environment.

```bash
conda update -n base -c defaults conda
```

Create a new venv using Python 3.10. This will serve as your base virtual environment for anything you wish to use with Python 3.10. This is the version you need for running oobabooga. If you have another project, you can always return to the base and build from there. This helps avoid the issue of conflicting versions resulting from using package managers.

```bash
conda create -n python3.10 python==3.10.*
conda activate python3.10
```

This gives us a clean environment to return to as a base. I tend to clone my conda venvs so it's easy to roll back any changes that have negatively impacted my environment. It saves time to be able to roll back to a known good environment and move forward again. These venvs are useful for rolling back to a known configuration. I recommend cloning your good venv, activating it, and applying any changes to that. Many packages or updates affect multiple python modules at once, and this is an easy way to roll back and then move forward, creating a new venv cloned from the previous one. Then, new items are installed into that venv. When it's working, clone that one, activate it, and do the next round of updates or changes. At any point, venvs can be completely removed and even renamed. So, you can take your final venv, if you're happy with it, and rename it back to the base for your application. I will try to do this as I go along in this installation, taking venv checkpoints which I can roll back to if needed.

Cloning a venv can also help you quickly determine if a compile, or some other module, provides any performance advantage. I can explain some of these techniques at another time.

## CMake

Ensure you have CMake installed. Many dependencies rely on CMake, which is beneficial as it builds based on the original hardware and software configuration of your machine.

You can find it here: <https://cmake.org/download/>

CMake is easy to install and will be needed for later steps like llama.cpp, llama-cpp-python, and other modules.

Download the latest source version of CMake. Avoid using the packages as they are universal binaries, and you might accidentally end up building something with x86_64 architecture. This is unverified, but it's better to be safe.

A lot of issues surrounding getting all this to work stem from various machines building libraries and packages running universal binaries through Rosetta. This allows them to run on macOS, but not necessarily take advantage of the M1/M2 system on chip and unified memory. I discovered that a number of libraries are universal binaries, which could be an issue. I first noticed this when I was looking in the "Activity Monitor" and was surprised when oobabooga came up running as "Intel". This was a result of my VSCode running using Rosetta.

**NOTE:** I am using a recent copy of GNU Make, which is a parallelizing make. Apple's make with macOS is an older version of GNU Make - 3.81, so it should be fine as well.

**NOTE:** This will want to install in /usr/local. You may not want it installing there, and there are some special things you may have to do for it to install there. I will update this later with information on how to get it installed in something like ${HOME}/local/bin, which works just fine too.

The steps are pretty simple and only take about 5 minutes:

```bash
# untar the tar file, it will create a cmake_source directory.
# cd cmake_source
./bootstrap
# I do this on the 12 core M2 Max and it flies. My last build I
# did -j30, it was fine.
make -j24
make test
make install
```

This creates 24 compile jobs. I have 12 cores on my MBP, so I use 2 times cores. This works rather well and builds quickly. Make should parallelize as much as it can based on dependencies.

## NumPy - Everything or Quickly

### NymPy Build Everything - OpenBLAS

Building OpenBLAS might be optional, but it can be built for incorporation into your own build of NumPy. If you want to get down into the nuts and bolts, then go ahead and build your own NumPy. However, installing using Conda will bring down a number of other libraries and Python modules, so you're probably best using conda to do this. However, if you compile and link your own, you can be 100% sure it will use the OpenBLAS and LAPACK you build here. I will show how to build OpenBLAS and NumPy using your library for OpenBLAS.

OpenBLAS (BLAS) and LAPACK - Basic Linear Algebra and Linear Algebra Pack. These were originally built in FORTRAN, and you may find them on netlib.org.  I didn't have the GNU Fortran compiler available and the GNU compiler collection has been a problem on macOS for quite a while since they changed their binary formats and use LLVM.  You must use their C compiler, but it works pretty well.  Make sure you have Xcode and the Xcode command line utilities loaded. (Actually before you begin any of this.)

**NOTE** This will want to install in /usr/local you may not want it installing there and there are some special things you may have to do in order for it to install there. I will update this later with information on how to get it installed in something like ${HOME}/local/bin which works just fine too.

Download the OpenBLAS repo and build it, then install.

```bash
git clone https://github.com/xianyi/OpenBLAS
cd OpenBLAS
cmake .
make -j24
make test
make install
```

### NumPy

With the installation of NumPy, we begin to change the nature of the Python environment and from here on, we will be making clones of the venv's in order to roll back anything which we need to change or fix before moving forward.

To create a clone of my baseline to this point, just in case something goes wrong, like other modules being updated or for some reason we want to roll this back. This one is fairly easy, since pip only installs one module for NumPy, however using conda, will install several modules, presumably as dependencies though pip doesn't. One thing which is included separately in the conda install is the OpenBLAS libraries.

```bash
conda cradte --clone python3.10 -n tgeb.00.numpy
conda deactivate
conda activate tgen.00.numpy
pip install --no-cache --no-binary :all: --compile numpy
```

You can check to see if this worked by opening up an interactive python session on the command line type the following:

```bash
python

import numpy
numpy.show_config()
```

### NumPy Quicker - Use Conda or Pip

Clone the untouched python3.10. I've found that teh following naming convention woeks well for being able to roll back:

  tgen.00.numpy

- appcode, a code for the application.
- sequence, this number will give me the sequence the venv was craeted in as a reference and is incremented for each recovery milestone.
- milestone, this is an identifier for the milestone, suc as here "numpy" we are about to install numpy on top of our environment.

 This convention allows us to easilky identify what the packages were or the significance of the milestobe in th eenvironment build process.  Removing the venv will remove fron that milestone forward, at least symbolically. The later venvs will still be installed, but I've found it's good to remove them too as I roll back, thoug hsome might be left in place for testing purposes.

 The sequence number will help in understanding what happened every step along the way, and can also be useful if you decide to list the venvs using:

 ```bash
conda info -e  | grep -v \# | sort
 ```

This will list your environments in th eorder they weer created if you use the sequence number and can help you determine what you might want to roll back.  Adding a .n.nn to th enumber could also signify a branch.  This si a lot to maintain manually, but is helpful when tracking down module dependencies and helping to keep one's view of the envirnments clear.

```bash
conda cradte --clone python3.10 -n tgeb.00.numpy
conda deactivate
conda activate tgen.00.numpy
conda install numpy
```

Either one of these options is viable, but if you go the OpenBLAS route, you definitely know what you have and it was built in your environment.

The main point of this installation was that during my build-up of the environment, I had a version of NumPy which was not configured correctly. So, whichever way you decide, make sure you validate nothing has pulled out your modules and put in ones you don't want or are questionable.

## PyTorch

Let's now install PyTorch or Torch and see what happens by using either pip, conda, or the instructions on the PyTorch site. PyTorch says there are two ways to install PyTorch/Torch. One using pip and the other with conda. They are slightly different builds. PyTorch is probably the most important package we install for oobabooga and most any other AI/ML application.

Method 1 is with Conda and is the preferred way to install, according to the [PyTorch documentation](https://pytorch.org/get-started/locally/#macos-version). Then clone the environment it was built in so we can roll back and move forward or fork if we want.

```bash
conda create --clone tgeb.00.numpy -n tgeb.01.torch
conda deactivate 
conda activate tgeb.01.torch
conda install pytorch torchvision torchaudio -c pytorch
```

Method 2 is with pip. After it's complete, we should clone the environment as a checkpoint for rolling back.

```bash
conda create --clone tgeb.00.numpy -n tgeb.01.torch
conda deactivate 
conda activate tgeb.01.torch
pip install torch torchvision torchaudio
```

Later library and module installations may require re-installs of PyTorch or numpy. For instance, I know that as it is now, Open Whisper downgrades and uses a different NumPy and the latest version of the Whisper modules will not use the latest NumPy.

Create the venv for whichever PyTorch installation you wish to use going forward, or use both of them and build them up as separate environments for testing purposes. Either should work, and I'll soon have some scripts which stress test MPS with dummy data for tensors, but can validate the GPU for MPS is used.

## oobabooga Base - Everything Else

Pick one of the venv's from the Torch install you wish to use, or use both of them. If at any time you wish to see what Conda environments you have along with the one which is active, use the following:

```bash
conda info -e
```

### Clone The oobabooga GitHub Repository

At this ppoint, get started setting up oobabooga in your working location, we'll use it later, referring to the requirements.txt to see which Python modules we will need.

Pick a good location for your clone of the project. I have a projects directory with several sub-directories off of it to contain certain projects and source code, but you can pick any place you want. For me, I use ~/.projects/AI as the location where I place anything related to AI. So, open up iTerm and create a location and get the oobabooga text-generation-webui repository.

This will pull clone my repository which has some changes that allow it to run with GPU acceleration.  This is unsuported, by th eoobabooga people, but I will try to keep my information as up-to-date as possible along with merging code into the repository on a regular basis.

```bash
cd
mkdir -p projects/AI
cd projects/AI
git clone https://github.com/unixwzrd/text-generation-webui-macos.git
```

Alternately, you could get the original oobabooga and try running it using this set of installation instructions.

```bash
cd
mkdir -p projects/AI
cd projects/AI
git clone https://github.com/oobabooga/text-generation-webui.git
```

### Install oobabooga Requirements

Change into the oobabooga text-generation-webui directory you cloned from GitHub earlier. Due to the structure of the file, it must be done with pip.

```bash
conda create --clone tgen.01.torch -n tgen.02.oobaboogabase
conda deactivate
conda activate tgen.02.oobaboogabase 
pip install -r requirements.txt
```

Now, at this point, we have everything we need to run the basic server with no extensions. However, we should have a look at the llama.cpp and llama-cpp-python as we may need to build them ourselves.

## Llama for macOS and MPS

The one loaded with the requirements for oobabooga is not compiled for MPS (Metal Performance Shaders) installed from PyPi at this time.

You're going to need the llama library and the Python module for it. You should recompile it, and I have validated that my build using OpenBLAS. I will also add instructions later for building a stand-alone llama.cpp which can run by itself. This is handy in case you don't want the entire UI running, you want to use it for testing, or you only need the stand-alone version.

The application llama.cpp compiles with MPS support. I'm not sure if the cmake configuration takes care of it in th elamma-cpp repository build, but the flag -DLLAMA_METAL=on is required here.  When I comipled lamma-cpp in order to compare its performance to the lamma-cpp-python. I dodn't have to specify any flags andit just built right out of the box. This could have been due to the configuration of CMake as it thoroughly probes the system for its installed software and capabilities in order to make decisions when it creates the makefile. It is required in this case.

```bash
conda create --clone tgen.02.oobaboogabase -n tgen.03.reblds
conda deactivate
conda activate tgen.03.reblds
pip uninstall -y llama-cpp-python
CMAKE_ARGS="-DLLAMA_METAL=ON -DLLAMA_OPENBLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" \
    FORCE_CMAKE=1 \
    pip install --no-cache --no-binary :all: --upgrade --compile llama-cpp-python==0.1.74
```

### Buiklding llama-cpp-pythin from source

This amy also be guilt from the latest source if you want to and installe and installed directly from your local repositoiry.

```bash
conda create --clone tgen.02.oobaboogabase -n tgen.03.reblds
conda deactivate
conda activate tgen.03.reblds
pip uninstall -y llama-cpp-python
CMAKE_ARGS="--fresh -DLLAMA_METAL=ON -DLLAMA_OPENBLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" \
    FORCE_CMAKE=1 \
    pip install --no-cache --no-binary :all: --upgrade --compile -e .
```

**NOTE** when you run this you will need to make sure whatever application is using this is specifying number of GPU or GPU layers greater than zero, it shoudl be at least one for teh GGML library to allocate space in the Applie Silicon M1 or M2 GPU space.

## Pandas

Pandas is now on the requirements list and I belieev it was being installed prior to that.  It's probably a good idea, just like some of the other modules to do a forced recompile  I haven't investigated whether or not it uses MPS, but it should probably be included.

```bash
# Optional, but will give finer granularity of you need to rollback.
conda create --clone tgen.03.reblds -n tgen.04.pandas
conda deactivate
conda activate tgen.04.pandas
pip uninstall -y pandas
pip install --no-cache --no-binary :all: --compile llama-cpp-python
```

## PyTorch for macOS and MPS

It may be necessary to re-install NumPy or at least upgrade it due to another module having different requiremnts and a diffrent version of a module like NumPy. The OpenWhisper python modules is incompatible with the latest NumPy, at least at the tiem I wrote this, but please let me know if you have additional information.

This seems to be the best method rather than using pip to install, the collection seems more up to date and more comprehensive than PyPi, this is actually coming from the source of PyTorch itself, so they are most likely the most up-to-date.  They also have nightly builds of you like to live on the edge.

```bash
conda create --clone tgen.04.pandas -n tgen.05.torch2
conda deactivate
conda activate tgen.05.torch2
conda install pytorch torchvision torchaudio -c pytorch
```

## Where We Are

This is a lot to cover, but there are more modules which get mis-installed and need to be repaired, re-installed, or built from source. This package has a lot o fmodules and a lot of dependecies, so expect breakage from time to time.  Making checkpoints fo rrollback along the way will help a lot if you get a bad modulke, you won't have to destroy your whole venv or figure out which modules need to be uninstalled and re-installed.

Once you feel comfortable with your checkpoints and working venv, you can remove some of th eones you aren't using and this will improve Conda's performance.

At his point, LLaMA models dhold start up just fine as long as they are GGML formatted models and you shoul see a noticiable performance improvement.  Put as many GPU layers as you possibly can and set the threads at a reasonable number like 8.

## Extensions

There are a number of extensions you can use with oobabooga textgen, but som ebreak other things with their requirements. At this point, here are the extensions I have had no problems with so far:

- elevenlabs
- silero

    These are both for TTS, Text To Speech, one reies on ElevenLabs to generate the speech, and the other runs locally using a torch speech model. I'd be interested in discovering other TTS ackages available which could be hosted locally withoutr relying on th eInternet.

- Whisper

  This is the speech to text utility, modules or libraries from OpenAI, which I believe may be hosted locally as th emodel it uses is fairly small, though I haven't had a chance to check it out yet.

  The issue with Whisper is that it requires some older Python packages wihch will cause NumPy to be downgraded and there you have a problem.  Hopefully this will be sorted out soon.

## NOTE THIS IS INCOMPLETE- To be continued

There are a number of other modules like Pandas and SciPy wihich need review on Apple Silicon, many of these are usetd in oobababooga and elsewhere.

Please run through the steps and feel free to comment, update, clarify, or contribute.
