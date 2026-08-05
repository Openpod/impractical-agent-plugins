---
name: impractical
description: Create and refine studio-quality motion-graphics launch videos with Impractical Motion. Use when the user wants a marketing, launch, promo, or product video made or tweaked, mentions Impractical, or wants to edit an Impractical draft, extract a brand kit, or export a final MP4.
---

# Impractical Motion

You direct the video yourself. Impractical provides the engine, the preview
build, the render, and the studio the user tweaks in — you provide the
direction. Everything runs through the **Impractical MCP server**; the video's
source code lives in the cloud, never in the user's project directory.

## 0. Setup check (once per session)

Call the `account_status` tool (`whoami` is its legacy alias). If the
Impractical tools are not available, the Claude plugin is not connected. Tell
the user to run:

    npx --yes @impractical-ai/motion setup --claude

Then ask them to run `/reload-plugins` and type `continue`. If the weekly quota
is exhausted, call `upgrade` to give the user the authenticated billing page.

## 1. Read the engine before you write anything

Call **`engine_types`** first. It returns the TypeScript surface a scene is
written against: the `ctx` object, every named text/logo/transition effect with
its options, the named eases, and the determinism rules. Writing a direction
without reading it produces slop.

## 2. Make a video

Before scaffolding, call **`motion_context`** with the user's brief — plus
`brandSlug` and `contentType` when you know them. It returns the current
versioned house skills, the best matching production references, available
template schemas, and rendered reference frames. Study those frames and sources
before choosing a template. The library is live: do not rely on a remembered
catalog from an earlier session.

When the workspace has personalization enabled, the result also carries the
user's resolved motion profile: `personalStyleSkill` (confirmed preferences —
follow it like house guidance), `acceptedExamples`, a reranked
style/template order, and a `contextId`. Pass that `contextId` to
`create_video` as `motionContextId` so the draft is pinned to the
recommendations that produced it.

