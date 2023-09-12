# Use the GPU on your Apple Silicon Mac

This stared out as a guide to getting oobabooga working with Apple Silicon better, but has turned out to contain now useful information regarding how to get numerical analysis, data science, and AI core software running to take advantage of the Apple Silicon M1 and M2 processor technologies. There is information in the guides for installing OpenBLAS, LAPACK, Pandas, NumPy, PyTorch/Torch and llama-cpp-python. I will probably create a new repository for all things Apple Silicon in the interest of getting maximum performance out of the M1 and M2 architecture.

## You probably want this: [Building Apple Silicon Support for oobabooga text-generation-webui](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS-Install.md)

## If you hate standing in line at the bank: [oobabooga macOS Apple Silicon Quick Start for the Impatient](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS_Apple_Silicon_QuickStart.md)

In the test-scripts directory, there are some random Python scripts using tensors to test things like data types for MPS and other compute engines.  Nothing special, just hacked together in a few minutes for checking GPU utilization and AutoCast Data Typing. There's now a script which does some simple timing of matrix manipulation so you may compare different BLAS and LAPACK libraries.

Anyone wishing to provide any additional information or assistance, pleas feel free.  If you are interested in working on this with me, please let me know as well. It's still only myself and a few volunteers assisting me at the moment.

## 12 Sep 2023 Dependency Hell... Again

For some reason the distribution of PyTorch, llama-cpp-python, and something in the oobabooga requirements.txt, all conflict wirh each other, including installing over an existing NumPy install or even downgrading it from 1.25.3 to 1.24.0. The PyTorch distribution at pytorch also had some conflicting dependencies with other things as well, so you can't just do a --force-reinstall/install with things, and get the daily build of PyTorch because something has change in the distribution. I did manage to fins a combination which works solves teh problems ad posted it in the #macOS-setup of the oobabooga Discord server. To save a bit, here's what I wrote:


> `RuntimeError: MPS does not support cumsum op with int64 input.` This has something to do with PyTorch and their Metal support on macOS using the Accelerate Framework for the M1/M2 GPU acceleration, in particular the version of NumPy and associated "numpy-base" package which is distributed with PyTorch is also problematic. Some time ago, this problem was solved, but seems to have returned. There are several issues with the install process you followed and most of it is due to ever changing libraries, modules and dependencies. Not only all that, but it does not seem to use the Apple Silicon M1/M2 GPU acceleration, so simply installing things in the proper order doesn't particularly help either.
> 
> You could probably repair the modules and libraries in your Python installation, but the quickest and probably the easiest way will be to create a new VENV and install things in this order:
> - Create ne VENV and make it active.
> - `pip install --upgrade --no-deps --force-reinstall --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cpu`
> - `pip install -r requirements_nocuda.txt`
> - (suggested) `NPY_BLAS_ORDER='accelerate' NPY_LAPACK_ORDER='accelerate' CMAKE_ARGS='-DLLAMA_METAL=on' FORCE_CMAKE=1 pip install --force-reinstall --no-cache --no-binary :all: --compile llama-cpp-python`
>
> All of these things may be done at the command line and in the directory you installed the webui.  Hopefully I have them in the correct order and haven't forgotten anything, and I did check this a couple of times.
>
> The torch install will install the one from the nightly build of PyTorch. Though if you are re-installing/upgrading torch from a prior install, you probably want to do it like this as well using the --no-deps option, though no guarantee that you don't need to reinstall everything which comes along with PyTorch. (including it's NumPy as versions other than 1.25.2 will produce an error)
> 
> The install of requirements_nocuda.txt will ***downgrade*** your NumPy version to 1.24.0 if you have 1.25.2 installed so you may want to re-install or upgrade your NumPy version anyway. Upgrades of NumPy will complain that you have a incompatible versions of torch, torchvision and torchaudio, all of which I think can be safely ignored as those packages are not aware of the new development builds of PyTorch. So the warning may be safely ignored, even though it's in nice red text.
> 
>The last, (optional) instruction will get you llama-cpp-python so you can use GGUF models with M1/M2 GPU acceleration on Apple Silicon. It will also build a new NumPy which is one of the requirements for llama-cpp-python. The accelerated NumPy will speed up anything like PyTorch or other modules which use NumPy fro matrix manipulation. A word of caution is that NumPy team had this initially in NumPy, but dropped it in version 1.20 or 1.21, I forget which, due to inconsistent results.  However, while it fails some of its regression tests, this is typical of NumPy to fail a couple, but I haven't had a chance to look into the exact failures. NumPy pre-built fails some regression tests as well, and it looks like it may be the same regression tests.

