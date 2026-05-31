---
name: script-to-shotlist
description: Turn a script, screenplay, or story into a director's shot list whose total runtime is DERIVED from the pacing of the writing — not guessed in advance — then craft a keyframe (storyboard) image prompt for each shot and a Seedance 2.0 video prompt for each generation. Use this skill whenever the user has a script/screenplay/story/voiceover/lyrics and wants to know how many shots it needs, how long the finished video will run, or wants shot-by-shot keyframe and video prompts. Triggers on "shot list", "break down my script", "how long should this video be", "how many shots", "storyboard my screenplay", "keyframes for my script", or any request to convert written narrative into a shootable plan for AI video. Use even for long pieces (3–10+ minutes): this skill is built for runtimes that exceed a single Seedance generation. This is the long-form companion to the ai-filmmaking skill — chain them, don't duplicate.
---

# Script → Shot List → Keyframes → Video

This skill converts written narrative into a production plan in three stages:

1. **Breakdown** — parse the script into beats, turn beats into a numbered shot list, and let the **total runtime emerge** from the sum of per-shot durations.
2. **Keyframes** — craft one image prompt per shot. That image *is* the keyframe: the start frame that seeds image-to-video.
3. **Video** — group shots into generation buckets and craft a Seedance 2.0 prompt for each.

## This skill depends on the `ai-filmmaking` skill

Do not reinvent prompt craft here. Read and reuse the `ai-filmmaking` skill for:

- **Character Sheet (its Template 1)** — build one per recurring character; this is the identity anchor.
- **Character-lock rule** — reuse each character's description *verbatim* in every prompt. Drift across many shots is the #1 long-form failure.
- **Lean-prompt rule** — short declarative slug lines, never novelist paragraphs.
- **`@image` numbering** — each reference gets a unique number; duplicates fuse characters.
- **Seedance prompt formats (its Template 3, Variants A/B/C)** — the per-generation video prompt uses these.

This skill adds the layer that one is missing: the macro-structure that holds a multi-minute film together.

## Core principle — pacing drives length

The user should never have to pre-declare a runtime. Length is an **output**: break the script into shots, give each shot a screen duration based on what it's doing, and sum them. Fast pacing = many short shots; slow pacing = fewer, longer holds. Always show the user the arithmetic so they see *why* the runtime is what it is.

---

## Stage 1 — Script → Shot List

### Step 1: Parse into beats

A **beat** is the smallest unit of dramatic intention — one action, one reveal, one line, one reaction. Walk the script top to bottom and mark each beat. Don't merge unrelated actions into one beat; don't split a single gesture into several.

### Step 2: Convert beats into shots

A **shot** is one continuous camera setup. Mapping is not 1:1:

- A line of dialogue + the listener's reaction is usually **two** shots (shot / reverse-shot).
- A single sustained action may be **one** shot.
- A montage beat may be **many** very short shots.

For each shot, record: **#, framing** (WS / MS / MCU / CU / OTS / insert / POV), **action** (lean slug line — who, where, what happens), **characters present**, **location/time**, and **estimated duration**.

### Step 3: Estimate duration — the pacing engine

This is the load-bearing mechanic. Estimate each shot's screen time by content type:

| Shot content | Duration heuristic |
|---|---|
| **Dialogue** | `words ÷ 2.5` seconds (≈ natural speech rate) **+ 0.5–1s handle** for the breath before/after. A 10-word line ≈ 4.5–5s. |
| **Action (single beat)** | 1–3s for a quick move; 3–8s for a sustained action. |
| **Establishing / scene-set** | 3–6s — let the audience read the space. |
| **Reaction / silent emotional beat** | 1–3s. |
| **Montage / quick-cut** | 0.5–1.5s each, deliberately staccato. |
| **Insert / detail (prop, hands, screen)** | 1–2s. |

These are *defaults read from the script's own tone*. A thriller cuts tight (bias to the low end); a meditative piece holds (bias high). Expose pacing as an override the user can dial:

- **Taut** — scale durations ×0.8, add more cuts.
- **Standard** — as estimated.
- **Contemplative** — scale ×1.3, fewer cuts, longer holds.

State which profile you used and why.

### Step 4: Sum to runtime

Total runtime = Σ shot durations. Present it plainly, e.g. *"42 shots, ≈ 3 min 48 s at a standard cut."* If the user had a target length in mind and the emergent number is far off, tell them which way and offer to retighten or expand pacing — don't silently force the script to fit.

### Shot-list output format

```
SHOT LIST — [title] · pacing: [profile] · total: [N shots / mm:ss]

# | Framing | Action | Who | Loc/Time | Dur
1 | WS      | ...    | ... | ...      | 4s
2 | MCU     | ...    | ... | ...      | 3s
...
```

---

## Stage 2 — Per-shot Keyframe (storyboard) image prompts

