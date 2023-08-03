# Use the GPU on your Apple Silicon Mac

This stared out as a guide to getting oobabooga working with Apple Silicon better, but has turned out to contain now useful information regarding how to get numerical analysis, data science, and AI core software running to take advantage of the Apple Silicon M1 and M2 processor technologies. There is information in the guides for installing OpenBLAS, LAPACK, Pandas, NumPy, PyTorch/Torch and llama-cpp-python. I will probably create a new repository for all things Apple Silicon in the interest of getting maximum performance out of the M1 and M2 architecture.

## You probably want this: [Building Apple Silicon Support for oobabooga text-generation-webui](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS-Install.md)

## If you hate standing in line at the bank: [oobabooga macOS Apple Silicon Quick Start for the Impatient](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS_Apple_Silicon_QuickStart.md)

In the test-scripts directory, there are some random Python scripts using tensors to test things like data types for MPS and other compute engines.  Nothing special, just ahchackedked together in a few minutes for checking GPU utilization and AutoCast Data Typing.

## 3 Aug 2023 - Coming Soon oobabooga 1.5 integration and Coqui for macOS

Currently I am finishing up changes to the release 1.5 oobabooga and integrating them into my fork and may or may not have additional performance improvements I have identified for Apple Silicon M1/M2 GPU acceleration.  I hope by the end of this week.  I will release it in my test fork I created to allow people to test and provide feedback.

