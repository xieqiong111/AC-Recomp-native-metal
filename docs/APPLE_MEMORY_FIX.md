# Apple memory policy — MAC-MEM-FIX-002

Native **default remains blocked**. The explicit **apple-high-v1** profile passed
five fresh native processes, including the actual patched upstream memory
functions. Both original-layout simulator applications passed the shared zeroing
and protection checks: **26.5 tested / 26.0 minimum target**. This is memory-policy
validation, not a runtime/gameplay build, physical-device test, or performance claim.
Independent review is assigned to the parent; no nested review was performed.

## Diagnosis and preserved baseline

On October 18, 2026, macOS 26.6.2 / Darwin 25.6.0 / arm64 / 16 KiB host pages,
the original probe again rejected occupied ranges without replacing mappings.
The final default process had these actual VM regions:

| Range (half-open) | Current / maximum protection | VM tag |
|---|---|---|
| `0x51A000000..0x81AC00000` | none / none | 0 |
| `0x81AC00000..0x81B000000` | RW / RWX | 2 (`VM_MEMORY_MALLOC_SMALL`) |
| `0x81B000000..0xB1A000000` | none / none | 0 |

Heap, alias, stack and kernel reservations failed in that process; modules and
optional thunks succeeded. Earlier processes had different randomized addresses.
Self-inspection records exact VM bounds, permissions and extended-info tags;
`vmmap` on a separate task-owned held probe confirmed `MALLOC_SMALL` mappings in
the occupied area. The inaccessible tag-0 ranges are adjacent to malloc-small
mappings, but their tag does **not** establish their owner. Do not attribute them
to nano malloc or xzone malloc solely from these observations.

A fresh original-range process with only `MallocNanoZone=0` still failed alias,
stack and kernel. No allocator environment workaround was validated. Additional
local candidate experiments and their failures are retained privately, not
promoted to recognized/supported launcher settings. No global allocator or
security setting changed. The policy below requires no allocator override.

The original whole-page test still reports `reused_first_byte=90` (`0x5a`) after
successful `MADV_FREE`, `PROT_NONE`, then RW recommit. The probe retains that test,
its original ranges and its failure exit separately from the new profile. Default
`--native` / `--all` still fail when original reservations are unavailable.

## Explicit native profile and generated pointer contract

Apply `runtime/patches/apple-memory.patch` to Wonder exactly at
`010ed7a596a621c15d3a8dfa5795ca8c3668b066`. The patch changes only original runtime
code in `runtime/include/nx/{cpu,runtime}.h` and `runtime/src/platform_mac.cpp`.
It applies cleanly to committed pinned blobs. No CMake or generated source change
is included in this unit.

For native Mac integration, compile **every runtime and generated-module
translation unit** with `-DNX_APPLE_HIGH_MEMORY=1`. Use a clean build directory;
do not reuse static libraries compiled for the default layout. The profile is a
build decision, never an automatic retry after reservation failure. Mixing
profiles across translation units is unsupported and not detected at link time.
Combining the profile with a caller-defined `NX_BASE` is a compile error.

| Region | Base | Size |
|---|---|---|
| Modules (`NX_BASE`) | `0x7000000000` | `0x20000000` |
| Heap | `0x7100000000` | `0x180000000` |
| Alias | `0x7300000000` | `0x400000000` |
| Stack | `0x7800000000` | `0x100000000` |
| Kernel | `0x7900000000` | `0x100000000` |
| Thunk identifiers | `0x7B00000000` | `0x100000` |

All ranges remain inside the original reported 39-bit guest address space
`[0x8000000, 0x8000000000)`. Sizes and direct guest-pointer-as-host-pointer semantics
remain intact. Generated code's existing
`NX_BASE + (module_base - 0x4000000000)` expression makes the SDK base
`0x7010000000`; generated canonical module-base inputs remain unchanged.
Tests compile the actual patched headers with this expression and both profiles.
Do not substitute a different `--base` in the generation command to compensate.
Reservation still uses a non-destructive hint, rejects alternate addresses and
returns failure. No `MAP_FIXED`, overwritten region, dynamic relocation or
changed guest-pointer representation is introduced.

The launcher must execute the consistently compiled binary normally, with no
allocator override required. The runtime's existing fatal-on-reservation-failure
policy remains mandatory. Five fresh memory-probe processes passed these exact
ranges; a future OS, linked dependency or application startup can still occupy
them. Validate again in the final linked runtime after SDL/framework startup.
The unchanged mobile layout remains selected when the opt-in macro is absent.
The high profile has not been tested on either simulator.

## Zeroing and protection semantics

Guest allocations use 4 KiB pages; this host uses 16 KiB pages. The patch preserves
the existing split at host-page boundaries:

