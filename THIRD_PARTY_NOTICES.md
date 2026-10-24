# Dependency provenance and reuse boundary

This file lists the third-party code this project builds on, adapts or references, with the license obligations that apply. The original code in this repository is MIT licensed (`LICENSE`). Most runtime patches modify the pinned SuperMarioWonderRecomp runtime, and a few carry adapted MIT or public-domain code; their full notices are reproduced below and next to the affected patches. `dependencies.lock` records exact source revisions, license-document hashes and source archive hashes actually retrieved. Candidate pins are not a claim of a successfully resolved or built toolchain.

## Original reference code

- [SuperMarioWonderRecomp LICENSE](https://github.com/BelmanteGu/SuperMarioWonderRecomp/blob/010ed7a596a621c15d3a8dfa5795ca8c3668b066/LICENSE): MIT, copyright (c) 2026 BelmanteGu. Primary reference: original `recomp/`, `runtime/`, selected synthetic `tests/`, importer and shader harness tooling. Preserve the full MIT copyright, permission and warranty notice with any substantial copied source.
- [LASRecomp LICENSE](https://github.com/peiikai/LASRecomp/blob/b7903f0dcde0732014b8aada56d0640fe4adef54/LICENSE): MIT, copyright (c) 2026 Peiikai. Comparison only. License explicitly excludes Nintendo property and third-party components. A later revision, `09f8e5ebf28aeefd6daa1456c73ae1b7a51efec5`, was also inspected for comparison only (see `runtime/patches/astral-fs-commit.NOTICE` and `docs/ASTRAL_FIBER_CONTEXT.md`); no LASRecomp source from either revision is copied into this repository except through the letsgo-recomp adaptation noted below. Its runtime and recompiler are reusable original implementations, but title-specific addresses/patches and generator outputs are not a generic Astral runtime.

These licenses grant rights in the authors' implementation, not the games. NSO images, lifted C/C++, shader code/control blocks, SPIR-V/MSL derived from game shaders, RomFS, fonts from firmware, dumps and captures must stay private/ignored. Do not copy entire reference repositories into tracked source: Wonder includes media and game-specific directories; LAS includes fonts, randomizer material and third-party trees with separate obligations. No claim of a full provenance audit of every upstream file is made.

## Selected critical dependencies

Full immutable license URLs and SHA-256 evidence are in `dependencies.lock`; license texts were read from those pins or the hash-verified source distributions.

The current macOS native Metal runtime uses Apple's Metal APIs and SPIRV-Cross to translate SPIR-V to Metal Shading Language. It has no Vulkan or MoltenVK runtime or build dependency. The Vulkan-Headers, Vulkan-Loader and MoltenVK pins below remain as historical source-provenance references for earlier or other Vulkan-based workflows; they are not active dependencies of this native Metal route. This dependency record does not establish gameplay acceptance.

| Component | Selected pin | License evidence / obligation |
|---|---|---|
| SDL3 | release-3.2.24 | `LICENSE.txt`, zlib. Retain notice, mark altered source and do not misrepresent origin. Matches the Wonder build guide's version. |
| Vulkan-Headers | v1.4.357 | Historical source pin, not used by the native Metal runtime. `LICENSE.md`: mostly Apache-2.0 OR MIT; `parse_dependency.py` exception. Preserve per-file notices; do not label the entire tree uniformly dual-licensed. |
| Vulkan-Loader | v1.4.357 | Historical source pin, not used by the native Metal runtime. `LICENSE.txt`: Apache-2.0 majority, more permissive per-file exceptions. Retain license, notices and applicable attribution when distributing. |
| MoltenVK | v1.3.0 | Historical candidate pin, not used by the native Metal runtime. `LICENSE`, Apache-2.0. Its `ExternalRevisions` audit remains pending for any future Vulkan-based use. |
| SPIRV-Cross | `300681bbcb8365a8d1eacdacdbdd0249bbf1d6b1` | Native Metal runtime SPIR-V-to-MSL compiler. Root `LICENSE` is Apache-2.0; the complete text and SHA-256 evidence are recorded below and in `dependencies.lock`. |
| Ryubing shader translator | a82350bb774f70fcbd41c9987bf67a3775409963 | `LICENSE.txt`, MIT, Ryujinx Team and Contributors. Full revision resolves the short hash in Wonder's translator guide. Keep MIT notice for translator and modified copies; transitive package audit still pending. |
| Capstone Python/source | 5.0.6 | Source archive `LICENSE.TXT` BSD-3-Clause (COSEINC); `src/LICENSE_LLVM.TXT` NCSA/University of Illinois notice. Both retained on redistribution. |
| python-lz4 | 4.4.4 | Archive `LICENSE` BSD-3-Clause (Steeve Morin). Bundled LZ4 has separate BSD-2-Clause notices; per-file audit remains before vendoring/distribution. |
| python-zstandard | 0.24.0 | Archive `LICENSE` BSD-3-Clause (Gregory Szorc), `zstd/LICENSE` BSD-3-Clause (Meta), `zstd/COPYING` GPLv2 alternative. Use the available BSD path and retain notices. |

Python package versions above are explicit bootstrap candidates, not versions locked by Wonder itself. No global tools were installed. CMake, Ninja, .NET SDK and SPIRV-Tools remain tool preparation work; their binaries and transitives are not covered by this source lock. Fonts/fonttools are unnecessary for the first smoke.

LAS-only dependencies are deliberately not adopted: its README lists SDL3, dynamically linked LGPL FFmpeg, MIT ImGui/VMA/SMAA/FSR, OFL fonts, and GPL-bearing separate generator/randomizer helpers (notably Unicorn/Keystone/oead/evfl). Its root MIT license does not relicense these. Choosing that route later requires auditing exact helper/package sources and distribution obligations, not copying its release bundle.

## Storage decision

Keep the pinned reference checkouts and dependency evidence under ignored `.local/references/`. Later vendor only reviewed original files from an explicit allowlist, with their complete notices, or build against the ignored pinned source with small reviewed patches. Git submodules cannot filter content: sparse checkout is a local convenience, not a way to make a whole upstream submodule content-safe. No submodule or wholesale vendor import is approved by this document.

## Full upstream MIT notices

Most `runtime/patches/*.patch` files modify or carry context from the
SuperMarioWonderRecomp runtime, and `tools/astral_spirv_harness/` patches its
offline SPIR-V harness. That harness, and `tools/prepare_shader_tools.py`, use
the Ryubing (Ryujinx) shader translator. Neither upstream tree is vendored; the
full notices below apply to the adapted portions and to any distributed build.

### SuperMarioWonderRecomp LICENSE

Revision `010ed7a596a621c15d3a8dfa5795ca8c3668b066`, `LICENSE` SHA-256
`6466066caf0162b4af89a3108a778843fa21f98dc8a0bc254fe08ededd04faa3`.

```text
MIT License

Copyright (c) 2026 BelmanteGu

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### Ryubing (Ryujinx) LICENSE.txt

Revision `a82350bb774f70fcbd41c9987bf67a3775409963`, `LICENSE.txt` SHA-256
`781bbdf040b7d0286c47cca6bffdb9148dfe751c0c9cabdf1a1752412a2e56b4`.

```text
MIT License

Copyright (c) Ryujinx Team and Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

## HWOPUS-IMPL-075 adaptation

`runtime/patches/astral-hwopus.patch` adds a service adapted from
[letsgo-recomp runtime/services/hwopus.c](https://github.com/raulzrrs/letsgo-recomp/blob/884f6d78035e537653c36579178cb0bb0c0b4f1d/runtime/services/hwopus.c),
revision `884f6d78035e537653c36579178cb0bb0c0b4f1d`, source SHA-256
`f6941c6805cbce32ac3575f56a1945e19f6a45f535d9b9bc80c8b9881208154d`.
Its manager/decoder lifecycle, big-endian packet envelope and interleaved signed
16-bit PCM semantics are adapted to `nx::Service` and validated `IpcCtx` buffers.
The original FFmpeg backend is replaced by existing host libopus, including real
size/init/reset/decode APIs; this is not a new codec. Permissive success patterns,
guessed work sizes and truncating PCM behavior are not retained. Full applicable
MIT notices are reproduced both here and in the added translation unit.

### letsgo-recomp LICENSE

```text
MIT License

Copyright (c) 2026 raulzito12345rrs
Portions copyright (c) 2026 Peiikai (LAS Recomp, see LICENSE-LASRecomp)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

This license covers the source code in this repository only. It grants nothing
for Pokémon: Let's Go, Eevee!, which belongs to Nintendo / The Pokémon Company /
Game Freak and must be supplied by each user from their own copy.
```

### LICENSE-LASRecomp

```text
MIT License

Copyright (c) 2026 Peiikai

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

This license covers LAS Recomp's own code. It does not cover The Legend of Zelda: Link's Awakening or any other
Nintendo property, nor the third-party components in third_party/, which keep their own licenses.
```

### Host libopus

Opt-in system dependency only; no libopus source or binary is vendored. Native
validation uses installed libopus 1.6.1, dynamically linked. CMake requires an
explicit `NX_HWOPUS=ON` and pkg-config `opus>=1.3`; this minimum expresses API
availability, not tested equivalence of all versions or platforms. Exact local
header/library hashes are retained in private evidence. Redistribution of a
linked binary requires the dependency notice and any applicable patent-license
obligations. The installed 1.6.1 COPYING text follows. FFmpeg is not linked by
this overlay.

```text
Copyright 2001-2023 Xiph.Org, Skype Limited, Octasic,
                    Jean-Marc Valin, Timothy B. Terriberry,
                    CSIRO, Gregory Maxwell, Mark Borgerding,
                    Erik de Castro Lopo, Mozilla, Amazon

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions
are met:

- Redistributions of source code must retain the above copyright
notice, this list of conditions and the following disclaimer.

- Redistributions in binary form must reproduce the above copyright
notice, this list of conditions and the following disclaimer in the
documentation and/or other materials provided with the distribution.

- Neither the name of Internet Society, IETF or IETF Trust, nor the
names of specific contributors, may be used to endorse or promote
products derived from this software without specific prior written
permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
``AS IS'' AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER
OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Opus is subject to the royalty-free patent licenses which are
specified at:

Xiph.Org Foundation:
https://datatracker.ietf.org/ipr/1524/

Microsoft Corporation:
https://datatracker.ietf.org/ipr/1914/

Broadcom Corporation:
https://datatracker.ietf.org/ipr/1526/
```

## Performance HUD font

`runtime/patches/astral-perf-hud.patch` adds `runtime/src/font8x8_basic.h`: the printable ASCII rows of
[font8x8_basic.h](https://github.com/dhepper/font8x8/blob/8e279d2d864e79128e96188a6b9526cfa3fbfef9/font8x8_basic.h)
by Daniel Hepper, revision `8e279d2d864e79128e96188a6b9526cfa3fbfef9`, source SHA-256
`49d8df366296b203ca3211bc0672cf2a762135bf12710735b6292756b19dffd5`. Public domain (after Marcel Sondaar's font8x8
and the IBM public domain VGA fonts); attribution kept in `runtime/patches/astral-perf-hud.NOTICE`.

## FSR 1 upscaling (AMD FidelityFX)

`runtime/patches/astral-upscale.patch` (NX_UPSCALE=fsr, the launcher's Upscaling setting) switches on the pinned
runtime's own FSR 1 present passes: `tools/shaders/fsr/easu.frag` and `rcas.frag` of SuperMarioWonderRecomp (MIT,
above), a GLSL port after AMD FidelityFX Super Resolution 1.0, Copyright (c) 2021 Advanced Micro Devices, Inc., MIT.
The patch adds no shader code; the full AMD MIT notice is in `runtime/patches/astral-upscale.NOTICE`.

## Launcher libraries, font and art

`runtime/patches/astral-launcher.patch` adds two unmodified single-header libraries by Sean Barrett and contributors
from [nothings/stb](https://github.com/nothings/stb) (master as fetched on 2026-10-19):

- `runtime/src/third_party/stb_truetype.h` v1.26, SHA-256
  `ecd30b05e0dd4fea3a13c26810dd9e1992dc379049482c393d5a19e6b5090aab`
- `runtime/src/third_party/stb_image.h` v2.30, SHA-256
  `594c2fe35d49488b4382dbfaec8f98366defca819d916ac95becf3e75f4200b3`

Both are dual-licensed by their authors: public domain (unlicense.org) or the MIT License, Copyright (c) 2017 Sean
Barrett. The full text of both alternatives is at the end of each header, which travels unmodified inside the patch.
This project uses the public domain alternative.

`assets/launcher/` (loaded at run time, outside the patch):

- `Baloo2-Bold.ttf`, SHA-256 `1dca71e24f8ce978987109d92031ab6fd5f7f54399139193a8f93478f7e957b2`: Baloo 2, Copyright
  2019 The Baloo 2 Project Authors (https://github.com/EkType/Baloo2), SIL Open Font License 1.1. It is a modified
  version of [google/fonts](https://github.com/google/fonts) `ofl/baloo2/Baloo2[wght].ttf`, instanced at wght=700
  and subset to Latin with fontTools. No Reserved Font Name is declared. The license is next to it in
  `assets/launcher/OFL.txt` (SHA-256 `ad09b05dc8bc678c9daf7c4c5f7ef1f55e5726127f4330b2e98e40b9dffcb860`).
- `title-bg.jpg`, SHA-256 `685911155273eb58be3f8beaea38ace7e0430edfabee6b31f4fc9b2448ba0833`: original AI-generated
  artwork made for this project (an empty neon city at night, approved by the owner). It contains no
  game assets, characters or logos, and was resized to 1600x900 and JPEG-compressed.
- `start_screen.jpg`, SHA-256 `2aff6a2492e0da3bad65fe93e4aa915a7e7a890a94bce9b9408ad7bbf28d112e`: a screenshot of the original launcher over the art above; no game content.
- `AC-Recomp.png`, SHA-256 `d76b8ce1cffd5a10c3f1401916dce6ec518815d09262aa16e9af5f3116bea13c`: the app icon, made from the background art above with the "AC" lettering in Baloo 2; the build turns it into the macOS .icns, Windows .ico and Linux icons.

Attribution is also kept in `runtime/patches/astral-launcher.NOTICE`.

## Replacement font (downloaded at build time)

When a player gives no firmware fonts, `tools/acr_build.py` downloads Noto Sans Regular
(`fonts/NotoSans/hinted/ttf/NotoSans-Regular.ttf` from
[notofonts/notofonts.github.io](https://github.com/notofonts/notofonts.github.io) at commit
`28b15b4b43b7bed62b5cf6e6b0b5ff5846270535`, SHA-256
`478c558ea716033cd60c03438f628dfa75694dcf6b5f6d505a2f05fd2b4f3823`) and its license (`fonts/LICENSE`, SHA-256
`f2095b08bed08b23a6fe26112fcd679a2bee3f002eef077eb05d215ed1051bd8`). Copyright 2022 The Noto Project Authors, SIL
Open Font License 1.1. It is not redistributed by this repository or its releases; the build checks both checksums,
installs the font only on the player's machine and copies the license next to it.

## Optional local Simplified Chinese font profile

The local Chinese profile uses AOSP `DroidSansFallback.ttf` from
`platform/frameworks/base` revision `4a5ee7b3096bba4b6a2eb3f01edc9a99681caeb7`
under Apache-2.0, with its complete `data/fonts/NOTICE` retained beside the local
font files. Types 1 and 2 use this static TrueType font; the other four types
retain the existing Noto Sans replacement. The font binary stays under ignored
`.local/`, and its hash and license evidence are recorded in `dependencies.lock`.
Canonical shared-font filenames name interface slots, not Nintendo font content.

The local language preference is `language.txt` containing `zh-Hans`; a user's
explicit `NX_LANGUAGE` continues to override it. This does not alter the runtime's
unconfigured language default.

## SPIRV-Cross Apache-2.0 license

The native Metal runtime uses SPIRV-Cross revision `300681bbcb8365a8d1eacdacdbdd0249bbf1d6b1` to translate SPIR-V shaders to Metal Shading Language. The source archive SHA-256 is `ca390d0cfb115504b97af1120c3a9479fb7b8f9e1a590fc80dca52ec1dd1c489`; the checked root `LICENSE` SHA-256 is `cfc7749b96f63bd31c3c42b5c471bf756814053e847c10f3eb003417bc523d30`. Preserve source-file copyright notices and this license with redistributions. This provenance entry does not claim gameplay acceptance.

```text

                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [yyyy] [name of copyright owner]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
```