I also got distracted earlier in the week by looking for another TTS alternative to Elevenlabs which runs locally and am also working on incorporating full Apple Silicon into [Coqui TTS](https://github.com/coqui-ai/TTS) soon as well, first as a stand-alone system, but then as an extension alternative for Elevellabs and Silero. Getting sidetracked with Coqui delayed ny progress on the 1.5 oobabooga effort. However, I should have a complete set of modifications for Coqui to support Apple Silicon GPU acceleration and I plan to create a pull request for Coqui, so they may integrate my changes. Honestly, their code seems very well written and it was very easy to read and comprehend. They did a great job with it.

As always, please leave comments, suggestions and issues you find so I can make sure they are addressed. Testers, developers and volunteers are also welcome to help out.  Please let me know if you would like to help out.

## 30 Jul 2023 - Patched and wWrking

I forked the last workng oobagooba/text-generation-webui I knew of that worked with macOS. I had to make some change=s to its code so it would process most of the model using Apple Silicon M1/M2 GPU. I am working on adding some of the new features of the latest oobabooga release, and have found further areas for optimization with Apple Silicon and macOS. I am working as fast as I can to get it upgraded as GGML encoded models are working quite well in my release. I have found some issues with object references in Python being corrupted and causing some processing to fall back to CPU. This is likely a problem for CUDA users due to the extensive use of global variables in the core oobabooga code. It's taking quite a bit of effort to decouple things, but after I do some of that, performance should improve even more.  Once I have that done, I want to incorporate RoPE, SuprtHOT 8K context windows, and new Llama2 support. the last item shouldn’t be terribly difficult since it's built into the GGML libraries which hare part of llama.cpp.

If you are interested in trying out the macOS patched version, please grab it from here: [text-generation-webui-macos](https://github.com/unixwzrd/text-generation-webui-macos)

I hope to have an update out within the week. Again, anyone who want to test, provide feedback, comments, or ideas, let me know or use the "Discussions", at the top of the GitHub page and add to the discussion or start a new one. Let's help the personal AI on Apple Silicon and macOS grow together.

## 28 Jul 2023 - More Testers (QA)

I've had a fe more people contact me with issues and that's a good thing because it shows me theer is an interest in what I am trying to do here and that people are actually trying my procedures out and having decent success.

I want to start getting more features into the fork I created like Llama2 support. If I can do that, the next ting I will likely do is start looking at some of the performance enhancements I have thought of as well as trying to fix a couple of UI/UX annoyances and a scripted installation and...

If anyone would like to help out, please let me know.

## 27 Jul 2023 - More llama.cpp Testing

Earlier problems with the new llama-cpp-python worked out. Seems setting **--n-gpu-layers** to very big numbers is not good anymore. It will result in overallocation of the context's memory pool and this error:

ggml_new_tensor_impl: not enough space in the context's memory pool (needed 19731968, available 16777216)
Segmentation fault: 11

An easy way to see how many layers a model uses is to turn on verbose mode and look for this in the output of STDERR:

**llama_model_load_internal: n_layer    = 60**

It's right near the start of the output when loading the model. Apparently the huge numbers above the number of layers is not best to, "*Set this to 1000000000 to offload all layers to the GPU.*" breaks the context's memory pool. I haven't figured out the proper high setting for this, but you can get the number easily enough by loading your model and looking for the **n_layer** line, then unload the model and put that into n-gpu-layers in the Models tab. BE sure to set it so it's save for the nest time you load the same model.

The output of STDERR is also a good place to validate if your GPU is actually being accessed if you see lines with **ggml_metal_init** at the start of them. It doesn't necessarily mean it's being used, only that llamacpp sees it and is loading supported code for it. Unload the model and then load it again with the new settings.

Someone gets a HUGE thank you for being the first person to give feedback and help me make things better! They actually went through my instructions and gave me some feedback, spotted a few typos and found things to be useful.  You know who you are! 👍

Someone else also asked if this would work for Intel, I tried, but the python which comes with Conda is compiled for i386, which should work(?) but doesn't and should be x86_64.  Might work for Intel macOS, but would be difficult when you try getting Conda to install PyTorch, that won't work well. I'm sure I could hack it to make it work, but that would be a nasty hack. Not only that, I was trying to run things on a 32GB MacBook Pro and having memory issues, I doubt many Intel Macs out there have much more than 32GB and even though they have unified memory, my bet is they would still be slow. I gave up when I figured out that Conda wouldn't install on my 16GB Intel MacBook Pro. Never thought I'd need tat much RAM, but initially I was going to get 64GB and then swapped my 36GB Apple Silicon MBP for 96BG. 😮

If anyone is interested in helping out with this effort, please let me know.  I'm in the oobaboga Discord #mac-setup channel a good bit, or you may reach me through GitHub.

## 25 Jul 2023 - macOS version patched and working

I managed to get the code back together from an unwanted pull of future commits, I had things mis-configured on my side. The patches are applied and it just needs some testing.  So far I have only really briefly tested with a llama 30B 4bit quantized model and I am getting very reasonable response times, though there it is running a range of 1-12 tokens per second. It seemed like more yesterday, but it's still reasonable.

I have not tested much more than a basic llama which was 4 bit quantized. I will try to test more today and tomorrow.

If anyone else is interested in testing and validating what works and what doesn't, please let me know.

## 25 Jul 2023 - Wrong Commit Point

I merged with one commit too far ahead when I created the created the dev-ms branch with a merge back to the oobabooga main branch. I'll need a bit of time to sort the code out.  Until then, I don't know of a working version around. I'll have to sort through my local repository and see if I have something I can create a new repository with or revert to a previous commit.

I'll update the status on my repository and here when I get it sorted out.

## 24 Jul 2023 - macOS Broken with oobabooga Llama2 support

The new oobabooga does not support macOS anymore.  I am removing the fork I was working on because there are code changeds speciffically for Windows and Linux which are not installed on macOS, so the default repository is now the one I generated a pull request for to fix things so Apple Silicon M1 and M2 machines would use GPU's.  It's going to get it sorted out, but I will do it as soon as I can.  Here's teh command to clone the repository and if you have any problems with it, let me know.

```bash
git clone https://github.com/unixwzrd/text-generation-webui-macos.git
```

## 24 July 2023 - LLaMa Python Package Bumped

New Python llama-cpp-python out. Need to be installed before loading running the new version of oobabooga with Llama2 support.

Same command top update as yesterday, it will grab llama-cpp-python.0.1.77.

I'm trying things out now here.

## 23 Jul 2023 - LLaMA support in llama-cpp-python

Ok, a big week for LLaMa users, increased context size roiling out with RoPE and LLaMA 2.  I think I have a new recipe which works for getting the llama-cpp-python package working with MPS/Metal support on Apple Silicon.  I will go into it in more detail in another document, but wanted to get this out to as many as possible, as soon as possible.  It seems to work and I am getting reasonable response times, though some hallucinating. CAn't be sure where the hallucinations are coming from, my hyperparameter settings, or incompatibilities in various submodule versions which will take a bit of time to catch up. Here's how to update llama-cpp-python quickly. I will go into more detail later.

### Installing from PyPi

```bash
# Take a checkpoint of your venv, incase you have to roll back.
conda create --clone ${CONDA_DEFAULT_ENV} -n new-llama-cpp
conda activate new-llama-cpp
pip uninstall -y llama-cpp-python
CMAKE_ARGS="--fresh -DLLAMA_METAL=ON -DLLAMA_OPENBLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" \
    FORCE_CMAKE=1 \
    pip install --no-cache --no-binary :all: --upgrade --compile llama-cpp-python
```

The --fresh in the CMAKE_FLAGS is not really necessary, but won't affect anything unless you decide to download the llama-cpp-python repository, build, and install from source.  That's bleeding edge, but if you want to do that you also need to use this git command line and update your local package source directory of just create a new one with teh git clone. The BLAS setting changed and only apply if you've built and installed OpenBlAS yourself. Instructions are in my two guides mentioned above.

### Installing from source

```bash
conda create --clone ${CONDA_DEFAULT_ENV} --n new-llama-cpp
conda activate new-llama-cpp
git clone --recurse-submodules https://github.com/abetlen/llama-cpp-python.git
pip uninsatll -y llama-cpp-python
cd llama-cpp-python
CMAKE_ARGS="--fresh -DLLAMA_METAL=ON -DLLAMA_OPENBLAS=ON -DLLAMA_BLAS_VENDOR=OpenBLAS" \
    FORCE_CMAKE=1 \
    pip install --no-cache --no-binary :all: --upgrade --compile -e .
```

**NOTE** when you run this you will need to make sure whatever application is using this is specifying number of GPU or GPU layers greater than zero, it should be at least one for teh GGML library to allocate space in the Apple Silicon M1 or M2 GPU space.

## 23 Jul 2023 - Things are in a state of flux for Llamas

It seems that there have been many updates the past few days to account for handling the LLaMa 2 release and the software is so new, not all the bugs are out yet. In the past three days, I have updated my llama-cpp-python module about 3 times and now I'm on release 0.1.74. I'm not sure when things will stabilize, but right before the flurry of LLaMa updates, I saw much improved performance on language models using the modules and packages installed using my procedures here.  My token generation was up to a fairly consistent 6 tokens/sec with good response time for inference. I'm going to see how this new llama-cpp-python works and then turn my attention elsewhere until the dust settles.

I submitted a couple of changes to oobabooga/text-generation-webui, but not sure when those changes will be pushed out. I will probably fork a copy of the repository and path it here, making it available until my changes are incorporated into the main branch for general availability. I should, hopefully have that a little later today, as long as git cooperates with me. I will be the first to admit I am not great with Git, so learning VSCode and using Git have been kinda rough on me as I come from a very non-Windows environment and have used many other version control systems, but never used Git much. I will probably get the hang of it soon and finish making the transition from using vi in a terminal window to a GUI development environment like VSCode.  At least it has a Vim module to plugin, now if they can get "focus follows mouse" to work within a window for the different frames, I'll be very happy.

## 20 Jul 2023 - Rebuilt things *Again* because many modules were updated

Many modules were bumped in version and some support was added for the new LLaMa 2 models.  I don't seem to have everything working, but did identify one application issue which will increase performance fro MPS, if not for Cuda.

The two TTS modules use the same Global model variable in them, so model gets clobbered if you use them. I've submitted a pull request for this. [Dev ms #3232](https://github.com/oobabooga/text-generation-webui/pull/3232) and filed a bug report [Use of global variable model in ElevenLabs and Silero extensions clobbers application global model](https://github.com/oobabooga/text-generation-webui/issues/3234). This was my first time submitting a pull request and submitting a bug report, took a long time to actually figure out how to do it, but maybe there is an easier way than what I did.  Anyway, with this fix, macOS users with M1/M2 processors should see a vast performance improvement if you are using either of these TTS extensions.

## 19 Jul 2023 - New information on building llama-cpp-python

Instructions have been updated.  Also, there were some corrections as I was rushed getting this done.  If you find any errors are think or a better way to do things, let me know.

## 19 Jul 2023 - NEW llama-cpp-python

Haven't tested it yet, but here's how to update yours.  Will change this with the results of my testing.

```bash
CMAKE_ARGS="-DLLAMA_METAL=on -DLLAMA_OPENBLAS=on -DLLAMA_BLAS_VENDOR=OpenBLAS" \
    FORCE_CMAKE=1 \
    pip install --no-cache --no-binary :all: --force-reinstall --upgrade --compile llama-cpp-python
```
