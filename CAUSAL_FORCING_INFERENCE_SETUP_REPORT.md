# Causal Forcing Inference Setup Report

Setup was performed on `torralba-3090-2.csail.mit.edu` under:

```bash
/data/scratch-fast/kwen1/Causal-Forcing
```

The goal was to follow the README as closely as possible while keeping all installs under the user's scratch storage and avoiding source compilation of `flash-attn`.

## Environment Created

Because `conda`, `mamba`, and `micromamba` were not initially on `PATH`, and `/usr/bin/python3` lacked `ensurepip` / `python3-venv`, a local micromamba install was created under scratch:

```bash
/data/scratch-fast/kwen1/micromamba
```

The README-style environment name was kept:

```bash
/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing
```

Environment creation command:

```bash
/data/scratch-fast/kwen1/micromamba/bin/micromamba create \
  -y \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  python=3.10 \
  pip
```

For future use, prefer direct paths or putting the env first on `PATH`:

```bash
export PATH=/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing/bin:$PATH
export HF_HOME=/data/scratch-fast/kwen1/.cache/huggingface
export PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip
```

Direct use of the env Python avoids `micromamba run` touching AFS lock/cache state:

```bash
/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing/bin/python inference.py ...
```

## Package Installation

Installed PyTorch first so the later `flash-attn` wheel could match the Torch/CUDA ABI:

```bash
PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip \
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  pip install torch==2.8.0 torchvision==0.23.0 \
  --index-url https://download.pytorch.org/whl/cu128
```

Then installed the repository requirements:

```bash
PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip \
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  pip install -r requirements.txt
```

Installed OpenAI CLIP from the README's upstream Git source:

```bash
PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip \
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  pip install git+https://github.com/openai/CLIP.git
```

Installed `flash-attn` from a prebuilt wheel, not from source. Torch reported CUDA `12.8` and `_GLIBCXX_USE_CXX11_ABI=True`, so the matching wheel used was:

```bash
PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip \
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  pip install 'flash-attn @ https://huggingface.co/strangertoolshf/flash_attention_2_wheelhouse/resolve/main/wheelhouse-flash_attn-2.8.3/linux_x86_64/torch2.8/cu12/abiTRUE/cp310/flash_attn-2.8.3+cu12torch2.8cxx11abiTRUE-cp310-cp310-linux_x86_64.whl'
```

Installed the repo in editable/develop mode as in the README:

```bash
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  python setup.py develop
```

Installed Hugging Face fast-transfer support:

```bash
PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip \
/data/scratch-fast/kwen1/micromamba/bin/micromamba run \
  -r /data/scratch-fast/kwen1/micromamba/root \
  -n causal_forcing \
  pip install hf_transfer
```

The Hugging Face token was supplied via `HF_TOKEN` during downloads. The raw token is intentionally not written here.

## README Alignment / Deviations

The README's main path was:

```bash
conda create -n causal_forcing python=3.10 -y
conda activate causal_forcing
pip install -r requirements.txt
pip install git+https://github.com/openai/CLIP.git
pip install flash-attn --no-build-isolation
python setup.py develop
```

What changed:

- Used local micromamba instead of conda because conda/mamba were not on `PATH` and system Python could not create a venv.
- Installed PyTorch explicitly before `requirements.txt` so the environment would have a known CUDA/Torch target for `flash-attn`.
- Installed `flash-attn` from a matching prebuilt wheel instead of running `pip install flash-attn --no-build-isolation`, to avoid compiling CUDA code on the cluster node.
- Used scratch-backed caches for pip/Hugging Face where possible.

The README's checkpoint download list was followed for:

- `Wan-AI/Wan2.1-T2V-1.3B`
- `Wan-AI/Wan2.1-T2V-14B`
- `zhuhz22/Causal-Forcing` chunkwise/framewise Causal Forcing checkpoints
- `zhuhz22/Causal-Forcing` Causal Forcing++ framewise 1-step/2-step checkpoints

## Key Installed Versions

Verified with the env Python:

```text
Python:          3.10.20
torch:           2.8.0+cu128
torch CUDA:      12.8
torchvision:     0.23.0+cu128
numpy:           1.24.4
flash-attn:      2.8.3
transformers:    5.12.1
diffusers:       0.31.0
huggingface_hub: 1.21.0
accelerate:      1.14.0
```

Validation:

```text
pip check: No broken requirements found.
```

