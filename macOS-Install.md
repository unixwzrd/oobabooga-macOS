# Apple Silicon Support for oobabooga text-generation-webui

This guide provides instructions on how to build and run the oobabooga text-generation-webui on macOS, specifically on Apple Silicon.

This repository is primarily for oobabooga users at the moment, many of the Python libraries and packages used here may also be used for Data Analytics, Machine Learning and other purposes.

I have a new repository on the way to assist with Apple Silicon M1/M2 and GPU performance VENV builds. This will produce configurable, repeatable, consistent VENV builds for Python packages and modules in all types of layering/stacking and at some point soon, branching builds. This will allow different installation procedures to be compared and evaluated for performance and through regression tests. Getting these consistent, working builds has been a bit difficult as new packages come out all the time and there are many cross-module/package dependencies, some incompatible, and some in conflict.

## 01 Nov 2023 - This is updated with new build instructions for everything really. Lots has changed and more is stable now.

Latest != Greatest, Latest + Greatest != Best, Stable == None

## TL;DR

1. **Python**: Install Python 3.10 using Miniconda. Create a virtual environment and install pip.
1. **CMake**: Install CMake from source to avoid potential issues with universal binaries. This is used for building other software.
1. **OpenBLAS**: Build and install OpenBLAS, a library for linear algebra operations. This can optionally be used when building NumPy.
1. **oobabooga Base**: Clone the oobabooga GitHub repository and install the Python modules listed in its requirements.txt file.
1. **Llama for macOS and MPS**: Uninstall any existing version of llama-cpp-python, then reinstall it with specific CMake arguments to enable Metal support.
1. **PyTorch for macOS and MPS**: Install PyTorch, torchvision, and torchaudio from the PyTorch Conda channel.

Check out [oobabooga macOS Apple Silicon Quick Start for the Impatient](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS_Apple_Silicon_QuickStart.md) for the short method without explanations.

Throughout the process, you're advised to create clones of your Conda environments at various stages. This allows you to easily roll back to a previous state if something goes wrong.

Please note that the guide is incomplete and is expected to be continued.

