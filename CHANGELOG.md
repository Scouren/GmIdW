# Changelog

All notable changes to this project are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/). No version has been tagged yet, so
everything is under Unreleased: the newest work first, then the history by date. Schemify embeds
this plugin as its built-in `gmid` at one commit of this repository.

## [Unreleased]

### Documentation (2026-11-14)

- README: a plain description and example up front; Install covers the built-in copy (store
  *Enable*, `builtin = "gmid"`, a project init file) and running a specific commit or a local
  checkout. It no longer tells users to put `plugin` records in a project manifest, which
  Schemify ignores. Added the development commands and a headless check; links to Schemify's
  public spec documents in place of internal references.
- This changelog in Keep a Changelog form.

### Id/W sizing (2026-11-06)

#### Added

- Id/W-driven sizing: a second function family keyed by current density Id/W (A/m) instead of
  gm/Id, on the same tables: `gmid.vgs_jd`, `gmid.gm_id_jd`, `gmid.w_jd`, `gmid.gm_jd`,
  `gmid.gds_jd`, `gmid.gain_jd`, `gmid.ft_jd` (`(dev, jd, l[, id])`) and their `_jd_at`
  variants with |VDS| last. Same names as the gm/Id family with `_jd` before `_at`, same
  argument order, so a docs page switches flow by renaming the call; `gmid.gm_id_jd` is the
  inverse of `gmid.jd`. W in the Id/W flow is `id / jd` directly (`gmid.w_jd`).
- `Lut::point_jd` / `Lut::point_jd_at`: the inverse lookup. VGS by scanning Id/W down from the
  highest VGS (Id/W is monotone in VGS, so a gm/Id dip has one answer per Id/W; the branch ends
  at cutoff or where leakage makes the current rise again), the cell fraction solved in log
  Id/W on the curve the gm/Id lookup walks (exact inverse on grid lengths: 4e-16 on the EKV
  oracle, 1e-16 on the sky130 samples), the Id/W key carried to each L row as `jd·l/l_row`
  between rows (agreement 3.2e-5 on EKV; on sky130 2.6 % nfet, 6 % pfet in gm/Id, within
  1.6 / 6.4 mV in VGS). `Point` gains `gm_id`; `Miss` gains `Jd`.
- README: the two flows, the inverse-lookup method and its accuracy, a 5-transistor OTA input
  pair sized both ways (same W).

#### Changed

- `FN_STRIDE` is 32 (was 16): 29 functions. fn_ids are generation-scoped and never stored, so
  docs pages are unaffected; the module's hash changes, so the cached values in designs are rewritten once.
- A failed simulator run no longer copies the tail of the simulator's output into the log (it
  can quote proprietary PDK model files): the output goes to `.schemify/gmid/work/<dev>.log`
  and the log names that file.

### 2026-11-07

- Changed: the SDK dependencies come from the public Scouren/Schemify-Plugins repository (same
  rev 39da966), and every URL names the Scouren organization (formerly OpenFAT). Module rebuilt.

### 2026-11-06

- Changed: built against Schemify 39da966d, the public SDK commit (`BodyLine::Attr` became a
  struct variant). Module rebuilt.

### 2026-10-29 to 2026-11-02

- Changed: repository references point at the OpenFAT organization; module rebuilt for the new
  dependency path.

### 2026-10-18

- Changed: the Schemify plugin `gmid-lut` became the root crate of this repository, with its
  history; the C++ library and `gmid_runner` CLI moved under `cpp/` as a secondary interface.
- Added: reproducible `build.sh` (host paths remapped, 200 KB limit), CI that checks the
  committed module against a rebuild, `tests/host.rs` loading it into Schemify's real plugin
  host, and a nightly job against Schemify's main branch.
- Changed: Rust 1.95 from rust-overlay, as Schemify builds with.

### 2026-10-10 to 2026-10-12

- Added: the gm/Id plugin. Expression functions `gmid.vgs`, `gmid.jd`, `gmid.w`, `gmid.gm`,
  `gmid.gds`, `gmid.gain`, `gmid.ft` over per-device lookup tables (`.glut`), loaded from
  `gmid.chars`, and characterization with ngspice (**Tools → gm/Id: characterize devices**).
- Added: `_at` variants that query at any VDS on the table's axis; the VGS scan crosses a dip
  in gm/Id, so pfet bins reach higher gm/Id targets.
- Added: `gmid.l(dev, l)`, which clamps a length into the table's range; the `id` attribute
  names the drain-current operating point (`ids` for BSIM-CMG through OSDI).
- Added: characterization with ESPice (`sim espice`), reading terminal currents through 0 V
  ammeters since ESPice has no device operating-point vectors.

### 2026-10-04

- Added: total gate capacitance (Cgg) as MOSFET plot 6 in the C++ tool; `packages.default` in
  the flake (library, headers, `gmid_runner`).
- Fixed: empty PMOS and PNP tables, and a crash on a relative `model_file`.

### 2026-04-11 to 2026-04-14

- Added: the standalone gm/Id visualizer in C++: DC sweeps through ngspice or Xyce and SVG plots
  of the standard figures of merit, with integration tests against ngspice, Xyce and volare.