My latest oobabooga-macOS was going to be a merge of the tagged release of oobabooga 1.5, but I have added some basic level of support for Llama2 and now that the GGUF file format is out, I am right now getting many of the new oobabooga features in their current main branch incorporated into mine for macOS and have stopped adding things to the 1.3.1a version in my test tree on GitHub, but GGUF and Llama2 support are in my dev branch and are working just fine, but not all the changes are implemented. If you'd like this my "in dev" version of oobabooga-macOS, you can clone the repository, even better, if you haven't modified any of the distribution files, you may be able to do a fetch, but here's how to clone.

```bash
# cd to directory wheer you want the clone to be placed.
git clone --branch dev https://github.com/unixwzrd/text-generation-webui-macos.git webui-macOS
cd webui-macOS
python server.py --chat
```

## 06 Spe 2023 This thread...

[Exacty this. PR #1728](https://github.com/oobabooga/text-generation-webui/issues/1728#issuecomment-1708455881)

Follow the link to the PR, and I was wrong in my reply, it's been almost two months. Time certainly does fly by...

@101is5 you get a special mention here because this made my day.

## 06 Sep 2023

Some people have reviewed the installation instructions and given me changes. They have been updated. Hopefully you were able to work through it, though comments and suggestions are always welcome. The scripted method for installing is coming soon.  I know I keep saying this, but I am in final testing of it and will let everyone know here when it's complete.

Thanks again to all who have helped out!

## 05 Sep 2023 - Testing and Updates

GPT-4 was offline a good portion of the week last week, and I have my Internet connection upgraded. The upgrade to my connection was so I could download more than 1.2TB per month. The good folks at Comcast/Xfinity were kind enough to add an additional $25 to my monthly bill for this. I've never heard of a cap like this and it was amazing that about 20 years ago in Japan, I could get Fibre to MY HOUSE for about $70/month with 100MB speed up and down. Toss in an extra $10 and you could get 1GB up and down (Yes, that's Giga).  That was way more than mu WiFi router could handle, but the machines directly connected, it was AMAZING! There were about 5 companies all competing for business, wchih was good for the consumer and drove prices down. With Comcast/Xfinity, I pay more and get worse service, they are the only provider I can get where I live, so they have me over the proverbial barrel.

I've updated the instructions quite a bit and I've been given some feedback from testing and walkthroughs of the instructions. I haven't had much time to get things running and play with the new llama.cpp, but it is quite fast and looks like it has many new features, including being able to convert just about any model into the new GGUF format.

Still working on the scripted VENV builds. Hope to have them up this week, but that';s the hope every week. I also and working through the GitHub Workflows to determine what needs to be done to create pre-built binaries and installs for the llama.cpp,python wheels to support oobabooga and macOS. Ive never created GitHub workflows before or wheels for Python, it may take a bit to get this going, but doesn't look like too much to get done.

It's been a very busy couple of weeks for me as things change quickly every day.  Please feel free to comment in the discussions, and as always thanks goes out to the oobabooga team.

Living the dream of high speed Internet and unlimited data.

## 29 Aug 2023 - GGML -> GGUF

* **NOTICE GGML File Format Change to GGUF**

The new llama.cpp is quite fast now and seems to take advantage of MPS now nicely. But in order to use the latest, any GGML files will need to be converted to GGUF files. It's quite simple to do. The script to do the conversion is in the llama.cpp repo and there is only one requirement is to import gguf into your Python installation.