Yes — in an image-to-video pipeline the storyboard image for a shot **is the keyframe**: the start frame the video model animates from. So craft one image prompt per shot (not a 9-panel grid per scene — at film length, grids explode into hundreds of panels). Use a 9-panel grid from the `ai-filmmaking` skill only when the user wants a *single designed continuous take* for one scene.

Each keyframe prompt =

- **Character** — the verbatim locked description / `@imageN` from the Character Sheet. Never paraphrase between shots.
- **Composition & framing** — match the shot list (WS/CU/OTS…), describe what's in frame and where.
- **Environment & lighting** — scene-specific light lives HERE, never on the character sheet.
- **Style block** — the one locked look for the whole film (see Style Bible below).
- **Aspect ratio** — state it on *every* shot.

For a shot with large motion, optionally also craft an **end keyframe** for first/last-frame interpolation — but flag that first/last-frame support is model-version-dependent; verify before relying on it.

Deliver each keyframe prompt in its own fenced block, labelled `### Shot N — keyframe`.

---

## Stage 3 — Per-generation Seedance video prompts

### Bucket shots into generations

A Seedance generation has a maximum length (currently 15s for Seedance 2.0 — **verify the cap for the model version in use; do not hardcode it as permanent**). A generation can contain *several cuts*, so a generation ≠ a shot. Group consecutive shots into buckets whose summed duration ≤ the cap, breaking buckets at scene/location changes.

```
GENERATION MAP
Gen 1 | shots 1–4  | 14s | INT. KITCHEN – DAY
Gen 2 | shots 5–6  | 12s | INT. KITCHEN – DAY
Gen 3 | shots 7–9  | 15s | EXT. STREET – NIGHT
...
```

This is what makes long films tractable: a 6-minute piece is just more buckets. The cap stops being a constraint and becomes a packing problem.

### Write each bucket's prompt

Use the `ai-filmmaking` skill's Template 3:

- **Variant C** when both character sheets and a keyframe/grid exist (highest fidelity).
- **Variant B** when a designed grid drives the scene.
- **Variant A** for text-driven buckets with light anchoring.

Within a bucket, the TIMELINE's sub-ranges map to the shot list's shots and their durations. Keep `SUBJECT` / `ENVIRONMENT` / `AUDIO` blocks **identical across all buckets in the same scene** — only TIMELINE and framing change. Default audio to `NO MUSIC` so the user scores the assembled cut. Bake dialogue into the TIMELINE using the parent skill's "He replies / She replies" convention.

Deliver each in its own fenced block, labelled `### Gen N — Seedance prompt`.

---

## The long-form glue (always produce these for 90s+ pieces)

Multi-minute films drift unless macro-state is tracked. Maintain three small artifacts alongside the prompts:

**Style Bible** — one locked block (palette, film grain, lens language, color grade, aspect ratio) reused in every keyframe and every Seedance prompt. Identical prompts still drift in grade across dozens of generations; a fixed block minimizes it and tells the user what to match in post.

**Continuity Ledger** — a table tracking per-scene state that must persist across buckets: location, time of day, wardrobe, prop states, and damage continuity (a cut on a face in scene 2 must still be there in scene 5). The Character Sheet stays neutral; transient state lives here.

```
CONTINUITY LEDGER
Scene | Loc/Time        | Wardrobe        | Carries | Notes
1     | KITCHEN/DAY     | parka, flare    | flare lit
2     | STREET/NIGHT    | parka, hood up  | flare → burned out
```

**Scene/Act Map** — group generations into scenes and scenes into acts, so structure and pacing across the runtime stay visible and the user can see where the climax lands.

---

## Delivery order

1. Pacing read + total runtime, with the arithmetic.
2. Shot list table.
3. Generation map.
4. Style Bible + Continuity Ledger + Scene/Act Map.
5. Keyframe prompts (fenced, per shot).
6. Seedance prompts (fenced, per generation).

If the script is long, offer to deliver in stages (e.g. ledger + shot list first, prompts on confirmation) rather than dumping everything at once.

## Common pitfalls

- **Guessing a runtime instead of deriving it.** Length is the sum of shots, always shown with its math.
- **Hardcoding the 15s cap as permanent.** It's the current Seedance 2.0 limit; verify and parameterize the bucketing.
- **Treating a shot as a generation.** A generation holds multiple cuts; bucket consecutive shots up to the cap.
- **9-panel grids at film length.** Use per-shot keyframes; reserve grids for a single designed continuous take.
- **Letting character or style drift.** Verbatim character blocks + a fixed Style Bible across every prompt.
- **Dropping continuity between scenes.** The Continuity Ledger carries transient state the neutral Character Sheet deliberately omits.
- **Baking scene lighting into the keyframe's character reference.** Scene light goes in the shot prompt, not the identity anchor.
- **Editorializing inside fenced blocks.** Prompts are paste-ready; commentary lives outside.
