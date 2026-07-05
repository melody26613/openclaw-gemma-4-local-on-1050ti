### Build llama.cpp Docker Image

1. Clone repo
```bash
git clone https://github.com/ggml-org/llama.cpp.git llama.cpp.20260703

cd llama.cpp.20260703
```

2. Check CUDA architecture

```bash
docker pull pytorch/pytorch:2.6.0-cuda12.4-cudnn9-devel
docker run --gpus all -it pytorch/pytorch:2.6.0-cuda12.4-cudnn9-devel bash
python3 -c "import torch; print(torch.cuda.get_device_capability())"
	(6, 1)
exit
```

3. modify the cuda version to match the host's version

Set cuda version to 12.3.2 & ubuntu 22.04

```diff
diff --git a/.devops/cuda.Dockerfile b/.devops/cuda.Dockerfile
index 91c3088c7..cd463234e 100644
--- a/.devops/cuda.Dockerfile
+++ b/.devops/cuda.Dockerfile
@@ -1,7 +1,7 @@
-ARG UBUNTU_VERSION=24.04
+ARG UBUNTU_VERSION=22.04
 # This needs to generally match the container host's environment.
-ARG CUDA_VERSION=12.8.1
-ARG GCC_VERSION=14
+ARG CUDA_VERSION=12.3.2
+ARG GCC_VERSION=12
 # Target the CUDA build image
 ARG BASE_CUDA_DEV_CONTAINER=nvidia/cuda:${CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION}
```

4. Build docker image

```bash
docker build --build-arg CUDA_DOCKER_ARCH=61 -t local/llama.cpp:server-cuda-12.3-20260703 --target server -f .devops/cuda.Dockerfile .
```

### Gemma4 E2B

```bash
mkdir -p unsloth/gemma-4-E4B-it-GGUF

nohup hf download unsloth/gemma-4-E2B-it-GGUF --include "gemma-4-E2B-it-UD-Q4_K_XL.gguf" --local-dir unsloth/gemma-4-E2B-it-GGUF &

nohup hf download unsloth/gemma-4-E2B-it-GGUF --include "MTP/*" --local-dir unsloth/gemma-4-E2B-it-GGUF &


# gemma-4-E2B-it-GGUF Q4
# no vision, KV cache f16, batch 2048/512
# 6.48 t/s (1. ask hello, 2. introduce algorithm)
# hello: ok
# weather: ok
# stock: ok
# typhoon: ok
docker run -d \
  --name llama-cpp-gemma4-e2b-q4 \
  --gpus all \
  -e CUDA_VISIBLE_DEVICES="0" \
  -e TURBO_AUTO_ASYMMETRIC=0 \
  --ulimit memlock=-1:-1 \
  --shm-size=16g \
  -v /mnt/d/workspace/models:/models \
  -p 11436:11436 \
  local/llama.cpp:server-cuda-12.3-20260703 \
  -m /models/unsloth/gemma-4-E2B-it-GGUF/gemma-4-E2B-it-UD-Q4_K_XL.gguf \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --ctx-size 131072 \
  --batch-size 2048 \
  --ubatch-size 512 \
  --flash-attn on \
  --port 11436 \
  --host 0.0.0.0 \
  --no-mmap \
  --mlock

# gemma-4-E2B-it-GGUF Q4 MTP
# no vision, KV cache f16, batch 2048/512
# 16.93 t/s (1. ask hello, 2. introduce algorithm)
# hello: ok
# weather: ok
# stock: ok
# typhoon: ok
docker run -d \
  --name llama-cpp-gemma4-e2b-q4-mtp \
  --gpus all \
  -e CUDA_VISIBLE_DEVICES="0" \
  -e TURBO_AUTO_ASYMMETRIC=0 \
  --ulimit memlock=-1:-1 \
  --shm-size=16g \
  -v /mnt/d/workspace/models:/models \
  -p 11436:11436 \
  local/llama.cpp:server-cuda-12.3-20260703 \
  -m /models/unsloth/gemma-4-E2B-it-GGUF/gemma-4-E2B-it-UD-Q4_K_XL.gguf \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --ctx-size 131072 \
  --batch-size 2048 \
  --ubatch-size 512 \
  --flash-attn on \
  --port 11436 \
  --host 0.0.0.0 \
  --no-mmap \
  --mlock \
  --model-draft /models/unsloth/gemma-4-E2B-it-GGUF/MTP/gemma-4-E2B-it-Q8_0-MTP.gguf \
  --spec-type draft-mtp --spec-draft-n-max 4

# hardware usage:
# GPU VRAM: 3.0 ~ 3.1 GB
# CPU RAM: 4.7 GB
```

[huggingface unsloth/gemma-4-E2B-it-GGUF](https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF)

[huggingface unsloth/gemma-4-E2B-it-GGUF MTP](https://huggingface.co/unsloth/gemma-4-E2B-it-GGUF/blob/main/MTP/README.md)