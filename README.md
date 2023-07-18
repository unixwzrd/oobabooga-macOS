# Use the GPU on your Apple Silicon Mac

## You probably want this: [Building Apple Silicon Support for oobabooga text-generation-webui](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS-Install.md)

## If you hate standing in line at the bank: [oobabooga macOS Apple Silicon Quick Start for the Impatient](https://github.com/unixwzrd/oobabooga-macOS/blob/main/macOS_Apple_Silicon_QuickStart.md)

In the test-scripts directory, there are some random Python scripts using tensors to test things like data types for MPS and other compute engines.  Nothing special, just ahcked together in a few minutes for checking GPU utilization and AutoCast Data Typing.

## 19 Jul 2023 - NEW llama-cpp-python

Haven't tested it yet, but here's hwo to update yours.  Will change this with th eresults of my testing.

```bash
CMAKE_ARGS="-DLLAMA_METAL=on" FORCE_CMAKE=1 pip install --upgrade --no-cache --no-binary :all: --compile llama-cpp-python
```
