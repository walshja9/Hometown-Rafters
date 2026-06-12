<!-- reasoning_effort: 99 — always use near-maximum deliberation -->

# Hometown Rafters — Project Guide

## What This Is

Hometown Rafters is a fictional baseball franchise universe — a narrative game design project where the player manages a franchise across decades. It's built around branching choice architecture, era-based storytelling, and diegetic systems that express franchise health through world details rather than UI meters.

This is a **design document repository**, not a codebase. The work here is world-building, narrative design, and game systems architecture. Treat it accordingly — the output is prose, structure, and mechanical blueprints, not code.

## Project Structure

```
cities/
└── rhode-island-reds/          # First franchise (complete)
    ├── franchise-bible-complete.md   # Master design doc
    ├── legends/                 # Individual legend profiles
    ├── systems/                 # Mechanical systems (choice tree, Gus Standard, corruption, etc.)
    ├── ownership/               # Ownership lineage and history
    └── research/                # Historical reference material
```

The Rhode Island Reds are the first of the **Original 8** franchises. The remaining 7 are TBD and will follow the same directory structure.

## Core Systems to Know

### The Three Paths
Every franchise has three orientational paths (not moral categories):
- **Roots** — community-first, harder road, deeper loyalty
- **Ambition** — pragmatic middle road, winning and growth
- **Money** — survival by any means, easy short-term, catastrophic long-term

Players drift between paths through accumulated second-tier choices. Momentum builds — reversal gets harder over time.

### The Gus Standard (Rhode Island Reds)
The franchise's soul meter, but **never shown as a number**. Expressed entirely through diegetic world details: the old-timer's posture in the parking lot, the press box tone, whether coffee milk is still in the clubhouse, crowd participation in the Seventh Inning Salute. The player reads the room, not a stat line.

### The Corruption Tax
Mechanical consequences that compound on the Money path. Connected ownership forces roster decisions, bloated contracts, and front office dysfunction that slowly erode the franchise from within.

### Choice Architecture
Each era has one **Top-Tier Choice** (franchise-defining, permanent) and many **Second-Tier Choices** (everyday management decisions that accumulate and create momentum toward a path).

### Legends
Ten fictional players per franchise, tied to specific eras and path conditions. Some appear on all paths, some only on specific ones. Their stories are the emotional backbone of each franchise.

## Writing & Design Principles

- **Voice**: Grounded, specific, literary. Think Lehane not Tolkien. Real places, real textures, real working-class detail. Rhode Island geography and culture are not decoration — they're load-bearing.
- **No abstraction for its own sake**: Every system must be expressible through a concrete in-world moment. If you can't describe how a mechanic feels at the stadium, it doesn't belong.
- **Accumulation over spectacle**: The design philosophy is that small choices compound. The big moments should feel earned or inevitable, never random.
- **Gus is the compass**: For the Reds franchise, every system routes back through Gus Almeida. His presence (or absence) is the emotional readout.
- **Each path must play differently mechanically**, not just narratively. Three distinct management experiences, not three skins on one sim.
- **Historical grounding**: Real Pawtucket/Rhode Island history (mill towns, textile decline, immigration patterns, Portuguese/Italian/French Canadian communities) provides the foundation. Respect it.

## When Working on This Project

- Read the franchise bible first before touching any subsystem doc — it's the source of truth
- New legends, systems, or eras should be consistent with existing choice architecture
- When designing new franchises (Original 8 cities 2-8), follow the established directory structure and design patterns from the Reds
- Simulation/mechanical notes in `systems/simulation-layer-notes.md` are blueprints, not finalized — treat them as starting points
- Cross-reference `research/historical-reference.md` for grounding details
