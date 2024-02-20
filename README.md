# LFX-mlx

## Environment
![image](https://hackmd.io/_uploads/BkDF1ul2T.png)


## Part One: Framework Execution:
### Install MLX framework
Follow the instructions [here](https://github.com/ml-explore/mlx?tab=readme-ov-file#installation) to install MLX
```bash
pip3 install mlx
```

### Generating text with Llama2
Follow the instructions [here](https://github.com/ml-explore/mlx-examples/tree/main/llms/llama#llama) to setup Llama2 model. I used the converted [model](https://huggingface.co/mlx-community/Llama-2-7b-mlx) from the MLX Community organization so the conversion step was not necessary. Besides, I also installed [git lfs](https://github.com/git-lfs/git-lfs) manually for downloading the model.

```bash
git clone https://github.com/ml-explore/mlx-examples.git
cd mlx-examples/llms/llama
pip3 install -r requirements.txt
git lfs install
git clone https://huggingface.co/mlx-community/Llama-2-7b-mlx
python3 llama.py --prompt "hello, I'm Alice" --model-path Llama-2-7b-mlx
```

### Result
![image](https://hackmd.io/_uploads/BJnEX_gnT.png)

## Part Two: llama.cpp and chatbot/API sever
Follow the instructions [here](https://wasmedge.org/docs/contribute/source/os/macos) and [here](https://wasmedge.org/docs/contribute/source/plugin/wasi_nn/#build-wasmedge-with-wasi-nn-llamacpp-backend) to build and install WasmEdge and its plug-in

### Get source code for WasmEdge
```bash
git clone https://github.com/WasmEdge/WasmEdge.git
git checkout origin/hydai/0.13.5_ggml_lts
cd WasmEdge
```

### Install required dependencies
```bash
brew install cmake ninja llvm
export LLVM_DIR="$(brew --prefix)/opt/llvm/lib/cmake"
export CC=clang
export CXX=clang++
```

### Build and install WASI-NN llama.cpp Backend
```bash
cmake -GNinja -Bbuild -DCMAKE_BUILD_TYPE=Release \
  -DWASMEDGE_PLUGIN_WASI_NN_BACKEND="GGML" \
  -DWASMEDGE_PLUGIN_WASI_NN_GGML_LLAMA_METAL=ON \
  -DWASMEDGE_PLUGIN_WASI_NN_GGML_LLAMA_BLAS=OFF \
  .
cmake --build build
# For the WASI-NN plugin, you should install this project.
cmake --install build
```

### Chatbot Example
Follow the instructions [here](https://github.com/second-state/WasmEdge-WASINN-examples/tree/master/wasmedge-ggml#llama-example-for-wasi-nn-with-ggml-backend) to execute the chat example

- Build and install WASI-NN ggml plugin
```bash
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- --plugin wasi_nn-ggml

source $HOME/.zshenv
```

- Get the model and the source code
```bash
# Get model
curl -LO https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF/resolve/main/llama-2-7b-chat.Q5_K_M.gguf
# Get source code
git clone https://github.com/second-state/WasmEdge-WASINN-examples.git
```

- Execute chatbot
```bash
cp WasmEdge-WASINN-examples/wasmedge-ggml/llama-stream/wasmedge-ggml-llama.wasm ./

# Execute
wasmedge --dir .:. \
  --nn-preload default:GGML:AUTO:llama-2-7b-chat.Q5_K_M.gguf \
  wasmedge-ggml-llama.wasm default
```

- Result of chatbot

![image](https://hackmd.io/_uploads/rkMLOOgnp.png)

### API Server Example

I followed the instruction [here](https://github.com/second-state/LlamaEdge/tree/main/api-server#create-an-openai-compatible-api-server-for-your-llm) to execute the API server example

- Build and install WASI-NN ggml plugin
```bash
curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- --plugin wasi_nn-ggml

source $HOME/.bashrc
```

- Get the model and the source code
```bash
# Get model
curl -LO https://huggingface.co/TheBloke/Llama-2-7b-Chat-GGUF/resolve/main/llama-2-7b-chat.Q5_K_M.gguf
# Get executable WASM
curl -LO https://github.com/second-state/LlamaEdge/releases/latest/download/llama-api-server.wasm

# For WebUI
curl -LO https://github.com/second-state/chatbot-ui/releases/latest/download/chatbot-ui.tar.gz
tar xzf chatbot-ui.tar.gz
```

- Execute
```
wasmedge --dir .:. --nn-preload default:GGML:AUTO:llama-2-7b-chat.Q5_K_M.gguf llama-api-server.wasm -p llama-2-chat
```

- Test via RESTful
```bash
curl -X POST http://localhost:8080/v1/chat/completions -H 'accept:application/json' -H 'Content-Type: application/json' -d '{"messages":[{"role":"system", "content": "You are a helpful assistant."}, {"role":"user", "content": "Who is Robert Oppenheimer?"}], "model":"llama-2-chat"}'
```

- Result of API server

![image](https://hackmd.io/_uploads/HJUBo_x26.png)