Gather from the user (ask only for what's missing):

- **Value prop** (required): one or two sentences on what ships and why it matters.
- Optional: their product's front-end code, a brand URL, screen recordings.

If they gave a brand URL and no kit exists yet:

    brand_list
    brand_extract  { url: "https://example.com" }

Then scaffold the project:

    create_video  { valueProp: "...", brandSlug?: "example-com", appCode?: "...", templateId?: "...", templateParams?: {...} }

This creates the project and a draft with starter files. It deliberately does
**not** run a server-side director — you are the director.

## 3. Direct it

A scene is two files, and both must exist for the preview to build:

- **`App.tsx`** — the stage markup. Plain React and CSS; this is the product UI
  on screen.
- **`direction.ts`** — the choreography. `export default (ctx) => { ... }`
  against one paused GSAP timeline.

Write them with `write_draft_files` (whole files) or `edit_draft_file` (exact
string replacement — prefer this for revisions). Every save rebuilds the
preview and **returns the build error if your code does not compile** — that is
your compile loop, so read it and fix rather than guessing.

Rules that matter:

- `brand.json` is law for colors and fonts. Read it; do not invent a palette.
- Never edit `project.json`.
- The timeline is rendered by seeking frame by frame. Anything that is not a
  pure function of timeline position — `Date.now()`, `Math.random()`,
  `setTimeout`, CSS animations, autoplaying `<video>` — desynchronises the
  export. Use `ctx.video()` for footage.
- Only `.ts/.tsx/.js/.jsx/.css/.json` files sync, max 100 per draft.

Craft, briefly — the full language is `guidance.motionDirection` in the
`motion_context` result. That copy is live and versioned; this summary is a
snapshot, so when the two disagree, the tool result wins.

- **Beat it out before writing code.** Every beat needs a time, a thing on
  screen, and a move. A beat with no move is not a beat.
- **Text enters on `rise`** — `text.reveal(el, 'rise')` on `promo-out`, lines
  ~0.2s apart. Do *not* default type to `'blur'`: word-by-word de-blur reads as
  out-of-focus, and across a stagger it turns to mush. Blur is for whole-frame
  moves — `fx.spike(el, 'blur')` alongside a whip or dive, where it hides the
  cut. Never animate blur above 20px.
- **Every ease is a named house curve.** `ease-out-*` for entrances,
  `ease-in-out-*` for travel and every camera move, `ease-in-*` for exits and
  dives only. Never leave an ease unspecified; never put `ease-in-*` on an
  entrance. The promo subset: `promo-out`, `promo-soft`, `promo-inout`,
  `promo-in`, `launch-out`.
- **Fast**: entrances 0.4–0.6s, exits shorter, staggers ~0.2s apart, nothing
  over 1s but a deliberate hold drift or slow push.
- **Nothing is ever static.** Every hold carries a slow drift or camera push.
- Hold long enough to read. A beat that looks legible scrubbing the timeline is
  usually still too fast on screen.
- Cut hard between sections; crossfading backgrounds reads as cheap.
- One idea per beat. If two things move for different reasons at once, split them.

### The five moves that separate a real video from a generated one

These are the engine's `ctx.fx` and `ctx.mask` kits. Reaching for them is most
of the gap between output that looks authored and output that looks scripted.
Full signatures are in the engine types; the rules of thumb are:

1. **Hide every whip cut with one spike, never two tweens.**
   ```ts
   camera.move(camera.framing('[data-next]'), { duration: 0.28, ease: 'promo-inout' }, 'whip')
   fx.spike('[data-panel]', 'blur', { amount: 14 }, 'whip')     // ONE call
   ```
   Hand-writing `tl.to(stage, { filter: 'blur(12px)' })` up and back is the
   single most common way this looks amateur: the two tweens drift out of sync
   and the user gets two unlinked controls. `fx.spike` is one tween, in and
   out, locked to the move.

2. **Feather full-frame reveals.** `mask.reveal(el, 'feather', { direction, softness: 0.3 })`
   — a hard edge across a whole frame reads cheap, and `clip-path` cannot make
   a soft one at all. Use `wipe`/`iris`/`slit`/`barn` when you want the hard
   edge deliberately; `radial` and `blinds` for softer variety.

3. **Land on light, not on opacity.** One `fx.spike(el, 'bloom')` as a beat
   settles reads as a light source. Fading opacity reads as a slideshow.

4. **Put colour inside the letters.** `mask.fill(el, { gradient, drift })` uses
   `background-clip: text`, so the gradient lives in the glyphs and moves
   through them. Far stronger than a coloured headline, and one call.

5. **Composite light instead of shipping a PNG.**
   `mask.blend(disc, 'screen')` behind type gives glow with no asset.
   `multiply` for ink on cream, `luminosity` for duotone.

Two things to get right or the effect misfires:

- **Grain must be continuous.** Apply it once to a full-frame plate that sits
  above every beat and is never cut (`fx.apply(plate, 'grain', { amount: 0.2 })`),
  or it visibly blinks at each hard cut.
- **Effects are seasoning.** One or two per beat. `displace`, `chroma`, and
  `threshold` are accents for a single hit, not a look to hold.

Ask `motion_context` for the **`optical`** style to see all of this composed in
one reference, or the **`whip-cut`**, **`soft-reveal`**, and **`gradient-type`**
templates for a working scene you can adapt.

## 4. Look at your work

    get_frames  { draftId: "...", times: [0, 1, 2, 3] }

This renders real frames on the user's machine and returns them as images. Ask
for several times in one call — once the page is open the extra frames are
nearly free.

**Always look at frames before telling the user a change is done.** If a frame
shows text clipped, an element off-frame, or a blank region, fix it and look
again. This is the difference between shipping a video and shipping a guess.

## 5. Export and hand off

    export_video  { draftId: "..." }
    studio_link   { projectId: "...", slug: "..." }

Always give the user the studio link — that is where they watch the cuts and
tweak visually. The export URL expires, so only export once they are happy.

## The personal motion profile

The `motion_profile` tool manages what Impractical remembers about this user's
taste. The rules are strict because trust in the profile depends on them:

- **Read before directing.** The personalized context from `motion_context`
  already includes the resolved profile; call `motion_profile { action: 'get' }`
  only when the user asks about their profile or you need rule keys.
- **`remember` is for explicit requests only** — "remember that I like…",
  "always keep my videos under 15 seconds". Never call `remember` for a
  preference you merely inferred.
- **Inferred preferences go through `propose`.** If the user's feedback during
  this session implies a durable preference, submit it as a structured rule
  with `propose`. It becomes a pending proposal that has zero effect until the
  user confirms it.
- **Present at most one pending proposal, and only after a completed video.**
  If `motion_context` returned `pendingProposal`, mention it once after the
  export or when the user is clearly done — one sentence, e.g. "Want me to
  remember that you prefer faster pacing? (yes/no)". Call `confirm` or
  `reject` with their answer. Never present more than one, never mid-work, and
  **never silently confirm**.
- The user can `pause`, `forget` a single rule, or `reset` everything; do these
  only when asked.

## Conflicts

The web tweak chat edits the same draft. `edit_draft_file` reads, edits and
writes in one call against the current revision, so ordinary races resolve
themselves. If a tool reports the draft changed underneath your edit, re-read
the file and re-apply your change on top of the new content — never paste stale
content back.

If a tool reports the draft is locked by a job, wait for that job with
`job_status { wait: true }` and retry.

## Costs

- Preview builds and exports run on Impractical's infrastructure; the direction
  thinking is yours, so iterate on the code freely.
- Preview rebuilds take tens of seconds. Batch your edits, then look at frames.