- Shared edge bytes are zeroed only inside the freed guest slice. Neighbor bytes
  remain intact and the shared host page remains RW. A 4 KiB slice cannot become
  a hardware guard while another live slice shares its 16 KiB host page; guest
  QueryMemory metadata remains responsible for that finer-grained distinction.
- Complete host pages are made RW, explicitly zeroed, optionally advised with
  `MADV_FREE`, then protected `PROT_NONE`. Retained lazy-free pages contain zeros;
  reclaimed anonymous pages also fault back as zeros. Failed whole-page protection
  changes abort because the upstream decommit interface has no error return.
  Failed advisory reclamation is harmless to correctness after explicit zeroing.
- This favors correctness over decommit cost: eager zeroing touches complete
  pages and may increase transient resident memory. Large-region throughput,
  memory-pressure behavior and guard/concurrency interactions need runtime-level
  validation. The existing caller contract must exclude concurrent access to
  memory being decommitted; partial edge writes still require committed RW bytes.

The native fixture and real simulator apps invoke functions extracted from the
**actual applied upstream patch**, not a duplicate zeroing implementation. They
check a shared 4 KiB slice, a mixed span with both shared edges and two complete
host pages, and five complete host pages. Every reused byte must be zero;
neighbors retain `0x5a`. Self-inspection requires full-page `PROT_NONE`, and a
Mach read must be denied before recommit. The occupied-range test separately
requires the original byte to survive a rejected overlapping reservation.

## Reproduction and evidence

Prepare an ignored upstream checkout at the exact pin (see `RUNTIME_MAP.md`).
Default source location is `.local/orchestrate/2026-10-18/apple-preflight/source`;
alternatively set `APPLE_MEMORY_SOURCE` to an existing pinned checkout. The script
reads committed Git blobs, copies only three original source files to its private
run directory, checks/applies the patch there and compiles only its memory section.
It never modifies the supplied checkout.

```sh
scripts/check-apple-memory --native                 # original baseline; observed exit 1
scripts/check-apple-memory --native-profile         # baseline + five high/fixed processes
scripts/check-apple-memory --all-profile            # also both fixed-policy simulator apps
scripts/check-apple-memory --simulators             # original layouts + fixed policy only
PYTHONDONTWRITEBYTECODE=1 python3 -m unittest discover -s tests -p 'test_apple_memory.py' -v
bash -n scripts/check-apple-memory
```

Profile mode succeeds when its selected policy checks pass, while `summary.json`
retains `macOS.exit_code=1` and `baseline_failed=true`. This is explicitly **not**
a default-baseline pass. Tests on this worker ran five cases, zero skips, all
passing; native tests skip explicitly on non-Darwin or absent pinned source.

Policy simulator modes require the preflight's existing private `devices.json`
registration and validate exact ID/name/runtime/type before touching either task
device. In a fresh worktree, copy only that registry from the preflight evidence
into this worktree's ignored preflight directory; do not invent arbitrary IDs.
The task apps are ad-hoc signed and signature-verified, installed/launched as
applications, and checked through their own atomic completion markers. Only
`ACS APPLE-PREFLIGHT-001 iPhone` and `ACS APPLE-PREFLIGHT-001 iPad` were used.
No Wonder device/process, device erase/delete, credentials or game input was used.

Evidence retained under this worktree's ignored `.local/`:

- `memory-fix/default.txt`, `nano0.txt`: initial unchanged baseline and unsuccessful
  process-local nano override; both retain `0x5a` on whole-page reuse.
- `memory-fix/vmmap-default.txt`, `hold.txt`, `overlap-tags.txt`: actual task-process
  VM maps and extended-tag diagnostics. `vmmap-reserved.txt` retains an unsupported
  flag diagnostic; it is not a successful map capture.
- `orchestrate/2026-10-18/apple-preflight/run-20261018T082257Z-0892f9/`: initial
  full patched-policy native and simulator pass.
- `orchestrate/2026-10-18/apple-preflight/run-20261018T082446Z-3e7324/`: final
  instrumented full run, with `macos.txt` default FAIL, five `macos-high-*.txt`
  PASS, iPhone/iPad PASS, source fixture, command log, signed app, inventories and
  `summary.json`. Both simulator reports preserve original lazy-reuse byte 90
  alongside successful `fixed_partial`, `fixed_mixed` and `fixed_whole` checks.

Integration still requires the parent-reviewed patch to be applied by runtime
preparation, consistent profile flags across all native generated/runtime targets,
and full linked-runtime reservation/boot tests. Neither gameplay, resident 6 GiB
heap capacity, Metal/Vulkan sharing, mobile signing for physical devices nor
performance is established by this unit.

## Local Darwin 27 integration (2026-10-21)

