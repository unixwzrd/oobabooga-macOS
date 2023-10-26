# Just Some Scripts

These are some scripts to stress test the GPU with meaningless tensors. Can be used with CUDA. MPS. or CPU computing engines.  This will let you see if your GPU is actually being used, and I'm sure of someone would like they could use them to determine the capacity of their GPU.

Enjoy.


The only one which really gives any information is this:

There are now several scripts which give timing and profiling information for comparing different VENV's configurations, bit in setup and config and with some timing and profiling. For now, these are for NumPy.

numpybench  - displays numpy config and times tests using time.
numpyprof   - displays numpy config and profiles tests using cProf.
numpytime   - displays numpy config and times tests using timit.


Environment variable NO_TEST is set in order to just get configuration and bypass the testing which may be time-consuming.

    export NO_TEST=1    # Turn off testong 

    unset NO_TEST       # Turn on testing (Default)