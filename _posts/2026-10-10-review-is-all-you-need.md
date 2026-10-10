---
title: "Review is All You Need"
date: 2026-10-10
author: jli
tags: ai agents game-development claude unity
---

![The heroine fighting skeletons inside Dustfang Bastion, one of the game's forts](/assets/images/review-is-all-you-need/hero.webp)

> TL;DR - An AI agent (Claude Code on Opus 5.5) built a Genshin-like action RPG prototype in Unity in under five days, out of purchased asset packs. It wrote all the code. My job was review. What made that work wasn't a clever prompt. It was a review page that the agent published as a Claude artifact every round. On that page I could drop a pin on a frame of a clip, and the agent could read the pin back, fix the problem and answer the note.

### The project

Token Isles (the repo is called TokenImpact) is a 2 km island with ten biomes and 38 named places. It has a sword-and-shield heroine and later a party of four, skeleton camps and forts, a two-phase necromancer boss, a quest line with conversations, loot, an inventory, puzzles, chests, and a fog of war lifted by altars. None of the scenes are built by hand. Editor "builders" generate everything from a library of purchased asset packs, so most work goes the same way: change a builder, run it, look at the result.

An agent can do the first two steps on its own. It can also look at the result, but it can't tell whether a sword swing feels heavy, a camera cut is jarring or a village looks lived in. That part is review, and review is where my time went.

| What | How much |
| --- | --- |
| Review rounds | 11, from Oct 6 to Oct 10 |
| Shots reviewed | 531 stills and clips (clips with the game's own sound from round 8 on) |
| Notes | 95: 76 fixed and answered, 19 from round 11 in progress |
| Code | about 65,000 lines of C# in 215 files, all written by the agent |
| Review media | about 1.2 GB on eight review pages and an index |

### Why an artifact

On day one I asked for visual review instead of prose. Reading a paragraph about a mountain doesn't tell you much. Pasting screenshots into chat doesn't work well either: chat has no timeline, a screenshot can't be compared with last round's, and nothing tells the agent which feedback it has already handled.

An artifact is a web page that Claude publishes to claude.ai. It is private by default and has a stable URL. A plain page would only be a gallery. Three runtime capabilities turn it into a review tool:

- **A database** (`db`) shared by the page and the agent. The page writes my notes, verdicts and round submissions. The agent reads them with its data tool and writes its replies into the same documents.
- **User identity** (`user`) records who wrote a note and whether a viewer may write.
- **Asset uploads** (`assets`) let me add a screenshot or clip from my own play session (the page caps an upload at 20 MB) and mark it up like any other shot.

Republishing to the same URL replaces the page and its media, but the database stays. So the page and the media can be replaced at any time, while the conversation on it is kept.

The loop has four steps:

1. **Build and capture.** The agent changes builders and regenerates. Then it records scripted clips in Play mode at a fixed 1/30 s step with an offline audio mix, and takes edit-mode stills from JSON shot lists.
2. **Publish.** It encodes the media, extends the round's manifest and publishes the page (the limits are covered below).
3. **Review.** I go through the reel, mark up shots, set a verdict per shot, press *Submit round*, and type "done" in the chat.
4. **Fix and reply.** The agent reads every note, fixes it, and replies on the note with what changed and the round that ships it, then sets the note to *fixed*. The next round shows the new version of each shot next to the old one.

[![The review page: a reel of shots on the left, a still of Kagetsu Keep with four numbered pins in the middle, the notes with the agent's replies on the right](/assets/images/review-is-all-you-need/review-page-pins.webp)](/assets/images/review-is-all-you-need/review-page-pins.webp)
*The review page, which the agent wrote for round 1 and has extended since. The reel is on the left, the screen and mark-up tools in the middle, and the notes sheet on the right. Four pins on round 5's Kagetsu Keep each mark a separate problem, and each one has the agent's reply from round 6.*

The page is one file of about 80 KB (HTML, CSS and JavaScript, no build step). Its own code calls it "a screening room for dailies". Reviewing is mostly keyboard: `P`, `B`, `D` and `V` pick pin, box, draw and select; `G` and `N` mark a shot good or needing work; `[` and `]` move between shots; `Space` plays; `,` and `.` step one frame. Filters show unreviewed shots, shots with notes, or shots that need work, by round. Speed matters here because I am the slowest part of the loop.

### Feedback an agent can act on

This is a real note from round 3, as the agent reads it from the database:

```json
{
  "media": "hud-explore@r3",
  "t": 11.474,
  "shape": "pin", "x": 0.2987, "y": 0.8756,
  "severity": "must",
  "category": "UI & HUD",
  "text": "This ui of bottles are taking to much spaces,",
  "status": "fixed",
  "reply": "Fixed in round 4. The stamina and energy flasks are now…",
  "replyRound": 4
}
```

[![The HUD clip paused at 00:11.14 with a pin on the left flask, and the same note with the agent's full reply](/assets/images/review-is-all-you-need/review-hud-note.webp)](/assets/images/review-is-all-you-need/review-hud-note.webp)
*The same note on the page. The timecode reads seconds and frames (11.474 s is 00:11.14 at 30 fps). The reply in full: the flasks shrank to under half their size, moved against the ends of the action bar, and the whole bar went from 52% to 41% of the screen width.*

Every field is there for a reason:

- **`media`** is the shot id plus its round, so there is never any doubt about which version a note refers to.
- **`t`** is the time in the clip, to the frame. On video, a mark only shows within 0.6 s of its timestamp, so it stays on the frame it is about.
- **`x`, `y`** are normalized to the frame, so they work at any player size. Boxes add width and height, and pen strokes add their points.
- **`severity`** (must fix, should fix, polish) tells the agent how to triage.
- **`category`** comes from a list per area. World shots use World layout, Terrain, Art & dressing, Places, Gameplay, Quests & dialogue, UI & HUD, Audio, Camera, Performance and Bug. Character shots use Animation, Ability & timing, Hit feel, VFX & audio, Character art, Camera and Bug.
- **`text`** can be short and sloppy, because the pin and the timestamp already say *where* and *when*. The words only need to say *what*.

[![A clip paused at 00:05.25 with a pin on a skeleton's sword, next to a panel of attack numbers and a timing bar](/assets/images/review-is-all-you-need/soldier-numbers.webp)](/assets/images/review-is-all-you-need/soldier-numbers.webp)
*A pin on a clip at 5.25 s; the red tick further along the timeline is the clip's other note. The agent puts numbers next to character clips (damage, reach, and the hit, next-input and cancel windows as a bar), so I review against the intended values and not only the picture.*

On the agent's side, a pin becomes a picture it can look at. A small tool, `frameat`, extracts the exact frame. Another, `pin_crop`, draws a ring at the pin and writes a 2× crop around it:

[![The frame at 5.836 s with a red ring on the skeleton's helmet, and a 2x crop around it](/assets/images/review-is-all-you-need/agent-view.webp)](/assets/images/review-is-all-you-need/agent-view.webp)
*"In attack 3, the sword is going through the character's body", as the agent sees it. Its reply in round 6: measured on the rigs, the wind-up ran the blade 17 to 36 cm into the helmet. Attack 3 became a different clip from the same pack that keeps the blade at least 6 cm from the head and body, and the change went to all three skeleton types.*

#### Video catches what stills can't

Stills show composition. Most of what makes a game feel wrong, though, happens over time:

- **Timing.** "This attack (attack 3) seems to be cancelled by another attack when it shouldn't" is pinned at 7.2 s of a clip. No single frame can show an attack being cancelled. The agent found two causes. It had cut the attack at 74% of its strike to skip a long settle. And in the clip the next attack started at a fixed time, while the recorder was playing every clip at half speed, so attack 3 ran into it. That is how we learned the footage itself was wrong.
- **Camera.** "Let's not transition the camera angle on this interaction" is pinned at 11.7 s of the waystone prayer. The note became a rule for the whole project: no interaction moves the camera.
- **Sound.** One note said the audio in every recording stuttered. The cause was the recorder, not the game. Unity's offline `AudioRenderer` only mixes whole 1024-sample blocks, and the recorder asked for 1600 samples per frame, so every frame ended with 12 ms of silence and sounds ran 1.56× too long. Every clip with sound was recorded again.

[![A clip of a skeleton knight's attacks with two timestamped notes, one asking "what are those slow down in attack sequence? did we added it in ourselves?" and the agent's reply "Yes, partly our own doing"](/assets/images/review-is-all-you-need/review-timing-note.webp)](/assets/images/review-is-all-you-need/review-timing-note.webp)
*Timing notes on a clip. Asked "did we add it ourselves?", the agent answered "Yes, partly our own doing" and explained both causes.*

[![The waystone prayer clip with a pin at 00:11.20 on the heroine, and the note "Let's not transition the camera angle on this interaction"](/assets/images/review-is-all-you-need/review-video-note.webp)](/assets/images/review-is-all-you-need/review-video-note.webp)
*A camera note on a video timestamp. The "Look at:" text on the right is the agent describing what the clip should show, so I check its claims instead of guessing what changed.*

In round 11, 16 of the 19 notes are pinned to a video timestamp.

#### Stills for space

Stills are still the right tool for layout. On Kagetsu Keep I put four pins on one still for four problems: foundations too narrow, houses sunk into the ground, and rocks running into buildings in two places. The fixes were general, not local. Every building on uneven ground now gets the ground levelled under it (or is left out where the slope is too steep), and scattered rocks, cliff pieces and trees keep out of every building's footprint at every place. The agent's check over all 516 structures at the 38 places found nothing touching a building.

[![Kagetsu Keep in round 5 and round 6 from the same camera](/assets/images/review-is-all-you-need/kagetsu-r5-r6.webp)](/assets/images/review-is-all-you-need/kagetsu-r5-r6.webp)
*The same camera in round 5 and round 6. Stills are taken from recorded camera poses, so an after shot repeats its before shot exactly.*

#### The agent's side of the deal

A note only works if both sides do their part. Four things on the page come from the agent:

- **A "Look at:" line on every shot**, saying what changed and what to check.
- **Numbers next to clips**, such as damage, timing windows, reach and frame cost, so feel can be compared with intent.
- **Compare buttons** that jump to a shot's earlier and later versions, with earlier notes and replies shown on the shot.
- **A reply on every note**, naming the round that fixes it, and saying so when the problem was the agent's own doing.

#### The footage has to be trustworthy

A review is only as good as its footage, and three of the problems review caught were in the footage, not the game. Until round 7, clips played at about half speed. In round 8, timers ran about 7× fast during capture, because code read the wall clock while the recorder stepped time at 1/30 s; all such code now reads a capture-aware `RealTime` clock. Then came the audio holes. Scripted clips also log a "fallback" whenever a step needed a direct API call because the player's own move didn't land. The fallback count is printed with every take, so a take that cheated gets recorded again or captioned. Footage that lies is worse than no footage.

### Working within the size limits

Artifacts have limits, and a game review produces a lot of video:

| Limit | Value |
| --- | --- |
| Per file | 16 MB |
| Per publish | 64 MB and 255 files, server-side copies included |
| Per page version | 256 MB and 511 files |

Round 11 alone added about 650 MB: 59 clips and over 400 stills. This is how it fits:

1. **Encode for review, not for archiving.** Clips are H.264 + AAC, 1280 pixels wide, 30 fps, 3 Mbps. Only a clip that would pass 15.5 MB gets a lower bitrate (a 60 s clip needs about 1.8 Mbps). Posters are 1280 wide and thumbnails 320, so the reel appears instantly. The media script warns about any file near the limit.
2. **Split pages, don't cut quality.** After round 10 I set a rule: never lower quality or cut clips to fit a page, start another page instead. Round 11 is five linked pages, split by subject: the world, trials and chests, party and items, sound/HUD/quests, and the boss.
3. **One page template, data in a manifest.** Every review page is the same HTML with a different `<title>`. The shots, rounds, earlier notes and cross-links live in `media.json`. Each round's build script extends the previous manifest and never drops earlier media or notes.
4. **Publish in batches to the same URL.** The first batch is the page, `media.json`, thumbnails, posters and stills, so the page is usable right away. Videos follow in batches of at most 60 MB and 250 files. Files left out of a publish stay on the page.
5. **Copy instead of re-uploading.** When a shot is redone, its earlier version is copied on the server from the older page (`{artifact, path}`), so the compare buttons work on the new page. The catch is that copies count toward the 64 MB of a publish: one page had 93 MB of copies and they had to be split across publishes.
6. **Carry history forward.** A new page's manifest carries every earlier note and reply as read-only history, so the full record is on the newest page.
7. **Link the set last.** The pages are published with placeholder links. Once their URLs exist, the manifests are rebuilt and only `media.json` and the index are published again.
8. **Check mechanically before publishing.** A one-minute script confirms the files exist, the sizes and batches are within limits, the server copies are present and the manifests reference only published paths.

[![The rounds index: eight review pages with the rounds each holds and how many of their 256 MB they use](/assets/images/review-is-all-you-need/rounds-index.webp)](/assets/images/review-is-all-you-need/rounds-index.webp)
*The rounds index lists every page and how full it is. Review I (rounds 1–3) holds about 232 MB, Review II (rounds 4–7) about 197 and Review III (rounds 8–10) about 205. Round 11's five parts hold 190, 157, 150, 215 and 92 MB.*

### Managing assets

There are two kinds of assets to manage: the game's and the review's.

**The game's assets.** No art was made from scratch. Everything comes from purchased packs on an external drive: 267 `.unitypackage` files holding 323,539 assets, 165 GB in all. A small Rust CLI, `uai`, indexes them for full-text search, follows dependencies across packages, exports an asset with its `.meta` files so GUIDs stay intact, extracts previews, and runs as an MCP server so the agent can use it directly. The agent pulls a single prefab with its dependencies instead of importing a whole pack, so the project's `Assets` folder is 3.8 GB and not the library's 165 GB. Before any building, I approved which pack would dress which biome. Review added more rules over time, for example "build every UI from the Synty interface prefabs" after round 2 rejected a custom HUD.

The packs themselves stay untouched. Builders make copies and adapt them: they strip AudioSources from effect copies, move materials to URP and cap billboard sizes. The few patches to third-party code are documented. Builders keep asset GUIDs stable, so rebuilding a prefab never breaks the scenes that use it. Binaries go to Git LFS (about 9,900 files), and Unity's YAML stays text so it can be diffed.

**The review's assets.** All of it is in the repo: each round's media in `Review/round<N>/`, one manifest per round, the build script that made it and the replies it sent. Stills are data too: each shot list records its cameras, so before and after shots repeat the same pose. The notes are exported to JSON in git, and that paid off. When I moved to a new claude.ai account, the first two review pages could no longer be opened from it. Their notes and replies survived because every later page carries them, and because the exports are in git.

The agent also keeps a runbook (`CLAUDE.md`) up to date: builder order, commands, recording and still tools, and gotchas. Each new session starts from commands that already work.

### What I'd copy

- **Make the review page part of the product.** The agent wrote it once and keeps extending it, and every round fills it.
- **Give each note a pin, a timestamp, a severity, a category and one sentence.** The pin says where, so the sentence can be short.
- **Prefer video.** Timing, camera and sound only show up there.
- **Ask the agent to say what to look at, and to answer every note** with the round that fixes it.
- **Treat the capture pipeline as part of the game.** Test it like gameplay code.
- **Split pages instead of cutting quality.** Publish in batches, copy instead of re-uploading, and carry history forward.
- **Keep everything in git, including the notes.**

Writing 65,000 lines of C# took the agent a few days. The scarce resource was never code. It was my attention, and how precisely I could say what was wrong. Every hour spent making review faster and more exact paid off in every round after it.
