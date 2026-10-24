# AC-Recomp

## 项目简介（中文）

**AC-Recomp** 是《异界锁链》(Astral Chain) 1.0.0 在 Apple Silicon macOS 上的实验性静态重编译移植。本分支的 Astral 目标全程使用**原生 Metal 图形后端**（无 Vulkan/MoltenVK），支持 MetalFX Spatial 超分（原生 1920×1080 输入 → 屏幕分辨率）。

本仓库包含的原创工作（详见 `docs/ASTRAL_NATIVE_METAL.md`）：

- NVN→Metal 命令重放后端：零拷贝池别名、bind 时 uniform 快照、counter 快速路径（无流内等待）
- 性能优化系列（同场景 A/B 实测：救援场景 38.6 → 45.85 FPS，+18.9%）：流水线化部分提交、GPU 写区间跟踪的精确 CPU 读取、present 流水线 + 60Hz deadline
- 诊断工具链（环境变量开关）：逐 encoder GPU 计数器采样、按管线绘制直方图、消融剖析器
- Apple 平台内存布局修复（动态/弹性内存补丁）

不包含任何游戏内容：需自备合法导出的本体与主机密钥；更新档（含 1.0.1）不使用；导入内容、生成代码、shader 与构建产物保留在 `.local/` 下不上传。

---

> [!IMPORTANT]
> **This project was built with heavy AI assistance.** Most of the code, tooling, reverse engineering and
> documentation in this repository was written by an AI model (Anthropic's Claude, through Claude Code), directed,
> tested and reviewed by the project's owner. It is shared as is, as a fan project. If you prefer not to use
> AI-written code, you are free to skip it. As with any code from the internet, you build and run it at your own
> discretion.

**AC-Recomp** is an experimental static recompilation of *Astral Chain* 1.0.0 for Apple-silicon macOS. This local branch replaces its graphics backend with native Metal throughout the Astral target.

No game content is included. You supply your own legally dumped base game and console keys. Updates, including 1.0.1, are not used. Keep imported content, generated code, shaders and builds private under `.local/`.

## Current Metal work

- Native Metal buffers, textures, samplers, MSL shader functions, graphics/compute pipelines and command encoders implement the NVN compatibility layer.
- CAMetalLayer presents the picture. The executable does not compile or link the Vulkan/MoltenVK renderer, and has no backend selector. Astral configuration rejects non-Apple hosts.
- Internal resolution defaults to **1920×1080**. **MetalFX Spatial** scales to the drawable's actual pixel size, preserving the game aspect ratio. Window sizes are in points; Retina pixels are separate. Bilinear remains a scaling-quality choice within Metal.
- Both 30 and experimental 60 fps executables are built; the launcher defaults to 60 fps. This is not evidence of sustained 60 fps or correct gameplay timing.
- Existing SDL input/audio, Chinese configuration, save profiles and controller mappings are preserved.

The native Metal work is **not yet accepted as fully playable**. On an Apple M5, native Metal has reached the animated 3D title, Chinese menus, save selection and a loaded FILE 01 character-creation scene. Further gameplay compatibility is being verified. Older upstream gameplay claims used the previous renderer and do not establish native Metal gameplay acceptance. See [native Metal status](docs/ASTRAL_NATIVE_METAL.md).

## Building on macOS

Use Apple silicon, macOS 13+, Xcode command-line tools, Python 3.11+, about 25 GB free, your own base-game NSP and console keys. Do not paste or upload key contents.

```sh
brew install cmake ninja pkg-config sdl3 opus spirv-tools zstd
brew install --cask dotnet-sdk
python3 tools/acr_build.py --nsp /path/to/base-game.nsp --keys /path/to/keys \
  --out /path/to/private-build --jobs 4
```

The builder fetches pinned sources and verifies the SPIRV-Cross archive hash. It imports the game locally, lifts CPU code, translates shaders to offline SPIR-V, composes the runtime patches and builds a native Metal app. SPIR-V is compiled to MSL for the actual GPU API. No Vulkan runtime is involved. See [build details](docs/BUILDING.md) and [MetalFX behavior](docs/ASTRAL_UPSCALE.md).

`runtime/patches/series.macos` is the authoritative overlay sequence against the pinned SuperMarioWonderRecomp source. Runtime changes belong in patches, not generated game code. `dependencies.lock` and [third-party notices](THIRD_PARTY_NOTICES.md) preserve exact source provenance. Older documents and inactive upstream platform code are historical references, not supported alternative backends for this Astral build.

## Limitations

The project is experimental; completion of the whole game, 3D rendering correctness, sustained performance, and 60 fps timing require actual gameplay acceptance. Unknown services and unsupported rendering operations fail explicitly. A successful compile or first frame does not prove compatibility. The locally built app currently depends on its build/data directory and local SDL/opus libraries.

Chinese fonts and user settings are local configuration. Default replacement Latin fonts do not cover Chinese; provide suitable licensed shared fonts through the documented font mechanism. Physical iPhone/iPad, Linux and Windows are outside this native Metal macOS target.

The original project and runtime are MIT licensed. Other components have their own licenses; see [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
