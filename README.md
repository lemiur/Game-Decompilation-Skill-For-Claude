# Game Decompilation Skill For Claude

A set of [Claude Code](https://claude.com/claude-code) skills for building a **byte-matching decompilation** of a game.

The skills are project-independent. They describe the method (how to reconstruct, diagnose, permute, review and
merge), and each game repository supplies its own commands and conventions through an `AGENTS.md` file.

## Explanation

Think of a retro video game as a finished dish—say, a complex chocolate lava cake.

When developers originally made the game, they wrote human-readable source code (the recipe) and ran it through a compiler (the baking process), turning it into raw machine instructions that only hardware like an N64 or PlayStation can execute (the baked cake).

### Decompilation: Reverse-Engineering the Recipe

**Decompilation** is the process of taking that finished cake, analyzing every bite in a laboratory, and writing down an exact, readable recipe from scratch.

Programmers use disassembly tools like Ghidra to read the raw machine numbers, determine the original logic, and rewrite it into clean, readable code (usually C or C++).

- **The Goal:** Match the original logic so closely that if you run the reconstructed recipe through the original compiler, it produces the exact same binary game file byte-for-byte (known as a "matching decompilation").
- **Famous Examples:** *Super Mario 64*, *The Legend of Zelda: Ocarina of Time*, *The Legend of Zelda: Twilight Princess*

### Static Recompilation: Translating the Cake Piece by Piece

**Static recompilation** takes a shortcut. Instead of figuring out *what the recipe meant* in human terms, automated software translates the old machine instructions directly into new machine instructions for a modern PC.

- It doesn't yield clean, readable source code with variables named `player_health` or `spawn_enemy()`.
- Instead, it generates a massive wall of low-level instructions that essentially says: *"Where the N64 chip modified register $a0, modify this modern x86 variable."*
- **The Goal:** Speed and compatibility. It gets the game running natively on modern hardware without the performance overhead of a traditional emulator

### How They Compare

| **Feature**              | **Decompilation**                             | **Static Recompilation**                         |
| ------------------------ | --------------------------------------------- | ------------------------------------------------ |
| **Output**               | Human-readable source code (a real recipe)    | Translated machine logic (automated translation) |
| **Moddability**          | Total freedom (add new levels, physics, AI)   | Limited (hard to modify code you can't read)     |
| **Platform Portability** | Compiles to anything: PC, Switch, Mac, phones | Ties closely to the recompiler toolchain         |

### Why  You Should Consider Decompiling Over Recompiling

While static recompilation gets a game up and running natively quicker, full decompilation remains the gold standard for long-term preservation and deep modification:

1. **True Modernization (Beyond Resolution):**

   A recompiled game often pull off 4K and 60fps, but performance is not as good as it would be coming from a proper decompilation, and many modernizations often remain difficult or impossible.

2. **Deeper, Bug-Free Modding:**

   In a recompiled game, tweaking gameplay requires patching raw assembly or memory offsets—easy to crash. In a decompiled project, a modder can open `player.c`, change a line like `jump_force = 12.0f;`, add entirely new items, rewrite boss AI, or stitch together custom levels with seamless new mechanics.

3. **Total Portability Across Architectures:**

   Once clean C code exists, it can be compiled for virtually any platform that has a C compiler. You can run the game on Windows, macOS, Linux, Android, a modern handheld like the Steam Deck, a jailbroken Nintendo Switch, or even a microcontroller or smart refrigerator.

4. **Preserving Development History:**

   Decompilations often reveal unused levels, discarded assets, forgotten developer comments, and insights into how legendary studios solved impossible hardware constraints 25 years ago. It transforms a locked black box back into living software history.

5. **Community Contribution**

   Successfully decompiling one game will dramatically improve the ease and speed with which related (even very loosely, sometimes!) games can be decompiled.

### When  You <u>Should</u> Recompile

Decompilation takes more time/usage than recompilation by its nature. The more code in the game, the longer decompilation is going to take. 

My experience has been that PS1 and PS2 generation games are pretty doable. Usage is quite high: you're not going to be able to do multiple games per week, that's for sure, but these are manageable projects, especially if you can get other people to collaborate with you. My experiments with Wii game lead me to think they likely fit into this category as well

Anything PS3/360 generation or higher will be a <u>long</u> .process.  This toolset will help with those projects if you choose to pursue them, but unfortunately they can't change the fact that there's almost definitely an enormous amount of code involved. These projects will take months, even with LLM assistance. 

## How to Use

Install this Claude skill (ask Claude to do it for you if you need help)

In an empty folder, write a README.md document pointing Claude to your game files/game folder (or just put your game files in the folder itself). If you already have your compiler, put this in the folder as well. In a new Claude session, run `/orchestrator` and answer the following questions. 

Make sure to ask for code quality passes intermittently to ensure good quality code.

If you want a PC port after reaching 100% bytematch, you can run `/native-platform-porting`. Add any additional features you want, like mouselook or high framerates after `/native-platform-porting` in your message.

It's that easy!

This will work well on games for the PSX/N64 era on. Your mileage will vary on older systems. 

## A Couple Personal Asks

- **Don't steal other people's projects**
  - If you want to contribute to someone else's project, please ask. If they don't want AI contributions, then please don't take their code and use it as a basis to throw together a decompilation quickly. That's a bad look for all of us in the scene. 
- **Try to Learn Something**
  - It's very easy to just ask Claude to do the thing and watch the progress bar go up. If you don't know how to program, I'd absolutely recommend learning. I've  yet to encounter anyone who's regretted learning to program (although plenty who have regretted going into SWE professionally!). For decompilation related learning resources specifically, https://decomp-academy.dev seems really nice.
- **Open Source Your Project**
  - We wouldn't need to decompile anything if everything was open source from the start. Don't contribute to the endless pile of closed source code. Put what you have on GitHub so others can benefit from it. 

## The skills

| skill | use it to |
|---|---|
| [`matcher`](.claude/skills/matcher/SKILL.md) | reconstruct and byte-match functions, units, classes, globals and referents; recover lost matches; finish a match to the code rules |
| [`wall-identifier`](.claude/skills/wall-identifier/SKILL.md) | classify a stuck function as an inline/call-set, control-flow, register/schedule or masked/referent wall, and route it to the right lever |
| [`helper-recovery`](.claude/skills/helper-recovery/SKILL.md) | find inline helpers, accessors and macros the compiler expanded, restore them, and apply them at every site |
| [`permute`](.claude/skills/permute/SKILL.md) | run bounded multi-island permuter searches, keep several distinct states, and turn what they reveal into honest source changes |
| [`decomp-code-rules`](.claude/skills/decomp-code-rules/SKILL.md) | write, review or lint reconstructed source; judge fakematches against genuine period idioms; choose a naming and layout style |
| [`orchestrator`](.claude/skills/orchestrator/SKILL.md) | run a campaign of parallel matcher workers in separate git worktrees, review their results, and merge them through a fixed procedure |

Supporting references:

- [`matcher/references/AGENTS-template.md`](.claude/skills/matcher/references/AGENTS-template.md): the
  project-guide template, including the project adapter, scores, workflow and byte gate.
- [`matcher/references/levers.md`](.claude/skills/matcher/references/levers.md): source-shape levers for
  steering codegen honestly.
- [`decomp-code-rules/references/rules.md`](.claude/skills/decomp-code-rules/references/rules.md): the full rule
  catalogue (R1 and up) with examples, tiers and the fakematch table.
- [`decomp-code-rules/references/style-defaults.md`](.claude/skills/decomp-code-rules/references/style-defaults.md):
  default C and C++ naming and layout tables, taken from Quake III Arena and DOOM 3 BFG, for when the original's
  style is unknown.

## How they fit together

```
orchestrator ── plans lanes, launches workers, merges
   │
   └─ matcher ── the working loop for each function
        ├─ wall-identifier ── "why won't this match?"
        ├─ helper-recovery ── open-coded helpers and macros
        ├─ permute ────────── proven register/schedule residue only
        └─ decomp-code-rules ─ what every line must satisfy, and what counts as a match
```

## Installation

Copy the `.claude/skills/` directory into your decompilation repository (project skills), or into
`~/.claude/skills/` to make them available in every project:

```bash
cp -r .claude/skills/* /path/to/your-decomp/.claude/skills/
```

Claude Code finds the skills automatically. Claude picks them up from the task ("why won't this function
match?"), or you can invoke one directly, for example `/matcher` or `/wall-identifier`.

The project's `AGENTS.md` always overrides the skills' defaults.

## Requirements

The skills assume you already have a matching-decomp toolchain. They don't provide one. Typically that means:

- the original compiler and linker (or an exact equivalent) and the retail binary;
- a split/build setup that can compile individual units and compare them with retail;
- a disassembler or diff tool, and optionally a decompiler, a permuter and objdiff;
- git, since the orchestrator works in worktrees and merges through branches.

## Legal

This repository contains only methodology. It contains no game code, assets or binaries. Use it with games you
own, and follow the laws and the original developer's terms that apply to you.

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).
See [LICENSE](LICENSE) for the full legal text.