Core imports passed for `torch`, `torchvision`, `numpy`, `flash_attn`, `clip`, `diffusers`, `transformers`, `accelerate`, `omegaconf`, `imageio`, `cv2`, and the repo's `CausalInferencePipeline`.

## Downloaded Assets

Downloads used scratch cache/location settings:

```bash
HF_HOME=/data/scratch-fast/kwen1/.cache/huggingface
HF_HUB_ENABLE_HF_TRANSFER=1
HF_XET_HIGH_PERFORMANCE=1
HF_TOKEN=<redacted>
```

Downloaded README/model assets:

```bash
hf download Wan-AI/Wan2.1-T2V-1.3B --local-dir wan_models/Wan2.1-T2V-1.3B
hf download Wan-AI/Wan2.1-T2V-14B --local-dir wan_models/Wan2.1-T2V-14B
hf download zhuhz22/Causal-Forcing chunkwise/causal_forcing.pt --local-dir checkpoints
hf download zhuhz22/Causal-Forcing framewise/causal_forcing.pt --local-dir checkpoints
hf download zhuhz22/Causal-Forcing 'causal-forcing++/framewise-2step.pt' --local-dir checkpoints
hf download zhuhz22/Causal-Forcing 'causal-forcing++/framewise-1step.pt' --local-dir checkpoints
```

Current sizes:

```text
/data/scratch-fast/kwen1/micromamba                  6.0G
/data/scratch-fast/kwen1/.cache/huggingface          344K
/data/scratch-fast/kwen1/.cache/pip                  1.6G
wan_models/Wan2.1-T2V-1.3B                           17G
wan_models/Wan2.1-T2V-14B                            65G
checkpoints                                          22G
```

Note: the inference path tested loads the 1.3B Wan model assets. The config field `real_name: Wan2.1-T2V-14B` did not appear to be used by the tested inference pipeline, but the 14B base was downloaded anyway to stay close to the README.

More specifically:

- `utils/wan_wrapper.py` hardcodes the text encoder weights to `wan_models/Wan2.1-T2V-1.3B/models_t5_umt5-xxl-enc-bf16.pth`.
- `utils/wan_wrapper.py` hardcodes the VAE weights to `wan_models/Wan2.1-T2V-1.3B/Wan2.1_VAE.pth`.
- `CausalInferencePipeline` constructs `WanDiffusionWrapper(**config.model_kwargs, is_causal=True)`.
- `WanDiffusionWrapper` defaults to `model_name="Wan2.1-T2V-1.3B"`.
- `configs/causal_forcing_dmd_framewise_2step.yaml` only supplies `model_kwargs.timestep_shift: 5.0`, not `model_kwargs.model_name`.
- `real_name: Wan2.1-T2V-14B` is used by the training-side `BaseModel` score-model setup, not by the tested few-step inference path.

## GPU / Node Observations

The setup ran on:

```text
torralba-3090-2.csail.mit.edu
```

During setup, `nvidia-smi` showed seven NVIDIA GeForce RTX 3090 GPUs, each with about 24 GiB VRAM:

```text
GPU indices: 0, 1, 2, 3, 4, 5, 6
GPU model:   NVIDIA GeForce RTX 3090
VRAM:        24576 MiB reported by nvidia-smi
```

Observed occupancy during setup:

```text
Most GPUs were idle at roughly 1 MiB used.
GPU 4 had another user's process at about 10 GiB early in setup.
Later, GPU 5 had another Python process using about 4.3 GiB; it was not from this setup.
```

The smoke tests targeted `CUDA_VISIBLE_DEVICES=0`. GPU 0 returned to idle afterward.

One current caveat: a later `nvidia-smi` check from this shell failed with:

```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver.
```

Follow-up diagnostics from the later shell showed:

```text
hostname: torralba-3090-2.csail.mit.edu
HOME: /afs/csail.mit.edu/u/k/kwen1
SLURM_JOB_ID: 1058220
/dev/nvidia*: absent
/proc/driver/nvidia/version: NVIDIA kernel module 575.57.08 present
torch.cuda.is_available(): False
torch.cuda.device_count(): 0
squeue -u kwen1: failed to create Slurm stream socket from this execution context
```

Interpretation: the later shell could see the NVIDIA kernel module but not GPU device files, so GPU devices were not exposed to that process context. If this recurs, verify the Slurm allocation/session and device exposure before launching inference. The earlier inference tests did run successfully on GPU 0.