- [Apple Silicon Support for oobabooga text-generation-webui](#apple-silicon-support-for-oobabooga-text-generation-webui)
  - [01 Nov 2023 - This is updated with new build instructions for everything really. Lots has changed and more is stable now.](#01-nov-2023---this-is-updated-with-new-build-instructions-for-everything-really-lots-has-changed-and-more-is-stable-now)
  - [TL;DR](#tldr)
    - [Status of Testing and BLAS](#status-of-testing-and-blas)
  - [Building for macOS and Apple Silicon](#building-for-macos-and-apple-silicon)
  - [Pre-requisites](#pre-requisites)
  - [Some initial setup](#some-initial-setup)
  - [CMake](#cmake)
  - [Get Conda (Miniconda)](#get-conda-miniconda)
  - [Verify we have everything set up for the rest of the build and install](#verify-we-have-everything-set-up-for-the-rest-of-the-build-and-install)
  - [Clone my oobabooga macOS GitHub Repository](#clone-my-oobabooga-macos-github-repository)
  - [Pip Install the PyTorch Daily Build](#pip-install-the-pytorch-daily-build)
    - [Using Pip for PyTorch](#using-pip-for-pytorch)
  - [Llama for macOS and MPS (Metal Performance Shaders)](#llama-for-macos-and-mps-metal-performance-shaders)
  - [NunPy](#nunpy)
  - [CTransformers](#ctransformers)
  - [Nearly finished](#nearly-finished)
  - [Where We Are](#where-we-are)
  - [Extensions](#extensions)



This guide is quite comprehensive and covers everything from getting the necessary prerequisites to building and installing all the required components. It also includes a section on how to clone and install the oobabooga repository and its requirements. The guide is still a work in progress and will be updated with more information in the future.

### Status of Testing and BLAS

The benchmarking project is going fairly well.  It took much more time to gather all the information on the linear algebra libraries, discovering other projects who use linear algebra, matrix manipulation, vector and tensor processing to discover that everyone is pretty much working in their own sandboxes. I was very surprised to see that there are no BLAS (Basic Linear Algebra Subroutines/System/Software) which would build or are built, at this time, other than the Apple Accelerate Framework, which appears to have little support from apple for the mathematical and scientific community, instead focusing more on consumer products like the iPhone. It seems a good deal of effort is put to consumer goods as they have a larger market share than Macs do.

Some people have built their own libraries like it seems with PyTorch, SciPy, and even GGML/GGUF are using their own libraries to handle the matrix manipulation. I'm very surprised at this and how little support there seems to come from Apple, but that's their market they are playing to. Macs make up around 6% of their revenue, and it's unlikely they will ever enter the Datacenter market for other than their own hardware. People it seems have found a way around what seems through my research to be a two or three year period where Apple was doing little in helping the Open Source data science people, AI people, or the mathematics community. Never mind the apparent lack off cooperation between the various project teams and not pointing any fingers here, it seems to me there could have been a bit more collaboration. Reading their PR's here on GitHub was rather interesting as it seems one project or another would be keeping tabs on what another project was doing, but there was never any coordinated effort to Open Source the whole thing for everyone. This may be one of the weaknesses of Open Source in not having any sort of central governing body helping to connect and guide resources in a particular direction to solve a problem for the common good. I might have missed something in the few weeks of research I did, but that was my impression, each group of developers watching the others and finally when one group started working on Apple Silicon support, they would take that as their cue to work on theirs instead of working together, maybe I'm wrong.

I have gathered a lot of information and it seems somewhat anticlimactic, but I seem to have found the best way I can to get the best performance possible from has been a moving target the past couple of weeks and will probably continue to be that way. It seems that at the core of the issue is NumPy is used by pretty much everyone to define core data types, and some other things I haven't discovered as my focus was in getting the most performance I could out of the Apple Silicon. I started down the path of eventually putting together a package building script in Bash and only near the end, discovered manipulating the graph data structure of layered and branching packages and libraries in a VENV was more than I could easily and simply handle in Bash. It does give consistent repeatable builds using a configuration file, and I am moving to Python in order to traverse the graph structure. Anyway, I will be opening another repository in a few days to throw everything in there.  My plan id to also use the package benchmark and regression testing suites to see different stacking, libraries and other issues will compare for compatibility, performance and overall function.

I'd hoped to have all this out by now, but I kept finding new information, but for now, I can at least give what seems to be my best possible configuration for running LLM's on Apple Silicon, and the few libraries I've looked at. As I mentioned NumPy seemed to be the bottle neck in all this, but they were also the project which gave me a clue as to what I was trying to achieve and that is all out raw BLAS performance. For now it's time to give this document a refresh and update it with new information. I will say that performance I am seeing is somewhere between 2-5 tokens/sec, maybe more at times, and it's very usable and that's with the 70B LLaMa2 model with four bit quantization. I do not have hard numbers, but wanted to get my build out there as soon as possible. The hardware is a MacBook Pro M2 Max with 96GB RAM.

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

I did it the long way so I could ensure I had the proper versions of libraries and modules which, for many reasons, get overlaid, reverted, or uninstalled, and a different version gets installed from a different repository. Some repositories are better in sync than others, but I tried going to the source for these things. I mention using the --dry-run argument, but sometimes the output is difficult to sift through. I will also explain setting up virtual environments or VENV using Conda.

## Pre-requisites

Before you begin, there are a few things you'll need.

1. **iTerm2**

    This should be the first thing you download.

    Download iTerm2 here: <https://iterm2.com/downloads.html>

    If you spend any time on the command line, this is a must-have, unless you're content with Terminal.app. There are many configuration options to explore. PROTIP: Set it up for tabbed windows.

    **IMPORTANT NOTE:** This is a universal application. Before you run it, find the application where you installed it, "Right Click" on it, select "Get Info", and ensure that "Open using Rosetta" is not checked. If it is, iTerm will think it's running on an Intel machine, which can cause problems during software builds.

2. **Xcode**

    You'll need a compiler. While it would be ideal if GCC ran on macOS, Xcode is a sufficient alternative.

    You can download Xcode from the App Store.

    **IMPORTANT NOTE:** If you ONLY want the command line tools and not the complete Xcode IDE, you can get just the command line tools for Xcode by running the xcode-select command. Open up the iTerm2 you downloaded and installed earlier or Terminal.app in /Applications/Utilities and get the command line tools for Xcode like this:

      ```bash
      xcode-select --install
      ```

3. **VSCode**

    Yes, two IDE's, but there are many plugins for VSCode. Unless you're developing macOS or other Apple apps, this is a great IDE. I like it because there's a Vi/Vim mode for it. If you're familiar with the keystrokes for Vi, you're good to go. An integrated terminal means you can run a command line while in the IDE, and it can even do ssh tunneling so you can develop on a remote machine appearing as though everything was local. There are many options, settings, and plugins for this, and finding the right ones may be challenging. However, if you're working with AI or Data Science, you'll likely want this for the Jupyter Notebooks support alone. It also integrates seamlessly with GitHub.

    Make sure you get the "Apple Silicon" zip file. The universal and the Intel version caused me problems when I migrated from my Intel Mac to Apple Silicon because it would run in Rosetta, and everything on the system would report that it was running on Intel when using the terminal. Universal could possibly run inside Rosetta as there is an option on some applications, like iTerm, where when you open the "Get Info" for the application, there is an option to run it using Rosetta. Make sure you don't have the universal build and this box is unchecked.

    Download VSCode here: <https://code.visualstudio.com/Download>

    Unzip the file wherever your downloads are and copy the application to your preferred location.

4. **GNU Coreutils (Optional, but a matter of taste)**

    This isn't strictly a requirement, but more of a personal preference. You might want to use something like Brew to install this. The problem is that the new ls command that comes with macOS displays directory listings in color, just like GNU ls which comes with most Linux distributions. However, the colors and configuration of your colors for macOS is not compatible with GNU ls, is extremely difficult to configure using the scant information in the man page, but the bottom line is the colors they chose for things gives me a headache, so any screenshots of my terminal will be done using GNU ls for directory listings.

    My theory is they are trying to actively discourage people from using the command line.

## Some initial setup

You will need to have your environment set up for all the following steps to work. These need to be done so your installation will go as smoothly as possible. You may wish to change some of these items for how you like to do things. However they should work for pretty much any non-privileged user.

```bash
### These commands are for a bash shell. I tryed switching to Zsh, but I have to much legacy with bash it
### wasted a lot of my time trying to get all my accumulated stuff to work with Zsh.
###
### So, let's make sure we are using a fresh bash shell.
exec bash -l

### Choose a target directory for everything to be put into, I'm using "${HOME}/projects/ai-projects" You
### may use whatever you wish. These must be exported because we will exec a new login shell later. "Normal" shell variables will not be passed to th enew login shell, we are just setting them up front.
export TARGET_DIR="${HOME}/projects/si-projects"
export MACOS_LLAMA_ENV="macOS-llama-env"

### Set a reasonable umask - this controls the default permissions for your files when they are created.
umask 0022


### Create the target directory where we sill be dowloading, building and installing from.
mkdir -p "${TARGET_DIR}'"
cd "'${TARGET_DIR}'"

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
```

## CMake

You will need the latest version of CMake, at least version 3.29.3. Make sure you have it installed and working. You may already have CMake installed, if you do, skip this step, but verify you are using the proper version. Many dependencies rely on CMake, which is beneficial as it builds based on the original hardware and software configuration of your machine.

You can find it here: <https://cmake.org/download/>

CMake is easy to install and will be needed for later steps like llama.cpp, llama-cpp-python, and other modules.

Download the latest source version of CMake. Avoid using the packages as they are universal binaries, and you might accidentally end up building something with x86_64 architecture. This is unverified, but it's better to be safe.

A lot of issues surrounding getting all this to work stem from various machines building libraries and packages running universal binaries through Rosetta. This allows them to run on macOS, but not necessarily take advantage of the M1/M2 system on chip and unified memory. I discovered that a number of libraries are universal binaries, which could be an issue. I first noticed this when I was looking in the "Activity Monitor" and was surprised when oobabooga came up running as "Intel". This was a result of my VSCode running using Rosetta.

**NOTE:** I am using a recent copy of GNU Make, which is a parallelizing make. Apple's make with macOS is an older version of GNU Make - 3.81, so it should be fine as well.

**NOTE** This will want to install in ${HOME}/local/bin. Alternatively, you could install in /usr/local but you will need administrator access and possibly have to disable Apples SIP (System Integrity Protection), a process I will not go into here as it affects overall system protection. Either way, you will need to make sure that where ever you install it, it is in your path.  I Am going to assume ${HOME}/local/bin in these instructions.

**NOTE:** This will want to install in /usr/local. You may not want it installing there, and there are some special things you may have to do for it to install there. I will update this later with information on how to get it installed in something like ${HOME}/local/bin, which works just fine too, as long as it's in your PATH.

The steps are pretty simple and only take about 5 minutes:

```bash
### Clone the CMake repository, build, and install CMake
git clone https://github.com/Kitware/CMake.git
cd CMake
git checkout tags/v3.29.3
mkdir build
cd build

### This will configure the installation of cmake to be in your home directory under local, rather than /usr/local
### This is just preference and will work for a non-privilged user.
../bootstrap --prefix=${HOME}/local
make -j
make -j test
make install
```

This creates 24 compile jobs. I have 12 cores on my MBP, so I use 2 times cores. This works rather well and builds quickly. Make should parallelize as much as it can based on dependencies.

## Get Conda (Miniconda)

**NOTE:** If Conda is already installed on your machine, skip this step, but this will also ensure your Conda setup is up-to-date. We're going to skip over the NumPy rebuild here because the llama-cpp-python build will bring NumPy along with it, and the Conda installation of PyTorch also brings along a different NumPy with support libraries in a "hidden" package called "numpy-base".

During this process, be cautious as some libraries require the properly compiled version rather than the version that comes with pip or conda. This is important because some extensions for oobabooga may uninstall perfectly fine versions of libraries and downgrade them due to dependencies. This can lead to performance loss and troubleshooting issues. This has happened to me with NumPy and llama.cpp. My goal here is to pay close attention to the libraries during the construction of the environment for running and managing LLMs using oobabooga. I aim to catch as many potential issues as possible.

One way to avoid conflicts, downgrades, and other issues is to use the "--dry-run" argument. This will show you what it plans to do without actually doing it. The output can be lengthy and you might miss things. As an extra precaution, I clone my virtual environments (VENV), then switch to the new one before making any potentially harmful changes.

```bash
 cd
 mkdir tmp
 cd tmp
 curl  https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh
 # Do a non-destructive Conda install whcih will preserve existing VENV's
 sh miniconda.sh -b -u

 # Activate the conda environment.
 source ${HOME}/miniconda3/bin/activate

  # Initialize Conda which will add initialization functions to your shell's profile.
 conda init $(basename ${SHELL})

  # Update your Conda environment with the latest updates to th ebase environment
 conda update -n base -c defaults conda -y

 # Grab a new login shell - this will work for any shell aand you wil enter back in the
 # tmp directory we just created..
  exec bash -l
```

Create a new VENV using Python 3.10. This will serve as your base virtual environment for anything you wish to use with Python 3.10. This is the version you need for running oobabooga. If you have another project, you can always return to the base and build from there. This helps avoid the issue of conflicting versions resulting from using package managers.

```bash
#### Create the base Python 3.10 and the llama-env VENV.
conda create -n ${MACOS_LLAMA_ENV} python=3.10 -y
conda activate ${MACOS_LLAMA_ENV}
```

This gives us a clean environment to return to as a base. I tend to clone my conda VENVs so it's easy to roll back any changes that have negatively impacted my environment. It saves time to be able to roll back to a known good environment and move forward again. These VENVs are useful for rolling back to a known configuration. I recommend cloning your good VENV, activating it, and applying any changes to that. Many packages or updates affect multiple python modules at once, and this is an easy way to roll back and then move forward, creating a new VENV cloned from the previous one. Then, new items are installed into that VENV. When it's working, clone that one, activate it, and do the next round of updates or changes. At any point, VENVs can be completely removed and even renamed. So, you can take your final VENV, if you're happy with it, and rename it back to the base for your application. I will try to do this as I go along in this installation, taking VENV checkpoints which I can roll back to if needed.

Cloning a VENV can also help you quickly determine if a compile, or some other module, provides any performance advantage. I can explain some of these techniques at another time.

## Verify we have everything set up for the rest of the build and install

Make sure we can find the CMAke we installed earlier and make sure we are in the target directory.

```bash
### Verify the installation
which cmake       # Should say $HOME/local/bin

### Verify you are running cmake z3.29.3
cmake --version

### Change to the target directory.
cd "'${TARGET_DIR}'"
```

## Clone my oobabooga macOS GitHub Repository

**NOTE THIS IS A DEVELOPMENT BUILD - IT WILL BE PROMOTED TO TEET SOON**

At this point, get started setting up oobabooga in your working location, we'll use it later, referring to the requirements.txt to see which Python modules we will need.

Pick a good location for your clone of the project. I have a projects directory with several sub-directories off of it to contain certain projects and source code, but you can pick any place you want. For me, I use ~/.projects/AI as the location where I place anything related to AI. So, open up iTerm and create a location and get the oobabooga text-generation-webui repository.

This will pull clone my repository which has some changes that allow it to run with GPU acceleration.  This is unsupported, by the oobabooga people, but I will try to keep my information as up-to-date as possible along with merging code into the repository on a regular basis.

```bash
### Get my oobabooga and checkout macOS-test branch
git clone https://github.com/unixwzrd/text-generation-webui-macos.git textgen-macOS
cd textgen-macOS
### Checkout the development build, this may change later, and I will update this here.
git checkout macOS-dev
pip install -r requirements.txt
```

## Pip Install the PyTorch Daily Build

### Using Pip for PyTorch

This will install the latest PyTorch optimized for Apple Silicon.

```bash
## Pip install from daily build
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu --force-reinstall --no-deps
```

Now, at this point, we have everything we need to run the basic server with no extensions. However, we should have a look at the llama.cpp and llama-cpp-python as we may need to build them ourselves.

## Llama for macOS and MPS (Metal Performance Shaders)

The one loaded with the requirements for oobabooga is not compiled for MPS (Metal Performance Shaders) installed from PyPi at this time. It is also probably best to build your own anyway.

You're going to need the llama library and the Python module for it. You should recompile it, and I have validated that my build using OpenBLAS. I will also add instructions later for building a stand-alone llama.cpp which can run by itself. This is handy in case you don't want the entire UI running, you want to use it for testing, or you only need the stand-alone version.

The application llama.cpp compiles with MPS support. I'm not sure if the cmake configuration takes care of it in the llama-cpp repository build, but the flag -DLLAMA_METAL=on is required here.  When I compiled llama-cpp in order to compare its performance to the llama-cpp-python. I didn’t have to specify any flags and it just built right out of the box. This could have been due to the configuration of CMake as it thoroughly probes the system for its installed software and capabilities in order to make decisions when it creates the makefile. It is required in this case.

```bash
export CMAKE_ARGS="-DLLAMA_METAL=on"
export FORCE_CMAKE=1
export PATH=/usr/local/bin:$PATH  # Ensure the correct cmake is used
pip install llama-cpp-python --force-reinstall --no-cache --no-binary :all: --compile --no-deps --no-build-isolation

```

## NunPy

NumPy is finally supporting Apple Silicon. You will have to compile it on install. Many packages I've found want to install their preferred version of NumPy or other NumPy support libraries. This will likely uninstall your NumPy in your VENV. You should be on the lookout when you install anything new that it does not overlay your NumPy with a previous version or a different installation of the current version.

## CTransformers

I include this one, but haven't tested it and it's unclear is it works properly on macOS.

```bash
export CFLAGS="-I/System/Library/Frameworks/vecLib.framework/Headers -Wl,-framework -Wl,Accelerate -framework Accelerate"
pip install numpy --force-reinstall --no-deps --no-cache --no-binary :all: --no-build-isolation --compile -Csetup-args=-Dblas=accelerate -Csetup-args=-Dlapack=accelerate -Csetup-args=-Duse-ilp64=true
```

## Nearly finished

While there are more advanced instructions in the "QuickStart" guide, basically you are now finished. In the command window you are in, you can set the preferred VENV to use, and the start options for the webui. This creates a short script for starting the webui from the command line or you may open Finder to the location you gave installed and simply double click on the file "start-webui.sh and it should run in a terminal window. The options may be edited later in the start-webui.sh created here.

```bash
# Add any startup options you wich to this here:
START_OPTIONS=""
#START_OPTIONS="--verbose "
#START_OPTIONS="--verbose --listen"

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
```

## Where We Are

This is a lot to cover, but there are more modules which get mis-installed and need to be repaired, re-installed, or built from source. This package has a lot of modules and a lot of dependencies, so expect breakage from time to time.  Making checkpoints for rollback along the way will help a lot if you get a bad module, you won't have to destroy your whole VENV or figure out which modules need to be uninstalled and re-installed.

Once you feel comfortable with your checkpoints and working VENV, you can remove some of the ones you aren't using and this will improve Conda's performance.

At his point, LLaMA models should start up just fine as long as they are GGUF                      formatted models and you should see a noticeable performance improvement.  Put as many GPU layers as you possibly can and set the threads at a reasonable number like 8.

Some other numbers, and parameters of note which I have verified through testing.  If you have others, please let me know and I'll have a look at them and add them is they work well.

| Parameter    |  Value                                                                           |
| ------------ | -------------------------------------------------------------------------------- |
| n_gpu_layers | Set this to the number of n_layers in the output of llama.cpp when it starts     |
| mlock        | Set this on, this will pin the memory so it doesn't get paged out or compressed  |
| n_batch      | the number of batches for each iteration, if someone has guidance for this, please let me know. |
| no_mmap      | I use this because with mlock set, there should be no need to reference the model as a file. |

## Extensions

There are a number of extensions you can use with oobabooga textgen, but som break other things with their requirements. At this point, here are the extensions I have had no problems with so far:

All the extensions should work with this version 

- Elevenlabs

  I've included this one because i use it, but am working on other TTS options which will run locally, like AllTalk and other Coqui-based solutions.
  
- Silero

    These are both for TTS, Text To Speech, one relies on ElevenLabs to generate the speech, and the other runs locally using a torch speech model. I'd be interested in discovering other TTS packages available which could be hosted locally without relying on th eInternet.

- Whisper

  This is the speech to text utility, modules or libraries from OpenAI, which I believe may be hosted locally as the model it uses is fairly small, though I haven't had a chance to check it out yet.

  The issue with Whisper is that it requires some older Python packages which will cause NumPy to be downgraded and there you have a problem.  Hopefully this will be sorted out soon.
