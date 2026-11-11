---
title: "Learning Journal #1: Teaching an AI to Draw Skateboarding Monsters (and Learning What It Can't Do)"
date: 2026-11-11
tags: [ai-art, flux, comfyui, print-on-demand, learning-journal, lessons-learned]
summary: "Two weeks, roughly 1,300 generated images, five failed fixes for one stubborn problem, and a collection of ten stickers. What I tried, what worked, what didn't, and where I'd love advice."
description: "Two weeks, roughly 1,300 generated images, five failed fixes for one stubborn problem, and a collection of ten stickers. What I tried, what worked, what didn't, and where I'd love advice."
ShowToc: true
cover:
  image: "/images/skate-punks-collection-grid.png"
  alt: "Ten cartoon monsters on skateboards, from a robot and a yeti to a mushroom, a skeleton and a vampire queen."
  caption: "The ten published designs."
  relative: false
  hidden: false
---

**Disclosure:** The art in this project is human-directed and AI-made. I choose the characters, write the prompts, review every image by hand, and decide what gets published. An image model draws the pictures. I worked with an AI coding agent (Claude Code) to build the tooling, and with Claude in chat to plan it.

## TL;DR

- I'm building a sticker collection of cartoon monsters on skateboards. The first batch had a problem: every monster was just standing on a board. I wanted tricks.
- Over two weeks I generated about 1,300 images, reviewed them by hand, and ten designs made it into my shop.
- One pose type worked well (standing on the board, wheels mostly hidden). Several never worked (a shove-it, a handplant, a mid-air grab).
- The model's biggest weakness was skateboard wheels and trucks. I tried five different fixes. None of them solved it.
- I tried to train a small custom add-on (a LoRA) on a 12 GB graphics card. It failed three times, so I parked it.
- The most useful lesson wasn't about AI art. It was about who defines "good" when a machine does the counting.
- I have specific questions at the end. If you've solved any of this, I'd love to hear how.

## About this journal

This is a working learning journal for my side projects. I write things down as I go, including the parts that don't work, because the failures teach me more than the wins do. Nothing here is a tutorial or an expert opinion. It's what I tried, what happened, and what I think it means. Where I'm guessing, I'll say so.

## The project

