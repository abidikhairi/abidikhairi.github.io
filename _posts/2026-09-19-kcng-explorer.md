---
layout: post
title: "KCNG Explorer: Probing a Protein Language Model on the 'Silent' Kv6 Channels"
tags: [protein-language-models, bioinformatics, gradio]
---
I just shipped [KCNG Explorer](https://huggingface.co/spaces/khairi/KCNG-Explorer),
a small research tool for poking at the **KCNG** subfamily of voltage-gated
potassium channel subunits — KCNG1 through KCNG4, better known as Kv6.1–Kv6.4.
They're an odd corner of the Kv family: on their own they don't form a
functional channel at all. They're "silent" subunits that only work as
heterotetramers with Kv2, modulating its gating rather than conducting current
by themselves. That makes them a nice, self-contained test case for asking
what a protein language model actually understands about a sequence.

## What it does

KCNG Explorer wraps a pretrained ESM Cambrian (ESMC) masked language model
around a focused workflow:

- **Load a sequence** — fetch directly by UniProt accession, or paste your own
  FASTA.
- **Score every point mutation** — for the loaded sequence, the app computes a
  full log-likelihood-ratio (LLR) matrix over all 19 alternative residues at
  every position, so you get the model's implied fitness landscape for the
  whole protein in one pass.
- **Cross-check against real variants** — those LLR scores are joined against
  UniProt's annotated natural variants (skipping any where the wild-type
  residue doesn't match the given sequence, e.g. a mismatched isoform), so you
  can see whether the model's notion of "tolerated" vs. "disruptive" lines up
  with what's actually observed in nature.
- **Look at the structure** — the same protein is rendered in 3D, preferring
  an experimental PDB structure via UniProt's cross-references and falling
  back to an AlphaFold DB prediction (fetched live from AlphaFold's API,
  since model versions get republished over time) when no experimental
  structure exists.

Because the mutation matrix is `19 × sequence length` model calls, it's cached
per-session and only recomputed when the active sequence actually changes —
everything else (variant table, heatmap, structure viewer) just re-renders off
whatever's already in state.

## From Streamlit prototype to a Gradio Space

The app started life as a Streamlit prototype and got ported to Gradio so it
could be deployed as a Hugging Face Space with GPU access (`@spaces.GPU`,
ZeroGPU-backed). Both UIs still live side by side in the repo, following the
same state-resolution pattern — a sidebar action resolves to
`(sequence, accession, matrix, matrix_sequence)`, and a single render function
turns that into every panel. Keeping resolution and rendering separate is what
let the migration be mostly a matter of swapping `st.session_state` for
`gr.State` rather than rewriting the app logic.

The less glamorous part of shipping it was the dependency graph. `esm >= 3.4.0`
is a hard floor for the ESMC wrapper API this app needs, which drags in
`transformers < 5.0`, which caps `huggingface-hub < 1.0` — and Gradio raised
its own `huggingface-hub` floor above that starting at 6.18.0. So the Space is
deliberately pinned to `gradio==6.17.3`, the newest release where that
three-way constraint is still satisfiable at all. Fun way to relearn that
"just bump the version" isn't always an option.

## Try it

The Space is live at
[huggingface.co/spaces/khairi/KCNG-Explorer](https://huggingface.co/spaces/khairi/KCNG-Explorer) —
load any KCNG1–KCNG4 accession and see where the model agrees (or disagrees)
with nature on what this "silent" channel family can tolerate.
