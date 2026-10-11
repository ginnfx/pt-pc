# PR #2: corrected Apple silicon test build

Build: `9dbf6da`, based on PR #2 at `be9559e`, with Game Mode support completed in both Mac packagers. October 10, 2026.

The app is **P.T. Performance Check**. To reopen it after quitting, run:

`/Users/suporte/PT-Mac/build/pr2-review-20261010/game-mode-test/Launch P.T. Performance Check.command`

Use this launcher so the app gets its separate test settings and save location. The test uses your existing extracted game files, disables game saving, and makes all loops available. It does not replace your installed game or its progress. Settings changed in the test menu persist only in the test profile.

## Starting settings

Open PC Settings with F10, or Fn+F10 if macOS uses that key for a system function.

| Setting | Value |
| --- | --- |
| Display mode | Window for the first checks; then try Borderless and Fullscreen |
| Window resolution | Keep the current size, approximately 1280 × 800 after the display check |
| Upscaler | MetalFX |
| Quality | Quality |
| Frame generation | Off |
| Reflections | On |
| Enhanced textures | Off initially |
| Film grain | 0 for easier image comparisons |
| Native HDR | Off |
| V-sync | On for the main gameplay check |

Keep the same resolution when comparing modes. A Retina window renders more pixels than the logical size shown in its menu, and Fullscreen may choose another supported display resolution.

## Short gameplay check

- [ ] **Startup and controls:** confirm the first-boot settings screen is visible, then start the game with Esc. Walk and turn in the hallway using your usual mouse/keyboard or controller. Check sound and movement.
- [x] **Fullscreen pointer:** in PC Settings, switch Window → Borderless → Fullscreen. Move the physical mouse over V-sync and click its value. It should highlight and toggle the row under the pointer. Return V-sync to On and switch back to Window.
- [x] **Reflections and loading:** in PC Settings → Extras → Loop Browser, choose **Loop 6: the bathroom, the flashlight**. Look at the bathroom, mirror, sink and floor while moving the camera. Check for missing surfaces, damaged reflections, flashing effects, freezes, or repeated loading stalls.
- [x] **Visibility:** choose **Loop 13: the maze and the peephole**. Move around corners and inspect the peephole. Walls and objects should not vanish when you turn or approach them.
- [ ] **Photo mode:** press F7 to enter and leave photo mode. Move the free camera around a corner; geometry should remain visible, and returning to gameplay should work.
- [ ] **Mode changes:** repeat a demanding view with Upscaler Off, then restore MetalFX / Quality. Toggle enhanced textures once and check that the image and menus resume. If enhancement reports a missing tool/model, keep that message separate from a rendering failure.
- [ ] **Microphone:** in PC Settings → Sound, toggle Microphone Off and On and use Microphone Test. Startup microphone access was already granted for this isolated app; macOS should not ask again while that permission remains granted.
- [ ] **Quit and reopen:** quit normally, reopen using the test launcher, and check that the first-boot options still render. The test starts fresh because game saving is disabled.

For an FPS comparison, turn V-sync Off and keep Frame limit Unlimited in both builds, using the same scene, window size and settings. Restore V-sync On afterward. The automated route timing is not an FPS measurement.

If a check fails, report the loop, display mode, upscaler/quality, and action that caused it. The current test log is:

`/Users/suporte/PT-Mac/build/pr2-review-20261010/game-mode-test/player-test/pt.log`

## Completed checks

Both corrected packagers produced signed arm64 game bundles with both Game Mode declarations. The installer metadata remains outside Game Mode. The native test app retained its icon, passed packaged voice loading, and all 14 packaging tests passed. First-boot options, the startup microphone request, entry into the opening room, and the three display modes were observed in the live app.

On October 10, 2026, the user confirmed that fullscreen mouse clicks line up and that the bathroom/mirror and maze render correctly with MetalFX Quality and reflections on. These were the remaining manual checks before merging PR #2. The other optional checklist items above have not been individually reported as completed.

The prior PR review also passed all 27 automated gameplay milestones through the ending, with 25 paired scene comparisons against the current Mac build and identical bathroom screenshots with visibility culling on/off. The user's settings and both save-file hashes remained unchanged after live startup.

PR #2 was merged as `f850734`, and the Game Mode correction was applied to the main Mac branch as `2375bc1`. Its source tree matched this tested build, and all 14 packaging tests passed again after integration. The installed game and preview 5 release package remain unchanged.