I sell stickers through a print-on-demand shop (RedBubble) under the name Skempies. [See the shop.](https://www.redbubble.com/people/Skempies/shop) The brand look is "rad, 90s, comic-book, a little weird." The first collection was Halloween. The second is Skate Park Punks: ugly-cute monsters (a one-eyed brute, a vampire punk, a skeleton skater, a mushroom, a robot, and more) riding skateboards.

The first batch of Skate Park Punks looked fine, but every character stood on a board in roughly the same stiff pose. I wanted action: ollies, flips, grinds.

### Ground rules I set

- One image model only: FLUX.1 [schnell]. It has a licence that allows commercial use. I avoided FLUX.1 [dev] because its licence doesn't.
- I only sell designs made on my own hardware. I run the model on a desktop with an RTX 4070 (12 GB of video memory).
- Uploads are always manual. I don't use third-party tools to push anything to RedBubble.
- I'm the final judge. No automatic score decides what gets published.

### How the pipeline works (the short version)

```text
prompt  →  image model (schnell)  →  upscale  →  remove background
        →  quality checks  →  automated copyright check
        →  my review (keep / touch-up / reject)
        →  launch review (upload / maybe / no)
        →  title, tags and description  →  I upload by hand
```

Each prompt is built from fixed parts: a character description that never changes, a pose phrase that I vary, and shared style text. Keeping the character text identical meant that when results changed, I knew the pose phrase caused it.

A note on prompts: schnell ignores negative prompts. Telling it "no extra wheels" does nothing, so every fix has to be written as positive description ("one skateboard with four round wheels, two at each end").

## What I tried: poses

I started with five poses: an ollie, a pop shove-it, a manual (rolling on the back wheels), a casual pose carrying the board, and a handplant.

| Pose | What happened |
|---|---|
| Carry the board (chill) | Read clearly right away. |
| Ollie | Read as airborne, but board and wheel details were shaky. |
| Pop shove-it | Looked like an ollie every time. The model couldn't show a spinning board under a still rider. |
| Manual | Riders stood on tilted boards. Almost never a real manual. |
| Handplant | Characters stayed upright. The model ignored "upside down". |

After three rounds of rewording I dropped the shove-it and handplant. Later I tried a mid-air grab (hand on the board, leg kicked out). Out of 36 images, none were usable.

What worked best by far was a plain "cruise" pose: the monster stands on the board, knees bent, board mostly hidden under its feet. About 27% of cruise designs I reviewed got approved, against 4% to 14% for most other poses. My guess, and it's only a guess, is that cruise hides the things the model draws badly: wheels, hands gripping objects, twisting limbs.

![On the left, an orange one-eyed robot with pincer hands standing upright on a neon-green skateboard. On the right, an orange robot in side view with arms spread, riding a skateboard in a failed attempt at a shove-it.](/images/robot-approved-vs-failed.png)

*Left: the "cruise" pose that worked. Right: a pop shove-it attempt that didn't.*

## The wheel problem, and five things that didn't fix it

In my review notes, skateboard wheels and trucks came up more than anything else: missing wheels, three or five wheels, wheels turned the wrong way, mismatched sizes. Here's what I tried.

| Attempt | What I did | Result |
|---|---|---|
| 1. Reword the prompt | Described the board geometry more carefully, named the board once, used a three-quarter view. | Small improvements at best. One round of rewording produced 0 approvals out of 21. |
| 2. Image-to-image refinement | Re-ran the image at low strength, hoping to fix just the wheels. | Changed under 1% of pixels. The wheels didn't change. |
| 3. A second model | Tested a newer licence-friendly model (FLUX.2 klein 4B) in a blind comparison against schnell, with the model hidden from me while I judged. | Roughly a tie (2 of 41 vs 3 of 43 approved). The new model drew wheels better, but did worse elsewhere. |
| 4. Repaint only the board | Used a segmentation model to mask the board, then had the newer model repaint just that area. | The mechanics worked perfectly (nothing outside the mask changed). Quality didn't: 0 of 12 approved. Twisted boards, clipped wheels. |
| 5. A different background remover | Not for wheels, but for a related problem: parts of figures going see-through after background removal. A second model rescued 7 of 31 flagged designs. | Useful, but it made other designs worse if applied blindly. I kept it as an opt-in. |

My conclusion: the wheel problem is a limit of the model, not of my wording. Past about two rounds without movement, I should have changed approach sooner.

## The LoRA attempt (and why VRAM is the real bottleneck)

A LoRA is a small add-on file that nudges a model toward a style or subject. I wanted one trained on my approved designs, hoping it would learn what a correct board looks like.

Generating images needs the model on the graphics card. Training needs the model plus extra numbers for every adjustment, plus optimizer state, plus a large text encoder. That's several times more memory. My card has 12 GB.

I prepared a 30-image dataset (balanced across poses and characters, with 8 images held out for testing) and installed a trainer in an isolated environment. Along the way I:

- moved my display to the motherboard's integrated graphics to free about half a gigabyte of video memory,
- raised the Windows Subsystem for Linux memory limit and added swap,
- patched the trainer twice after reading its code,
- ran training with no network access and a watchdog that stops it if memory thrashes.

Three attempts failed. The last one got through six of seven preparation stages, then ran out of GPU memory just before the first training step. I also saw an unexplained reading (PyTorch reporting about 25 GiB allocated on a 12 GiB card) that I never got to the bottom of.

I parked the LoRA. The options left are a trainer built for smaller cards, or a card with 24 GB. A used card with that much memory costs more than this project can justify.

What I'd tell my past self: read the trainer's code path before spending hours on a run. A dry check that re-implements a loader's logic instead of calling the real function can pass while the real run fails. I learned that one the hard way.

## The reality check: who decides what's "good"?

The pipeline originally had an automatic scorer: a vision model that rated each image from 1 to 10. It gave 10 out of 10 to everything. It couldn't tell a great image from a broken one.

So I reviewed by hand. After the first big batch, the AI agent that helped me build the tools reported 153 of 240 images as "usable." I reviewed 187 of them and approved 3, marked 23 as touch-ups, and rejected 161.

The agent wasn't lying. It was counting designs that read as the right pose and had a clean file. I was counting designs I'd put in a shop. Those are very different standards.

It happened again later. I had 73 approved designs. When I went through them for the actual launch, I marked only 27 as "upload". The bar I used to approve a design while tuning the pipeline was much looser than the bar for selling.

The lessons:

- Define the metric by the person who decides, and don't let the machine grade itself.
- Do a separate launch review before generating more. I almost generated several hundred more images for characters that already had enough good designs.
- Measure an automatic judge's noise before relying on it. My copyright-similarity check stopped one image 1 time out of 10 when I re-ran it on the same file, and only about 8% of its stop reasons named a specific character. It's a rough filter, not a legal clearance, and I treat it that way.

## Getting from "approved" to "published"

After review, a few unglamorous steps turned out to matter:

- **File checks.** I measured every upload file against the sticker spec: size, margins, transparency, stray pixels.
- **Grey patches.** One monster printed with a grey patch between its legs, a leftover piece of background the cutout missed. RedBubble prints transparent areas as white, so I fill those enclosed pockets on copies of the files, only when the colour matches the background and nothing nearby is artwork. (It changed only 2 of my first 27 files, because most candidates were too close to real artwork colours to touch safely.)
- **Titles and tags.** I generate these from templates. I also learned that 15 of my first 27 designs shared a title with another design of the same character, which I'd want to fix.
- **A sync script.** Approved designs get copied (never moved or overwritten) to a folder with a manifest of file hashes, so I can upload from my laptop and always know what's been done.

When I looked at the first seven uploaded designs in my shop's dashboard, the critique was obvious in seconds: four of the seven were the same silhouette, a creature crouched on a low board. In a thumbnail grid they read as one pose in four colours.

## A cheap fix: recolor instead of regenerate

For the last three slots I didn't want more crouches, and generating new poses for those characters had returned about 4% approval. So I wrote a small tool that recolors finished stickers with no generation at all. It selects regions by colour range, changes hue only inside the mask, never touches the black linework or the transparency, and checks that everything outside the mask is bit-identical.

Two results made my shortlist: a neon-yellow yeti on a dark skateboard, and a robot with a neon-green board. My first yeti version had a bug: two cream-coloured toes got recoloured yellow because cream sits close to yellow in hue. The fix was to protect pale colours. The final version was clean.

![Two versions of a furry yeti on a skateboard: the original in blue with a pink board, and a recolor in neon yellow with a dark board.](/images/yeti-recolor-before-after.png)

Recoloring is free of wheel and anatomy problems because the geometry doesn't change. The trade-off: if you list a recolor as a separate design, it needs its own title, or the shop looks like it's padding.

## Numbers, with the caveats

| Batch | Images | Approved by me |
|---|---|---|
| First full run (all characters, 4 poses) | 240 | 3 of 187 reviewed (first pass) |
| New-pose pre-test | 48 | 3 |
| Scaled run 1 | 180 | 27 (15%) |
| Scaled run 2 | 126 | 12 (9.5%) |
| Scaled run 3 | 158 | 18 (11.4%) |
| Push-off and board-at-side batch | 144 | 5 (3.5%) |
| Mid-air grab test | 36 | 0 |

By the end I had 91 approved designs, 27 that I rated as shop-ready, and 10 published.

Please read these numbers skeptically. Many comparisons rest on 12 to 50 images, which is too few to separate a 10% effect from chance. Everything here comes from one model, one set of characters, and one person's taste. I have no sales data yet, because the shop is brand new. I can tell you what the model drew. I can't tell you what will sell.

## Lessons learned

### About the model

- Schnell draws creatures, faces and single objects well. It struggles with countable small parts (wheels, fingers) and with hands gripping things.
- Prompt wording has a ceiling. After two rounds without movement, change the approach.
- Describe what's visible, not the name of the trick.
- A comparison needs paired seeds and enough images, or it's just a story.

### About engineering

- Read the code path before a long job. Make dry checks call the real functions.
- Isolate new tools, back things up, verify with hashes, and test on copies of your real data.
- Set stop rules and limit yourself to one retry attempt per idea. Mine stopped three failed training attempts from becoming ten.
- Escape your output. One unescaped quote in generated HTML hid every card on a page, and no error appeared. Test pages in a real browser.
- Windows' Linux subsystem can shut itself down when idle, taking background services with it.

### About working with an AI agent

- Restate your rules in every prompt, because fresh sessions forget them.
- Ask for a plan, exact commands and a dry run before it touches anything.
- Make it report mistakes plainly. The corrections were more useful than the confidence.
- Several claims, from both me and the AI, turned out wrong once data arrived. For example, I was told ground shadows only existed in raw images and cleanup removed them. In fact 7 of 52 upload files still had them.

## Where I'd love advice

If you've worked on any of these, I'd be grateful for a comment or a message:

- **Wheels, hands and small mechanical parts.** Has anyone found reliable fixes with FLUX.1 schnell or similar models? Prompting, masking, ControlNet-style pose guidance, compositing a separately made object?
- **LoRA training on a 12 GB card.** Has anyone trained a FLUX (especially schnell) LoRA on 12 GB? Which trainer and settings? Is there a known fix for running out of memory while caching text embeddings?
- **Filtering duds automatically.** My vision-model scorer was useless. What do people use to pre-filter large batches before a human looks?
- **Print-on-demand tips.** Do identical or near-identical titles hurt visibility? How many tags and which tag styles matter in practice?
- **Sanity-checking for lookalikes.** How do you check that a design doesn't resemble an existing character, without relying on a weak automatic judge?

Got advice or a correction? Open an issue on the [blog's GitHub repo](https://github.com/salamancerAI/salamancerAI.github.io/issues) or find me on [Dev.to](https://dev.to/salamancerai).

## What's next

- Watch the shop for a couple of weeks and record views, favourites and sales per character and pose, before generating more.
- Plan a Valentine's-season collection with what I learned: pick subjects the model draws well (single creatures and busts), choose poses that hide hands and mechanical parts, and settle colour and board rules before generating.
- Maybe revisit board compositing (a 3D-made board placed under the character) and the LoRA, if I get a bigger graphics card or a better-suited trainer.

If you read this far, thank you. Corrections and suggestions are very welcome.
