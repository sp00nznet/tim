# The Even More Incredible Machine — Static Recompilation

Static recompilation of **The Even More Incredible Machine for Windows**
(Presage Software Development / Dynamix / Sierra, 1993) from its shipping Win16
binary to native C.

Built on the [pcrecomp](https://github.com/sp00nznet/pcrecomp) toolchain, and
directly on [operationneptune](https://github.com/sp00nznet/operationneptune)
and [gizmos](https://github.com/sp00nznet/gizmos) — the two Borland projects
already in the collection.

## Project Status: **P0 complete, P1 not started**

Reconnaissance only. Nothing has been lifted, and nothing builds or runs yet.
What is here is the P0 write-up and `analysis/ne_parse.txt`.

---

## What P0 found

```
TEMIM.EXE   306,688 bytes   NE, Microsoft linker 5.10, Windows, PROTMODE
            34 segments (27 CODE, 7 DATA), 197,832 bytes of code
            4,743 relocations
            imports: KERNEL, USER, GDI  -- and nothing else
```

Built with **Borland C++** (`Borland C++ - Copyright 1991 Borland Intl.` sits in
the string table), version 1.01c.

Three things make this a clean target:

**It imports nothing but the Win16 core.** KERNEL, USER, GDI. No multimedia
DLL, no third-party runtime, no VBRUN. The entire shim surface is the one
`tools/ne/` already generates, and `win16.py`'s PASCAL stack-purge table
already covers it.

**It names its own entry points.** The NE entry table carries four exported
symbols, and they are not decoration — they are the Windows callbacks, which
means the message loop and the dialog procedures are identified before a single
byte is disassembled:

```
TIMWINDOWPROC       seg 8:0x0000    the main window procedure
CONFIRMDLGPROC      seg 7:0x08F6
STATUSDLGPROC       seg 7:0x05EC
DESTROYALLMONSTERS  seg 7:0x0D66
```

**It is Borland, and the collection has two Borland projects that work.**
Operation Neptune is *playable* and ships its own linker map — it is the
calibration target for `disasm/score_recovery.py`. Gizmos & Gadgets runs its
whole attract sequence. Borland's CRT starts up nothing like MSVC's, which is
precisely why having two working references matters more than the third project
being easy.

197 KB of code across 27 segments puts this between Bang! Bang! and El-Fish in
size. El-Fish's 121 segments and 2,236 functions already lifted and linked, so
27 is not the hard part.

## The one complication

The `.PRS` files are the content — `TIM16.PRS`, `TIM256.PRS`, `TIMWIN.PRS`,
2.4 MB across the three. Puzzle definitions are `.TIM` files, small (740 B to
3.4 KB) and there are nine on the disc. Music is plain MID.

The data disk note in the string table is worth reading before assuming this is
standalone:

> The data disk requires the original version of 'The Incredible Machine' in
> order to run.

That is about the *data disk*, not this executable, but it says the file
formats are shared with TIM 1 (1992) — so whatever `.PRS` turns out to be, it
probably reads two games' content, not one.

## Where it goes next (P1)

1. `tools/ne/ne_parse.py` → `ne_decode.py` → `ne_xref.py`. Seed function starts from
   the four named entry points and the relocation targets.
2. Score the recovery. Borland large-model far calls are what
   `disasm/largemodel16.py` was built for on DinoPark Tycoon (3,609 of 3,611
   far calls resolved); check whether this is large model before assuming.
3. Lift with `tools/lift/ne_lift.py` and link against pcrecomp's
   `runtime/win16/`. That runtime and the NE generators (`gen_image`,
   `gen_segments_h`, `gen_win16_stubs`, `gen_unresolved_stubs`) are upstream
   now, so lift → compile → link is a closed loop for a new Win16 target.
4. `.PRS` before anything renders.

## Layout

```
tim/
  original/       win3_TemIM3x.zip
  original/ex/    extracted
  analysis/
  docs/
```

## Credits

The Even More Incredible Machine © 1993 Presage Software Development /
Dynamix / Sierra On-Line. This project neither contains nor distributes any
part of it.

The code and documentation here are MIT; [LICENSE](LICENSE) spells out that
the grant stops at our own work and does not reach the game or anything
lifted from it.
