# Native Metal scaling

`astral-native-metal.patch` supersedes the older Vulkan FSR present path for this macOS Astral target. It uses native Metal textures and encoders directly; no exported Vulkan objects or MoltenVK bridge is involved.

The default build renders internally at 1920×1080. At presentation, the actual drawable pixel dimensions are queried, and a centered aspect-fit rectangle is calculated. MetalFX Spatial encodes into the same Metal command buffer at that rectangle's dimensions, followed by a native Metal composite into the drawable. A non-16:9 screen retains bars. There is no fixed 4K output.

`NX_UPSCALE=metalfx` (default) selects MetalFX Spatial. `bilinear` selects native Metal bilinear compositing. This changes the scaling quality, not the graphics API. Legacy `fsr`, `fsr1`, and `1` values migrate to MetalFX. MetalFX support or allocation failure is explicit, without silently substituting another backend. At or below the source dimensions there is no upscale pass.

The launcher's window-size values are logical points. Retina drawable pixels can be larger; they do not change the internal game resolution. `--render-res` changes the lifted internal resolution at build time. The current default is 1920x1080.

On Apple M5, actual native execution logged MetalFX Spatial 1920×1080 → 2560×1440, then 2940×1654 after fullscreen sizing. This establishes scaler execution, not full-game rendering correctness or sustained performance. See [native Metal status](ASTRAL_NATIVE_METAL.md).
