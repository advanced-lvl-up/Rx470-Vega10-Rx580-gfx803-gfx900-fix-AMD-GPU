unofficial support for rx580, Vega 10, rx470, rx590, rx480, rx570, sp2048, or similar GFX803 & GFX900 using zluda &amp; Windows
-----------------------------------

A forum for testing unsupported hardware. Section I attempts solving various issues with AMD GFX803 & GFX900 and their limited Windows support. Feel free to copy, or improve it. This tensile library is tested for Windows Zluda only (native ROCm should work also, hopefully), Linux version is work in progress.  The provided workflows should be fine for testing any system type. Check in images-workflow folder for examples.

Intro, Super Ultimate Definitive Ultraman Blazar Edition Guide For Unsupported Hardware
-----------------------------------

Firstly, if you have ComfyUI, SDNext or similar already up and running with Zluda, then simply replace the tensile library in ROCm Hip SDK 5.71 with the custom one provided. the default is typically `c:/program files/AMD/ROCM/5.7/bin/rocblas/library` or similar. Also put the rocblas.dll provided into `c:/program files/AMD/ROCM/5.7/bin`  (backup your old files first). That should be all. It should work after zluda recompiles as usual.

Hosted here is a recompilation and potential bug fix for AMD's Tensile library for GFX803 & GFX900 (aka Ellesmere, Polaris, sp2048, Rx470, Rx580, R9 Fiji?, Vega, etc.) making it more stable for Zluda or ROCm generally. I'm not a programmer, many corrections are likely needed.  Some GPUs of the era (Rx580 16GB sp2048) require a mainboard with MMIO / Above 4G mapping. Refer to Section III if your board needs help enabling this option.  Change log:

- Looked at the Tensile 6.24 code, fixed a potential kernel bug that caused crashing, multiple memory leaks
- Tried adding minor optimizations to the GEMM kernels for a major speed boost (in some workloads)
- Added support for fp16 and default datatypes  bf16 &  other general fixes.  Overall progress:  gfx803 is 90% complete.  gfx900 is 25% complete (usable though not tested much)
- Enabled inline assembler for FP32 mode. --force-fp32 and --force-fp16 Comfy options should both work now 
- UniPC Sampler now works.  Compiled optional rocblas.dll support for Hip SDK 6.24 in both 803 and 900 archs
- Added debug kernels with modified code for fp16 accumulation. +8% faster (fp16 only) and helps GPU thermals. For testing only, at 1792x1792 pixels or with video models, you could end up with incoherent prompt following due to renormalization issues

You can learn more here https://github.com/vosen/ZLUDA and here https://github.com/lshqqytiger/ZLUDA
Thank you to the authors.

Important notes on installation:
-----------------------------------
You'll need this https://github.com/lshqqytiger/ZLUDA fork.  If you have HipSDK 5.71 then download `ZLUDA-windows-rocm5-amd64.zip` file from his releases, and follow the usual instructions.  I think 5.71 is the recommended option for legacy archs like 803.

And then as mentioned before,  replace the `rocblas.dll` in `c:/program files/AMD/ROCM/5.7/bin`, with the one from my fix (rocm.gfx800-gfx900-for.hip.sdk.5.7.1-and-6.2.4.7z).  And replace library folder at `c:/program files/AMD/ROCM/5.7/bin/rocblas/library`  (default path, yours may be different)

First time generating using zluda requires 10 to 30 mins of compiling time for emulation. then about 3 minutes compile time for each major change in settings or samplers. Works instantly from then on.  You can set `ZLUDA_COMGR_LOG_LEVEL=1` environment variable if you want an indication of when compiling occurs.

Optional HIP SDK 6.24 support for gfx803 and gfx900:
-----------------------------------
Special instructions for 6.24.  You can now try HipSDK 6.24 for gfx803 and gfx900 also.  After installing 6.24, download both versions of zluda from the link above. Then install `ZLUDA-windows-rocm6-amd64.zip` for SDK6.   Afterwards, replace the `nvcuda.dll` file in your zluda directory with the one from `ZLUDA-windows-rocm5-amd64.zip`.  Then install my rocblas.dll fix as usual into `c:/program files/AMD/ROCM/6.2/bin` along with the library to `c:/program files/AMD/ROCM/6.2/bin/rocblas/library`  This allows legacy archs to use 6.24. (Experimental, you probably wont need this?)

Troubleshooting:  Having only one version of the SDK installed potentially helps avoid conflicts in `HIP_HOME` and `HIP_PATH` system environment variables. Also check that your user and system `path` environment variables only point to `your_chosen_path/AMD/ROCM/6.2/bin`, and not the other version if you've previously installed it.

Recommendations:
----------------
Use (https://github.com/LostRuins/koboldcpp) with Vulkan from lostruins for LLM.  A kobold fork for zluda is ready but seems unecessary until fixed.  FLUX1 Schnell GGUF Quant_3ks is a good test since it can resolve in 1 step at 512x512 res in about 9 seconds, and the bulky t5 encoder is not strictly required. The gfx803 test system at native Flux resolution 768x1280 takes about 21 seconds per step. (portrait & landscape resolutions are slightly faster). Going higher than native res is not recommended.

Disable previews. VAE decoding is still buggy in most popular implementations. Using tiled VAE if you have issues or, better yet using TinyVAE is recommended if the diffusion model supports TinyVAE. For example if VAE decode cuts into system swap memory it can sometimes crash.  This is normal.  Make sure to have a large System Page File (16GB virtual memory & higher) and keep an eye on VRAM usage to be safe, minding GPU thermals as well.

Using FP32 CPU Vae decode is actually a good alternative for testing stability if you have low VRAM or low system RAM. In ComfyUI it looks like this:  example Comfy commandline args: "--force-fp16 --use-quad-cross-attention --disable-cuda --fp16-text-enc --fp32-vae --cpu-vae --disable-smart-memory --novram"  (for CPU VAE decode)

More typically you would use something like "--force-fp16 --use-quad-cross-attention --disable-cuda --bf16-vae"   (for GPU VAE decode, default Comfy memory management).  Using GGUF quantized models can solve many memory related issues as well.  fp16 is faster & should be used in models that support it

Beginners Guide (incomplete, refer to other guides for now)
-----------------------------------

Recommended: AMD Rx470 or higher with 8+ gigabytes VRAM for this guide, (you are here)
Recommended: 12 to 16GB of System DDR3 RAM or better (24 to 32GB of DRAM is preferred if VRAM = 4GB)
Required: Intel 4th gen+ CPU or AMD equivalent, (mainboard with PCIe 3.0 Atomics support required for GFX803, Yes?)

1. Install Windows 10 or 11, debloat, disable logging / Microsoft telemetry bloatware. Install ROCM 5.71 SDK, Python 3.10.6 or 3.11, and other required dependencies.

2. Get the latest graphics driver for Polaris/Vega. Compute workloads cause excess heat. Underclock and undervolt the card using Wattman App. Run some simple games or Vulkan apps to test stability.

3. Install ComfyUI or something similar.  Work in Progress, I'll write a detailed guide for beginners in the future. For now just use any known working AMD Fork of whichever UI you prefer.

4. Install Zluda for ROCm 5.7, and make the necessary changes in your activated python VENV (virtual environment)

5. Work in progress. Most guides on running Zluda are quite confusing. A simplified guide is planned for later updates

-----------------------------------
Tensile and ROCm are open source alternatives created by AMD, refer to the related Github repository for license and limitations.
-----------------------------------

