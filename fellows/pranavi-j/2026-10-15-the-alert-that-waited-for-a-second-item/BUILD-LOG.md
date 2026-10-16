# BUILD-LOG — The Alert That Waited for a Second Item

## 2026-10-15/2026-10-16 — Full build (both aspects)

**Gate P:** Beat sheet approved 2026-10-15. **FACTCHECK:** resolved same day — both B06 honest
limits (caught by code review not a live incident; the related `source_feed` compatibility gap)
kept at full weight, no softening.

**Audio:** Kokoro `af_bella`, all 9 beats. B01-B08 generated via `generate_audio_kokoro.py`
(measured durations: B01 15.94s, B02 12.5s, B03 16.54s, B04 16.49s, B05 22.9s, B06 23.38s,
B07 10.03s, B08 1.51s — B08 is short because its narration is literally just "Explained with
Claude Code."). B00 is a real silent mp3 (`ffmpeg anullsrc`, 4.05s measured) with
`audio_policy: "silence"` set explicitly per `build_safety.py`'s silent-required-audio gate.
`beat_sheet.json`'s `actual_duration_s` fields updated from the real measured values (not the
pre-audio estimates); total measured runtime 123.34s audio / 123.5s compiled (24fps rounding).

**`scenes.py` (landscape):** authored with the `T()` safe-text helper (Pango/Cairo
small-font-size-artifact workaround) used for every on-screen string, copied from this fellow's
closest sibling, `2026-10-06-the-classifier-that-learned-when-not-to-guess/scenes.py`, for visual
consistency. B03 shows the real `hasData` guard line verbatim from `B6-VERIFICATION.md`
(`const hasData = newItems && newItems.length > 1 && ...`), with the exact substring
`newItems.length > 1` isolated via Manim `Text` character-index slicing (no re-creation, no
paraphrase) and boxed gold/crimson. B04 shows the real one-character-class diff
(`- newItems.length > 1` struck through, `+ newItems.length > 0` added) chained to the real traced
dead end (`fails the check -> has_items: false` -> `If-node TRUE branch: [ ] — nowhere to go`).
B05 shows the real 0/1/2-item before/after harness table, the 1-item row boxed gold as the one
that changed, plus a closing line noting the re-run against the real patched file on disk. B06
shows both honest limits as separate full-weight cards, per house discipline.

**Landscape final:** 3840x2160, 123.5s. GATE V (standalone `final_frame_check.py --lenient`, run
from inside the reel folder, on the true final master): **0 BLOCKER, 0 MAJOR**.

**Vertical (`vertical/scenes.py`):** genuine hand-authored portrait redesign for all 9 beats
(rendered 2160x3840), same `T()` helper plus the portrait `config.frame_width` patch (Manim's CLI
`-r` flag doesn't recompute `frame_width` for portrait otherwise — copied verbatim from the
sibling reel's documented fix). B02's 3-tile horizontal row became a single stacked column; B03's
code line was re-wrapped across more/narrower lines (same verbatim substrings, nothing reworded)
with the same character-index highlight technique; B04's chain was already a single column in the
landscape parent and only needed narrowing; B05's 3-card horizontal row and B06's 2-card
side-by-side row each became stacked columns.

**Vertical final:** 2160x3840, 123.5s (same duration as the landscape master — full-length
companion via `./art vertical`, no beats dropped, no cap, no rewritten outro, no added endcard).
GATE V on the true final master: **0 BLOCKER, 0 MAJOR**.

**Real bugs hit and fixed during this build:**

- **Stale render cache:** `run.sh`/Manim's own `media/` cache does not detect `scenes.py` source
  changes and will silently reuse old renders. Deleted `manim/`, `media/`, `clips/`, `_qc/` and any
  previous slate/final mp4s before every re-render that followed a scene edit, in both the
  landscape reel and the `vertical/` subfolder, and verified every resulting clip's mtime postdated
  `scenes.py`'s mtime via `stat -f %m`/`os.path.getmtime` before trusting any render as fresh.
- **GATE B false TEXT_ON_CURVE on an intentional highlight box (landscape + vertical B03):** B03's
  real `hasData` guard line is boxed around only the substring `newItems.length > 1` (via Manim
  `Text` character-index slicing on the full code line), not the whole line — the box's own edges
  necessarily cross the parent line's centerline, which `manim_layout_audit.py`'s GATE B correctly
  flagged as "label on a curve/line" (`TEXT_ON_CURVE`) under `--curve-strict`. Fixed using the
  gate's own documented escape hatch: `flag_box._qc_intentional = True` (and the same for B04's
  genuine strike-through `Line`, which deliberately crosses the struck code line).
