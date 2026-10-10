# PR #2: Mac performance review

Reviewed on October 10, 2026: [ignusloki/pt-pc #2 — mac perf pass](https://github.com/ignusloki/pt-pc/pull/2), by ginnfx.

- Target: `codex/macos-arm64-port` at `e2e2d0002fcb9011b36cc97887b261a6d796576c`.
- Reviewed head: `be9559e143a8cccc8914d07476eedad5bbf9225f`.
- Scope: 20 commits; net changes in 18 files, with 252 additions and 61 deletions.
- GitHub reported a clean merge and no check runs. The head and base were checked again after testing and had not changed.

## Recommendation

Accepted for the Apple silicon Mac branch after the Game Mode packaging correction and normal gameplay checks. No blocking regression was found in the source review, Apple silicon build, or automated gameplay and screenshot checks. The user confirmed that fullscreen mouse clicks, bathroom/mirror rendering, and maze rendering work with MetalFX Quality and reflections on. These results do not establish Windows, Linux, or Intel compatibility or reproduce the author's FPS measurements.

PR #2 was merged on October 10, 2026 as `f850734`. Our tested packaging correction was then applied to `codex/macos-arm64-port` as `2375bc1`. Before adding these reports, the resulting source tree exactly matched the tested `9dbf6da` tree. All 14 packaging tests passed again on the merged branch. The installed game and preview 5 release package were retained. Tests used separate settings with game saving disabled; the installed settings and both save files have identical before/after hashes.

## What changes

| Change | Purpose and review result |
| --- | --- |
| Reflection shader | Uses cheaper depth samples after the first coarse step and exits after a hit or termination. Retains the reconstructed depth for the first step and refinement. Sampled scene comparisons did not reveal damaged reflections. |
| Texture and mesh uploads | Submits uploads without waiting on the CPU for each copy. Retains staging buffers and command buffers until fences complete. Pending staging is bounded at 256 MiB for textures and 128 MiB for buffers. Reviewed same-queue barriers and shutdown cleanup; GPU checks passed. These limits do not cap total game memory. |
| Effect textures | Decodes with up to four workers per batch, uploads four completed textures per frame, and uses a transparent placeholder while decoding. Skips pending opaque and HNM liquid shapes whose shaders do not honor alpha. |
| Render passes | Skips redundant read-to-read image barriers and discards old attachment contents where a pass overwrites the target. Accumulating bloom and motion-blur compositing retain their contents. |
| Visibility | Reuses existing occluder volumes to hide camera draws. Keeps mirror and shadow visibility separate, excludes sky, and disables this culling in free-camera/photo mode. |
| Game Mode | Adds the games category and legacy Game Mode support declaration to the native Mac packager. Packaging consistency needs the follow-up below. |

The net diff retains our app icons, startup microphone permission request, microphone setting, fullscreen pointer fix, Fast Walk option, and MetalFX queue handoff. The settings persistence test still passes. The microphone permission dialog was not re-tested in this review; its implementation is unchanged by this PR.

## Recommended packaging change

In `tools/macos/package.py:36–39`, keep `GCSupportsGameMode = true` and add `LSSupportsGameMode = true` to game bundles. Apple documents the latter for macOS 26 and says omitting it may prevent Game Mode from activating. Apple also explicitly permits combining the modern and legacy declarations. The legacy declaration is supported; this is a compatibility improvement, not proof that Game Mode currently fails. See [Apple's key documentation](https://developer.apple.com/documentation/bundleresources/information-property-list/lssupportsgamemode) and [Apple DTS guidance](https://developer.apple.com/forums/thread/782343).

Apply both declarations to the game metadata in `tools/macos/make_app.py:122–140` as well. The current Mac GitHub workflow calls this second packager at `.github/workflows/macos.yml:59`, and it has neither declaration. Otherwise, the native installer runtime and workflow-produced game apps differ in Game Mode eligibility. Keep the installer itself out of the game category and update the existing packaging assertions for the modern key.

This is a small packaging follow-up. It does not require rewriting the renderer or removing the PR's performance work. Game Mode activation is ultimately decided by macOS at runtime.

## Validation performed

Host: Apple M1 Pro, macOS 26.6.2. Review sources were built in an isolated worktree at the exact PR head. Built both `pt` and `pt_release` for arm64, with the existing macOS 14 minimum.

| Check | Result |
| --- | --- |
| Native build | Passed. |
| Six CPU test executables | Light culling, blur/sway, pipeline-cache storage, display modes, voice matching, and settings round trip passed. Voice matching passed all 68 cases, including its expected missing-model error case. |
| Packaging suite | All 14 tests passed, including the PR's new category and legacy Game Mode assertions. |
| Reflection GPU test | All 29 expectations passed. This existing test does not cover every branch of the modified reflection march. |
| GPU queue handoff test | Passed timeline, split-submission, and cleanup checks. |
| Additional startup smoke runs | Nine runs completed across native rendering and MetalFX Quality/Performance, with culling enabled/disabled. These are startup checks, not nine complete gameplay routes. |
| Complete scripted route | Reached the ending with 27/27 milestones, using MetalFX Quality, reflections on, enhanced textures off, at 1280×720. |
| Bathroom route | Passed 8/8 milestones with culling on and 8/8 with culling off. All three paired screenshots were pixel-identical. |
| Photo mode | Passed 5/5 milestones, including entry/exit and session state. |
| Current-release comparison | The cached current Mac 1.0.4 build passed the same 27/27 route. All 25 paired scene screenshots closely matched. The largest mean RGB difference was 0.035 on the 0–255 scale; most views were identical or differed by only a few pixel levels. |
| User data preservation | Settings and both save-file hashes remained unchanged after all runs. |

The complete scripted routes simulate 90,000 frames and render settled snapshots at selected points. They are not 90,000 continuously rendered frames or a substitute for real-time gameplay. Audio devices and microphone capture were disabled, and the voice puzzle used scripted input. Enhanced-texture mode, hardware ray tracing, VR, and real controller input were not exercised by these routes.

Metal API diagnostics were enabled; one startup run also enabled Metal shader validation. No GPU failure or validation error was observed in those runs. Khronos Vulkan validation layers were unavailable through the direct MoltenVK path, so Vulkan validation cannot be reported as passing.

## Pre-existing errors and remaining checks

The complete route logged four `trapLightEnable.lua:27` errors about a nil table. The current-release baseline produces the same four errors and still passes all milestones. They should be tracked separately; they are not evidence of a regression introduced by this PR.

The author reports a small FPS improvement with reflections and lower loading stalls. The scripted route's total runtime is not an FPS benchmark. Verify that claim with the same resolution, graphics settings, scene, and repeat runs in normal gameplay.

The final manual merge check passed: the user confirmed fullscreen mouse alignment, bathroom/mirror rendering, and maze rendering with MetalFX Quality and reflections on. Additional optional checks are listed in `docs/pr-2-gameplay-checklist.md`; this confirmation does not separately establish results for every optional setting or an FPS improvement.

No GitHub checks have run because the current Mac workflow responds to release publication and manual dispatch, not pull requests. That is an existing workflow limitation. Before carrying these shared renderer changes into the original cross-platform repository, obtain Windows/Linux builds and tests and a native Intel Mac check. Focused automated coverage for asynchronous upload lifetimes and pending effect textures would also strengthen a future upstream submission.

## Local evidence

- Review binary: `/Users/suporte/PT-Mac/build/pr2-review-20261010/macos-arm64/pt`.
- Review release binary: `/Users/suporte/PT-Mac/build/pr2-review-20261010/macos-arm64/pt_release`.
- Test helper and logs: `/Users/suporte/PT-Mac/build/pr2-review-20261010/`.
- Gameplay results: `walkthrough-results.json` and `baseline-results.json` in that directory.
- Screenshot comparisons: `image-comparison.json` and `walkthrough-shots/` in that directory.
- Preservation records: `profile-preservation.json`, `walkthrough-profile-preservation.json`, and `baseline-profile-preservation.json` in that directory.

The existing managed review worktree was reused for the initial PR review, then placed on `codex/pr2-game-mode` for the packaging correction. Its tracked source files are clean. The tested changes have now been integrated into `codex/macos-arm64-port`.

## Implementation follow-up

The recommended Game Mode correction has now been committed locally as `9dbf6da` on `codex/pr2-game-mode`, based on the reviewed PR head. Both Mac packagers declare legacy and modern Game Mode support; the existing runtime packaging assertion checks the modern declaration too. The corrected native and workflow bundles were built, signed and inspected successfully, and all 14 packaging tests passed. The native app's icon and packaged voice loading were verified.

An isolated **P.T. Performance Check** app is available under `build/pr2-review-20261010/game-mode-test/`. Its launcher supplies separate settings and save paths, disables game saving and unlocks the loop browser for testing. Live startup displayed the original options screen, requested microphone access before gameplay, and entered the opening room. Window, Borderless and Fullscreen transitions were observed. Settings and both save-file hashes remain unchanged in the normal player profile.

The normal gameplay checks are recorded in `docs/pr-2-gameplay-checklist.md`. The user confirmed the remaining pointer and rendering checks, and integration into the Mac branch is complete. No release package has been replaced by this test build.