[pip install gguf](https://github.com/ggerganov/llama.cpp/tree/master/gguf-py)

then run the script:

[convert-llama-ggmlv3-to-gguf.py](https://github.com/ggerganov/llama.cpp/blob/master/convert-llama-ggmlv3-to-gguf.py)

There are other conversion scripts in the [llama.cpp](https://github.com/ggerganov/llama.cpp) for converting other formats. There is a notice regarding the latest repository to support GGML and that GGUF will be given a priority for now. This is great news for everyone who wants to use models.

Last llama.cpp commit to support GGML: [master-ef3f333](https://github.com/ggerganov/llama.cpp/releases/tag/master-ef3f333)

This is the PR Discussing the new GGUF format: [PR: #2398](https://github.com/ggerganov/llama.cpp/pull/2398)

## 29 Aug 2023 - Updated Instructions

Added and changed a few things in the instructions. Updated the dev and main repositories with new requirements. A few other items, not really significant. Continuing to test performance and for any issues or problems.

There does seem to be an inconsistency with one of my llama builds with torch, and I'm tracking it down now, but the one which is the base packages before rebuilding anything is due to the SciPy support that gets loaded with the oobabooga requirements, they took a different approach to their builds and combining their own Numpy embedded with the SciPy package somehow, or at least that's what it appears to be to me.  I will continue to investigate and have an update soon.

## 28 Aug 2023 - Performance Improvements And LLaMa2 works.

* UPDATED QUICK INSTALL. PLEASE NOTE REGARDING GGML FILES, 0.1.78 llama-cpp-python must be used with GGML files. Both that version and the latest will work with LLaMa2.

Have spent much time looking at Python packages for numerical analysis, data analytics and AI. There are many different combinations of libraries, depending on which order you install them and whether you compile in the Accelerate Framework. I believe it is working for the most part, but I haven't tested it completely. However, PyTorch now fully supports Apple Silicon, but other Python modules dance on some of the libraries installed by NumPy and PyTorch, specifically the BLAS libraries. I have a configuration tool I was putting together in Bash, but the logic ran me into a wall regarding graph traversal, so I am planning to have that re-done in Python.

I will be putting up the Bash script, likely in its own repository, since other VENV build tools will be added and I plan to add benchmarking and regression testing to the process. There is value in it now as it can help simplify your builds for your VENVs and create consistent builds every time and in a sequence you configure. It also contains a portion for setting up/updating your personal Conda. Through this process I have discovered a lot about the state of Linear Algebra packages available, who uses them and what configurations they support.

Presently, there are no BLAS packages which directly support Apple Silicon which I am able to find.  If someone does discover one, I will add it to the mix. The build configurations I've checked are OpenBLAS, BLIS, Apple's Accelerate Framework, and the various libraries included with Python packages. There is a lot here and I will have more on that, hopefully soon.

The library which llama.cpp uses, formerly GGML, is now GGUF, and will require any GGML models to be converted to GGUF format. This process is fairly quick, but involves installing a Python package from source, though it may be available soon on PyPi or other location.  I will write that up as soon as I have time. This new version seems to be very fast, relative to things from the past, especially on Apple Silicon, making use of the Unified Memory and M1/M2 GPU. I am updating my "dev" version of oobabooga, specifically for macOS, and it is now available for download and will run with LLaMa2. It will even load the 70B LLaMa2 model on a MacBook Pro with 96GB Unified Memory.

I know it's been a little while since I had anything to write, but there's a lot to report.  I am planning on more updates later today and over the next few days. Please feel free to take advantage of the "Discussions" here on my GitHub if you like or you can reach out to me as I am usually in the oobabooga Discord in the #mac-setup channel.

Thank you to all who have helped support me in working on this project, your kind assistance is very much appreciated, of all varieties.

## 18 Aug 2023 - Current Status of Testing and Configuring

While most of this may seem like a moving target, I have discovered inconsistencies in how packages will overlay each other's dynamic linked libraries  and will also not completely uninstall themselves using either Pip or Conda. I have searched most everywhere I can find information about linear algebra and matrix manipulation. It was way more than I ever really wanted to, but it's given me more understanding of how things work inside language models and given me a deeper appreciation for understanding less about how they store and retrieve information the way they do. I completely understand the theory of how they are seeking a minima in the matrix space, but exactly how things get coded in their matrix and how they can retrieve things, still amazes me. I suppose those who say they are simply statistical models which are good at predicting the next token in sequence, but even considering that, it opens up a whole can of worms, technically and even philosophically. Maybe everything is deterministic and it's tru we have no free will, only the illusion of it. Anyway, that's an entirely different discussion.

I am texting my VENV build, benchmarking, and regression test automation.  I have run through the process manually for a few iterations, but it quickly become error-prone to continue manually. I have automated all aspects of the VENV build down to the BLAS libraries I'll be using.  I know I keep saying it will be just another few days, but I want to make sure everything is tested and I have a flexible enough build and test framework constructed so if new conditions arise, the framework is still usable or rather re-usable when new situations arise.

I also updated the information in the build process regarding some of my findings about package managers which only support my view that they can cause more problems than they solve if they are not carefully coordinated between various packages and package build teams. Sometimes common areas are subject to pollution.

## 14 Aug 2023 New Direction Forward

So far it's taken a bit more than a week and I've been in contact with a few others to tests do of our ideas on optimizing the various Python modules/packages on Apple Silicon and here my progress so far:

### My progress so far

- Put development and work on next release of oobabooga for macOS on hold until performance issues are investigated. This means merging oobagooba code for their 1.5 release into my codebase is on hold along with any changes I am putting into the code until there was an explanation or understanding of the performance issues.
- Test a full build of the environment using modules which are required by the oobabooga/text-generation-webui.
- Collecting some basic timing metrics along the way as the environment is built up.
- Look for changes in those metrics in order to identify possible issues.
- Compare GGML models running in obabooga and native llama.cpp to see if there is any difference between them.
- Gather as much information from as many sources regarding performance and issues arising from using the M1/M2 GPU.

### Some things I've learned along the way

- NumPy rejected using the Apple Accelerate Framework, stating it was giving inaccurate results during testing, so in their 1.20.1 release, they deprecated its use.
- There are numerous BLAS and LAPACK packages, including BLIS, which I have tested, though primarily to see if they were using the GPU for processing and so far I cannot see any using it.
- Apple's Accelerate Framework is "kinda" built into many of the numerical analysis packages, but it is difficult to see if they actually support it or not.
- NumPy who officially state they do not approve use of Accelerate, include it in their build, if you build it in a particular manner. I discovered this by accident after doing a recompile without any BLAS/LAPACK libraries in a searchable location for it to link with.  It linked with the Accelerate framework.
- While things seem to compile with the Accelerate Framework, it is unclear to me where the libraries are I am linking to exist. Running otool on my final linked package reveals that I am linking to a location where symbolic links exist for the libraries, but they are all broken.

My conclusions so far is there are performance issues related to inefficient processing is,  there are problems, they need more investigation, there is much contradictory information floating around, there is more testing to be done. These issues should not hold up further release of code into the "dev" tree for my branch of oobabooga. I intend to do this very soon, in the next day or two. After that, it will be available for download by anyone to test and report any issues related to running on macOS. I will go back to looking for performance issues again in the various modules.

My reasons for releasing is to allow people to use and test some of the new features like increased context length, and llama2 support mainly. This seems to make sense because the issues will still be with us in the near future, and holding up any software release doesn't change that. Further, even though NumPy doesn't "officially" support the Accelerate Framework, the latest releases of PyTorch and I assume SciPy as well both support Apple Silicon and both of these sit on top of NumPy, so, if they certify things work on Apple Silicon, then I can only assume they are happy with the NumPy ability to work with it too.

I'll continue testing and benchmarking things and will try to get some real numbers produced and presented soon. No matter what, keep watching this spot for my latest updates on this issue.

## 11 Aug 2023 - This Kind of Explains the Issue With pip, conda, etc al.

Well, I haven't tried the latest main branch of oobagooba as I'm still on a working 1.3.1 I have in my repository.  I'm sorting some performance and library compatibility issues out now, but hope to be back to getting a 1.5 release which is tested and running on macOS using Metal.  Metal also happens to be the piece I'm looking into deeply because there seem yo be issues about it using GPU or CPU or both, I have just about got a test framework setup for different combinations of things like NumPy, Pandas, PyTorch, and will test them in various configurations.

One item I discovered is depending on what was installed when and whether you have BLAS and LAPACK libraries, you can get distinctly different  results, so I'll also run the regression tests for NymPy especially.

There are two options for matrix manipulation, use GGML (in llama.cpp) or use LAPACK and BLAS, all available from different places. I have not found a BLAS/LAPACK which runs on the Apple Silicon GPU, which is why GGML exists, as far as I can read. The only thing which does run on the M1/M2 GPU is the BLAS and LAPACK Apple includes with their Accelerate Framework.  NumPy does not recommend using this as it gives inconsistent results past version 1.20.1 of NumPy, however, I'm trying the newer versions of the libraries because the speedup is about 5-7x.

There are so many funny interdependencies in the libraries and library managers between pip and conda, sometimes they leave bits of what they installed after you uninstall them.  This only makes matters worse, because if you are using NumPy, you could be using a library from just about anywhere.

I'm also writing the Apple Dev Team a letter letting them know I am not particularly thrilled with their response to problems with Accelerate being basically the same as the NumPy, PyTorch and SciPy people give is "Oh, it's not out problem, contact the vendor (Apple)"  When I look at Apple's site, their response is, "Oh, it's not our problem, contact the developer (Open Source Team)."  Both groups need to talk with each other otherwise nothing will get done. and  Apple should be much more supportive of the OpenSource Community or they will loose out to Nvidia and Hugging Face.

## 10 Aug 2023 - Have Something Interesting

Wanted to give an update. Have found an issue with re-installing and installing new modules and packages. All seems to be linked back to BLAS and LAPACK.  it seems the installation order affects how the BLAS and LAPACK libraries are handles, even when not doing a recompile of the module or package. I'm able to reproduce my results, but haven't found any definite answer to the performance issues encountered using oobabooga. Many people with many different opinions on the cause, but I haven't seen a real solution yet.

I've been combing through anything I can find, but it's all very limited in content. Apple doesn't seem to be talking about how to use their high performance architecture and I even found in the release notes for NumPy that they had issues a couple of releases ago regarding inconsistent results from the Accelerate Framework, advising anyone who had a problem with this to contact Apple. Not cool on either party's part in pointing fingers. From what I can tell, no one from the NumPy team or Apple has actually tried to sit down and resolve the issues together, though this is speculation gathered from what I have read and been able to find on the Internet, which is sparse.

I have other conjectures as well, that the whole reason that llama-cpp-python exists if due to this dependency issue and it comes with its own required linear algebra support library. Probably due to what I discovered that packages which need this will bring it along and drop it in the Python library, but also don't uninstall it when they are installed, so there's a libblas sitting in the lib directory of your venv's "root" hierarchy.

After looking at the release notes for NumPy, I figured it would be a good idea to get the source for NumPy and try the regression tests from the various configurations I find being installed. I even just today found a new one which gets configured and not really mentioned anywhere I can recall,of "openblas64_" which was pulled from my /usr/local/lib build I did of OpenBLAS. To muddy things further, depending on which libraries and where they come from, there are differences in the  results of regression testing. Even when using the stock standard NumPy install. All needs tracking down which I am working on doing as quickly as possible.

Not alone in this, several have approached me willing to assist in testing in with various configurations. So far, we replicated my results, but still not sure how to proceed. Bottom line is, I am hopeful regarding using the Accelerate Framework, but I must do more testing on the combinations, ordering and dependencies of the Python modules and packages. I'll keep updating here with my progress. More to come...

## 07 Aug 2023 - Numpy Accelerated with Apple Silicon

I Spent a good portion of today and yesterday evening rebuilding my machine learning and data analytics Python packages required for oobabooga support, the information is also helpful for anyone who uses Apple Silicon Macs for these purposes.

Need to look into this some more.  Don't want to lead anyone down a blind alley, but I think I have found a significant performance solution for Python Data Analytics and AI packages. I'll need to collect more information, but I have repeatable tests which show dramatic increases and performance with NumPy and very reduced CPU utilization. More to come...

## 07 Aug 2023 - Optimizing The Environment, Silver Lining in macOS  13.5

I'm have to rebuild my venv because I accidentally created inconsistencies in my Python packages, and this seems like a good a point as any to try to optimize the packages. In doing this I found something interesting when getting NumPy installed. Looking for a faster alternative to OpenBLAS or a way to have use the Apple Silicon GPU, I discovered BLIS, a faster linear algebra library.  To compare I put together a quick Python script t benchmark different builds of NumPy but noticed  when Pip does a recompile of NumPy, it actually looks for and finds the Apple [accelerate Framework](https://developer.apple.com/accelerate/) which leverages their superscalar GPU and Neural engine technology for processing vectors and tensors. I assume this is due in part by the recent upgrade to macOS 13.5 and the NumPy team putting it into their package when you compile at install time using Pip.

More info to come as I try to optimize these packages which are used for AI, but also Data Science and Analytics. Keep an eye out here and you will see updated information posted in this repository regarding package optimization soon.

## 06 Aug 2023 - Performance issues and Python Packages

**THIS IS IMPORTANT**
Whatever terminal package you use to access the command line, make absolutely sure the "Open using Rosetta" checkbox is unchecked in the "Get Info" pop-up.  Someone reported an issue with the pip installation instructions, and I was able to replicate the issue I believe because I had set iTerm2 to open in Rosetta a while back to do some testing and forgot to change it back. I was in the process of rebuilding my venvs have to rebuild a couple of them because running in Rosetta can produce inconsistent results, especially for things like builds. Also, if your Python packages and libraries are installed in this manner, they will perform slower because the Intel code has to be translated to run.

Another thing, NOT DO the macOS 13.5 update, it is broken in many ways. WebKit crashes every now and then, Private Relay breaks things like accessing OpenAI and more, there are issues with the overall UI as well.

## 03 Aug 2023 - Coming Soon oobabooga 1.5 integration and Coqui for macOS

Currently I am finishing up changes to the release 1.5 oobabooga and integrating them into my fork and may or may not have additional performance improvements I have identified for Apple Silicon M1/M2 GPU acceleration.  I hope by the end of this week.  I will release it in my test fork I created to allow people to test and provide feedback.

I also got distracted earlier in the week by looking for another TTS alternative to Elevenlabs which runs locally and am also working on incorporating full Apple Silicon into [Coqui TTS](https://github.com/coqui-ai/TTS) soon as well, first as a stand-alone system, but then as an extension alternative for Elevenlabs and Silero. Getting sidetracked with Coqui delayed ny progress on the 1.5 oobabooga effort. However, I should have a complete set of modifications for Coqui to support Apple Silicon GPU acceleration and I plan to create a pull request for Coqui, so they may integrate my changes. Honestly, their code seems very well written and it was very easy to read and comprehend. They did a great job with it.

As always, please leave comments, suggestions and issues you find so I can make sure they are addressed. Testers, developers and volunteers are also welcome to help out.  Please let me know if you would like to help out.

## 30 Jul 2023 - Patched and Working

I forked the last working oobagooba/text-generation-webui I knew of that worked with macOS. I had to make some change=s to its code so it would process most of the model using Apple Silicon M1/M2 GPU. I am working on adding some of the new features of the latest oobabooga release, and have found further areas for optimization with Apple Silicon and macOS. I am working as fast as I can to get it upgraded as GGML encoded models are working quite well in my release. I have found some issues with object references in Python being corrupted and causing some processing to fall back to CPU. This is likely a problem for CUDA users due to the extensive use of global variables in the core oobabooga code. It's taking quite a bit of effort to decouple things, but after I do some of that, performance should improve even more.  Once I have that done, I want to incorporate RoPE, SupertHOT 8K context windows, and new Llama2 support. the last item shouldn’t be terribly difficult since it's built into the GGML libraries which hare part of llama.cpp.

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

The new oobabooga does not support macOS anymore.  I am removing the fork I was working on because there are code changed specifically for Windows and Linux which are not installed on macOS, so the default repository is now the one I generated a pull request for to fix things so Apple Silicon M1 and M2 machines would use GPU's.  It's going to get it sorted out, but I will do it as soon as I can.  Here's the command to clone the repository and if you have any problems with it, let me know.

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