- **GATE V canvas-fill underfill, real (landscape B02):** the real final-cut frame extraction (not
  just the gate's own report — personally re-extracted and inspected frames) found this beat's
  0/1/2 counter composition covering only 45% of the safe area (min 55%) — width was already ~91%
  but height only ~51%, a real vertical underfill from tight tile padding and inter-group spacing,
  not a false positive. Fixed by enlarging the tile padding and the buff between title/tiles/zinger.
- **GATE V canvas-fill underfill from mid-animation timing, real (landscape + vertical B04):** the
  real final-cut frame extraction sampled this beat at its 50% timestamp and found the closing
  caption not yet on screen (it previously landed well past the midpoint of the beat's 16.49s
  duration) — only 41% of the safe area (min 55%), missing the caption's own width/height. Fixed
  by shortening the `self.wait()` calls between the diff, the fails-box and the dead-end box so the
  full composition (including the caption) is on screen before the 50% sample point, in both
  aspects. An arithmetic slip in the first attempt at this fix (recomputing the final `self.wait()`
  remainder) initially made the clip ~1s too long (manim's own compile step reported
  "B04: clip 17.6s center-cut to 16.5s", silently trimming the excess) — caught by independently
  re-summing every `run_time=`/`self.wait()` value in both scene files against each beat's measured
  audio duration before re-rendering, which also confirmed all other beats' timings were exact.
- **GATE B real safe-area overflow (vertical B05):** the post-render layout audit found the second
  closing caption's real bottom edge at y=-3.46, just outside the +/-3.4 safe-area half-height — the
  3-row-card column's height cap (4.6) left no real room for the two independently-chained captions
  below it. Fixed by tightening the cap to 4.3.
- **GATE V false edge-bleed on the review slate (both aspects, every beat):** `./art run`'s
  embedded GATE V check runs against the review slate (`<slug>-slate.mp4`), which burns in a
  per-beat timestamp overlay in the top-right corner for human review. `final_frame_check.py`'s
  `BURN_IN_EXCLUDE` constant only blanks the bottom-left beat-label strip, not this corner
  timestamp — so every single sampled frame was flagged "content crosses the title-safe right/top
  edge," 18/18 frames, identically, across every beat including the simplest title card. Confirmed
  this was the slate's own review chrome (not real scene content) by extracting a raw frame
  directly from the slate mp4 and visually inspecting the corner overlay. Per the toolkit's own
  design (`run.sh`'s comment: "A clean final requires the compiler's mandatory verification... use
  `./art final`"), this check is not meant to gate the review cut — proceeded to `./art final` for
  both aspects, whose own embedded GATE V check (against the real candidate, no slate chrome)
  passed cleanly, and separately re-ran `final_frame_check.py --lenient` directly against both true
  final masters per this build's own requirement: **0 BLOCKER, 0 MAJOR** on both.

**Deliverables confirmed, both renamed and verified:**
- `deliverables/landscape/AlertSecondItemGate_SaiPranaviJeedigunta.mp4` (+ `.verified.json`,
  SHA-256 confirmed matching the compiled master: `74b68e42...`)
- `deliverables/vertical/AlertSecondItemGate_SaiPranaviJeedigunta.mp4` (+ `.verified.json`,
  SHA-256 confirmed matching the compiled master: `5b7644ab...`)

Both final masters were personally re-inspected frame-by-frame (B02's counter and B04's dead-end
chain at both its 50% and 85% sample points in landscape; B05's full before/after table with both
captions visible, and B06's honest-limits cards in vertical) directly from the shipped
deliverables, in both aspects, as a final check before considering this project done — no
text-rendering artifacts, no clipping, no overlap in either aspect.

Publishing not authorized. Nothing inside `brutalist/` was modified; nothing inside `/Users/pranavijs/mycroft` was modified.
