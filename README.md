# KernelSU: backslashxx source archive for sagit

Independent archive maintained by W-874 for Xiaomi Mi 6 (sagit), Linux 4.19 non-GKI. Original project: [backslashxx/KernelSU](https://github.com/backslashxx/KernelSU); official upstream: [tiann/KernelSU](https://github.com/tiann/KernelSU). The original repository became unavailable. This is not an official release channel and does not claim authorship of the archived implementation.

## Provenance

| Reference | What it preserves |
| --- | --- |
| Original pin `9a02a5c1e552fb21cdb505c3c0428901eca43848` | Original commit and all 2,828 reachable commits are preserved in the offline bundle, not uploaded as this ref. The local original had only `FETCH_HEAD`, no branch or tag names. |
| `archive/verified-history` (`b4ea2ba2fd9544d9e27de3e8f93f7e8a0d685d46`) | Original, unrewritten history through this ancestor; 2,743 commits. |
| `archive/sagit-driver` (`527c5665dc146016f80d834ca8543e63b97ad490`) | Exact original `kernel/` tree (`5106c8fc1b7b52c055be485a4be14ce5de33c7fd`) plus the root license. An export, not the original commit. |
| `archive/sagit-source` (`c1ec87e7ce2f4f15c24046dbe645dceea5599025`) | Same driver plus Manager, ksud and remaining source from the pin. Retained files and modes are identical to the pin. |
| `maintenance/sagit-4.19` | Source export plus local documentation and integration changes. Do not merge `upstream/main` into the export. |

The source export excludes the published dummy Manager keystore, the nine userspace executables below `userspace/ksud/bin/`, and the originally tracked but ignored `AGENTS.md`. Some prebuilt redistribution inputs are unverified. The original history is not attached to this export: removing a secret only at the tip would still distribute it in Git history. The exact original kernel already trusts a publicly known test signer. Do not distribute trusted Manager builds signed with it or treat this preserved driver as production-hardened. No APKs, signing services or GitHub Actions releases are provided. See `LICENSE` (GPLv3), `kernel/LICENSE` (GPLv2), and individual files for additional licenses, including MIT for picoprintf.

## Updating against official upstream

Keep `origin` pointed to this archive. Configure `upstream` as `https://github.com/tiann/KernelSU.git`. The original pin descends from `2a60b2893b8d0d885d1ceff2287d8ad854f0e0f4`; it has 93 subsequent non-merge commits. At rescue time, that exact common ancestor was still official `main`: no missing official commits. Do not compute ancestry from the synthetic public export. To inspect original ancestry offline, clone the full rescue bundle with `git clone -b archive/sagit-pinned sagit-original-history.bundle original`.

**Review ledger** (no functional backports at rescue):

| Official commit | Decision | Evidence |
| --- | --- | --- |
| `2a60b2893b8d0d885d1ceff2287d8ad854f0e0f4` | Last official tip reviewed | All ancestors included in original pin. |
| `dbb73665`, `a23184a0`, `f866d782` | Already present | Profile-name validation, Manager argument handling and module-config length checks. Examples of small future candidates, not a queue to reapply. |
| `28db74f6`, `0faec0cc` | Already present with downstream changes | SELinux generation handling and module preinit use different legacy paths. Review behavior, not just context lines. |
| `02ffbd44`, `bec9c134` | No new sagit port | x86-64 and 6.6-specific changes do not justify adding a backend or changing ABI here. |

For the next review:

```sh
git fetch --no-tags upstream main:refs/remotes/upstream/main
last=2a60b2893b8d0d885d1ceff2287d8ad854f0e0f4
git merge-base --is-ancestor "$last" upstream/main && git log --reverse --oneline "$last"..upstream/main
```

If the ancestry check fails, compare histories before advancing the cursor. Classify each official change as **present**, **direct candidate**, **adapt**, **not applicable**, or **defer**. Prioritize authorization, bounds, lifetime and ABI fixes. Use `git cherry-pick -x` for an isolated compatible patch; for a manual adaptation record `Upstream-Commit: <full SHA>` in the message. Append the upstream SHA, disposition, downstream SHA, scope and verification to the ledger. Advance the reviewed tip only after all intervening changes have dispositions.

This fork uses a unity kernel build with ARM/ARM64 legacy hook paths and LSM/version backports, different Manager identity checks, and old-kernel credential, seccomp and policydb handling. The top-level v6 UAPI is shared at the baseline, but the kernel carries separate headers. Check actual 32/64-bit ioctl layouts. The older Manager and ARM32 ksud support, custom preinit runner and module boot order are not interchangeable with GKI/KMI code. Do not enable optional shell privilege settings as a compatibility workaround.

## Build and release gates

Sagit already vendors the driver in `kernel/xiaomi/msm8998/drivers/kernelsu`; it does not run `setup.sh` or download this repo during a normal product build. Preserve its opt-in Kconfig delta and `vendor/xiaomi/sagit-kernelsu.config`. For an external integration the maintenance setup helper defaults to the immutable driver export; do not use the historical helper on `archive/*`. Verify source identity with `git rev-parse archive/sagit-driver:kernel`.

In the configured uwuAOSP product environment build `uwu_sagit`'s `bootimage`; require `CONFIG_KSU=y`, shell-always-SU and KernelSU ADB-root disabled, successful built-in link and `Image.gz-dtb`. Compare the vendored code to the pinned driver, allowing only documented local changes. For a future userspace change build matching Manager and ksud with separately verified dependencies and test-only signing in an isolated environment. For a functional release, test allow/deny/revoke, rejection of untrusted Managers, 32/64-bit clients, enforcing SELinux, module install/disable/preinit and recovery rollback in a separate authorized validation window. A successful compile is not evidence of a secure signer policy, a working boot, or behavioral equivalence.