The final linked native build on an M5 MacBook Air with macOS 27.0.1 failed to
reserve the high-v1 alias. A non-fixed mmap request for
`0x7300000000 + 0x400000000` returned `0x737c800000` and was correctly rejected.
A subsequent exact Mach allocation failed with status 3; its region query found
an existing RW mapping at `0x76ee000000 + 0x400000`, inside the requested alias.
Another fresh process failed earlier at the high heap: a `PROT_NONE` mapping at
`0x71b4000000 + 0x300800000` overlapped it. Moving only the alias therefore does
not address all observed high-profile conflicts.
This is a virtual address conflict, not evidence of 16 GiB of physical RAM use.

The local patches use `mach_vm_allocate` with `VM_FLAGS_FIXED` and without
`VM_FLAGS_OVERWRITE`, followed by current protection `VM_PROT_NONE`. Conflicts
remain fatal; no existing mapping is replaced, and no alternate address is
accepted. The maximum protection remains available for later RW commits.

An initial local experiment used a Darwin 27 compile definition to select the
complete pre-existing low Apple layout in both `cpu.h` and `runtime.h`: modules
at 12 GiB, heap at 16 GiB, alias at 24 GiB, stack at 40 GiB, kernel at 44 GiB and
thunk identifiers at 56 GiB. Every generated translation unit was rebuilt.
That failed experiment is superseded by the explicit dynamic profile below.

This local integration must be validated in the final native process. Neither
successful compilation nor the address choice alone establishes gameplay,
save/reload correctness, performance, or compatibility with other hosts.

The complete low-layout attempt also failed in the real final process: the
module reservation `0x300000000 + 0x20000000` returned Mach status 3. Its next
mapping was `0x2e6000000 + 0xcda000000`, current/maximum protection `1/1`, covering
the requested module range. The local Darwin 27 layout is therefore a failed
experiment, not a validated compatibility fix. Both fixed-layout families are
blocked on the observed host mappings; gameplay, input, audio and save/reload
were not reached. The original source and game inputs remain recoverable.

### Mapping ownership and dynamic Darwin 27 profile

Actual final-process `vmmap` plus recursive Mach metadata identified the low
module collision as the dyld shared cache's combined `__LINKEDIT` mapping,
`[0x2e6980000, 0x34a394000)`. The larger read-only entry is a shared submap,
not a single 51 GiB resident allocation. At high addresses the process has
`VM_MEMORY_MALLOC_SMALL` regions (tag 2), with about 24 GiB of surrounding
inaccessible reserved space. Both were already present in a diagnostic dylib's
constructor before `main()`, ruling out simply moving reservation before SDL.
The diagnostic injection only observed failures; it did not replace mappings.
Raw address metadata and vmmap evidence remain under the ignored local build.

The current `NX_APPLE_MAC27_MEMORY` profile preserves compile-time `NX_BASE`
at 448 GiB for every generated and runtime translation unit. Only heap, alias,
stack and kernel bases are chosen once in `memory_init()`. Their full sizes
remain 6/16/4/4 GiB. The fixed 512 MiB module range and adjacent 1 MiB thunk
range are reserved first. A top-level Mach scan conservatively excludes entire
submaps and guards, then searches remaining gaps below the 39-bit limit.
All 24 region orders are considered, preferring the 12 alias-before-stack orders.
QueryMemory sorts the alias and stack ranges by address and marks their complement
inaccessible, so a high alias region no longer requires enough space for a stack
above it. Individual region checks preserve pointer classification independently
of the selected order. Layouts with no contiguous 16 GiB gap still fail explicitly.
Candidates are page-aligned; only exact non-overwriting reservations establish
ownership. If a candidate changes after scanning, only this attempt's successful
reservations are released and boot fails explicitly. No allocator protection is
disabled, no region size reduced, and guest pointers remain direct host pointers.
Kernel allocation and VI token initialization now follow layout selection.
The native build verifies the profile definition in every compile command.

The rebuilt final native process passed reservation, loaded main/sdk with zero
unresolved strong links, entered `nn::init::Start`, initialized Vulkan/MoltenVK
and the 48 kHz stereo audio device, and visibly rendered the animated 3D start
screen with the ZL+ZR prompt. After normal window-close (exit 0), a fresh process
through the standard launcher / Play / Default profile also reached that screen.
The two observed alias bases were `0x71a0100000` and `0x783a000000`; both retained
the full region sizes. Render logs around the start screen reported about 30 fps.
Neither run used diagnostic injection, headless mode or scripted game inputs.
The first run remained at the start screen for over 20 minutes before closing;
this is start-screen evidence, not a full-game stability or performance result.
The keyboard launcher accepted Enter, but entering beyond the in-game ZL+ZR
prompt, audible output, combat, save/load and physical controllers remain pending.
The computer-use tool cannot press two non-modifier letter keys simultaneously;
the user can press the default Z+C mapping directly. No tests were run.