## Smoke Tests Run

### CPU checkpoint load

Loaded the framewise 2-step Causal Forcing++ checkpoint with the pipeline on CPU for checkpoint compatibility:

```text
KV inference with 1 frames per block
load result: <All keys matched successfully>
```

Checkpoint tested:

```bash
checkpoints/causal-forcing++/framewise-2step.pt
```

### One-frame GPU smoke inference

Ran a one-prompt, one-frame smoke test on GPU 0:

```bash
CUDA_VISIBLE_DEVICES=0 \
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
HF_HOME=/data/scratch-fast/kwen1/.cache/huggingface \
/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing/bin/python inference.py \
  --config_path configs/causal_forcing_dmd_framewise_2step.yaml \
  --output_folder /tmp/causal_forcing_smoke_output \
  --checkpoint_path 'checkpoints/causal-forcing++/framewise-2step.pt' \
  --data_path /tmp/causal_forcing_smoke_prompt.txt \
  --num_output_frames 1 \
  --use_ema
```

Result:

```text
Completed successfully.
Produced an MP4 under /tmp/causal_forcing_smoke_output.
```

### 21-frame GPU smoke inference

Ran a default 21-frame smoke test on GPU 0 with thread caps:

```bash
CUDA_VISIBLE_DEVICES=0 \
OMP_NUM_THREADS=4 \
TORCHINDUCTOR_COMPILE_THREADS=1 \
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
HF_HOME=/data/scratch-fast/kwen1/.cache/huggingface \
/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing/bin/python inference.py \
  --config_path configs/causal_forcing_dmd_framewise_2step.yaml \
  --output_folder /tmp/causal_forcing_smoke_output_21f \
  --checkpoint_path 'checkpoints/causal-forcing++/framewise-2step.pt' \
  --data_path /tmp/causal_forcing_smoke_prompt.txt \
  --use_ema
```

Observed result:

```text
Free VRAM before run: about 23.3 GiB
Peak observed GPU 0 usage during active generation: about 9.3 GiB
Generation progress: 21/21 frames at about 1.82 it/s
End-to-end prompt iteration: about 21.66 s
Output size currently on disk: 4,164,622 bytes
Output metadata observed: H.264 MP4, 832x480, 16 fps, about 5.06 s
```

Output path:

```bash
/tmp/causal_forcing_smoke_output_21f/A cinematic shot of a quiet mountain lake at sunrise, with soft mist over the water and detailed ref.mp4
```

## Recommended Inference Command

For a normal demo run, use the env Python directly:

```bash
export HF_HOME=/data/scratch-fast/kwen1/.cache/huggingface
export PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip

CUDA_VISIBLE_DEVICES=0 \
OMP_NUM_THREADS=4 \
TORCHINDUCTOR_COMPILE_THREADS=1 \
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True \
/data/scratch-fast/kwen1/micromamba/root/envs/causal_forcing/bin/python inference.py \
  --config_path configs/causal_forcing_dmd_framewise_2step.yaml \
  --output_folder output/framewise_2step_cf++ \
  --checkpoint_path 'checkpoints/causal-forcing++/framewise-2step.pt' \
  --data_path prompts/demos.txt \
  --use_ema
```

Check GPU availability first:

```bash
nvidia-smi
```

## Issues / Future Notes

- Do not install into `/usr/bin/python3`; it is system-owned and did not have `ensurepip` available.
- Keep envs and caches in `/data/scratch-fast/kwen1`, not AFS.
- `micromamba run` may try to use `/afs/csail.mit.edu/u/k/kwen1/.cache/mamba/proc/proc.lock`; direct env binaries avoid this.
- If `pip` warns about `/afs/csail.mit.edu/u/k/kwen1/.cache/pip`, set `PIP_CACHE_DIR=/data/scratch-fast/kwen1/.cache/pip`.
- The third-party `flash-attn` wheelhouse avoided CUDA source compilation. If Torch is upgraded, reinstall a matching `flash-attn` wheel.
- The 14B Wan model is large and was not exercised in smoke inference. Expect it to be impractical on a single 24 GiB RTX 3090 unless the code path uses offload/quantization or model parallelism.
- The tested Causal Forcing++ framewise 2-step path worked on one 3090 for a 21-frame prompt.
- `/tmp` smoke outputs are node/session-local; durable outputs should go under the repo or another scratch path.
