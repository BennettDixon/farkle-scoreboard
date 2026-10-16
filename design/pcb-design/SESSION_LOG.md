> **Scope:** Compact record of each PCB/enclosure design session (human + AI): who took part, what was read, what was proposed or decided, and what is still open. Follows the project's "vibe coding" practice of distilling every design conversation into markdown in `design/`.
> **Status:** **LIVE LOG** - Append one entry per session, newest last. Decisions recorded here must also be reflected in `MODULAR_PCB_PROPOSAL.md` (or its successor spec).

# PCB & Enclosure Design: Session Log

## Folder Index (`design/pcb-design/`)
| File | Role |
|:--|:--|
| `ELECTRONICS_CURRENT_STATE.md` | Snapshot audit of the hand-wired build: subsystems, pin/power budgets, findings F1–F11 |
| `MODULAR_PCB_PROPOSAL.md` | Proposed board set, interconnect contract, power, certification, enclosure, phasing, open questions Q1–Q9 |
| `SESSION_LOG.md` | This file |

---

## Session 1: 2026-10-16, Kickoff & Architecture Proposal

**Participants:** Bennett Dixon (helping on schematic/enclosure/PCB), Claude (AI, acting as senior EE reviewer). Guillermo not present, so everything below is a **proposal pending his review**.

**Brief from Bennett:**
1.  Follow this repo's design-doc layout and record sessions as markdown.
2.  All changes via PR (from a fork) to `dev`.
3.  Summarize the current electronics by subsystem: compute, screen, gamepad, power, and so on.
4.  Use **certified PSUs and pre-certified wireless** rather than certifying our own.
5.  Propose a set of PCBs. One board is acceptable, but modular is preferred.

**Read:** every file in `design/` (focus on `SCHEMATIC_AND_HARDWARE_GUIDE.md`, `HARDWARE_UPGRADE_MIGRATION_PLAN.md`, `SOUND_EFFECTS.md`, `COMPONENT_LIBRARIES.md`, `wiring.png`), plus `platformio.ini` and the component pin `#define`s.

**Key findings (see `ELECTRONICS_CURRENT_STATE.md` §5):**
*   **F1:** the no-CS ST7789 shares SPI with the MAX7219s and SD, so it sees their traffic. This is a probable contributor to LCD corruption.
*   **F2:** worst-case load is ~5.8 A against a 3 A USB-C source, with no current limiting.
*   **F3:** LED return current shares the logic ground. This is the likely root of the A2/LCD noise (Issue #80).
*   **F4/F5:** the multi-bit buttons need steering diodes, and two-button chords alias to score buttons (FARKLE+CLEAR → +500).
*   Documentation drift: `wiring.png` is V1; there's a Mermaid fence issue; and AGENTS.md says `esp32dev` while platformio only has `uno_r4_wifi`.

**Proposed (not yet decided):**
*   Four custom designs: **PWR, CORE-A (Uno R4 shield), SEG ×3, PAD**. Buy the grid, strip, LCD (with CS), and DFPlayer.
*   Phase A keeps the Uno R4 WiFi and the **existing pin map**, so there's no firmware change. Optional Phase B moves to a module-down ESP32-S3-WROOM-1 behind the same connectors.
*   Certification boundary: external UL/CE USB-C adapter, no internal battery, radio only via pre-certified module. The product still needs FCC Part 15B and, in the EU, RED.
*   eFuse current limit ~2.9 A plus a firmware LED budget ≤2.4 A.
*   Single-board alternative documented and deferred to "Phase C" once the layout freezes.

**Open questions for Guillermo:** Q1–Q9 in `MODULAR_PCB_PROPOSAL.md` §11. The most blocking are Q1 (goal/certification), Q3 (integrated vs tethered gamepad), and Q4 (current diode/CS reality).

**Next session agenda:**
1.  Walk Guillermo through the findings and answer Q1–Q9.
2.  Freeze the §5 interconnect pinout.
3.  Start KiCad schematics for PWR and CORE-A (create `hardware/kicad/`).

---

## Session 2: 2026-10-16, Build Photo Review

**Participants:** Bennett Dixon, Claude. Input relayed from Guillermo: a photo of the working breadboard build, and the answer to Q1.

**Answered:**
*   **Q1:** personal build, not a sales project. Certification becomes a *design-for* goal rather than a gate (proposal §7). We keep the certified-adapter and pre-certified-radio rules because they're cheap and keep a sale possible later.

**Learned from the photo (audit §7):**
*   The score displays are **off-the-shelf chainable 5-digit MAX7219 modules**, so a custom SEG board is no longer needed for Phase A.
*   Power is a **Xiaomi USB-A power bank** (typically rated 5.1 V/2.4 A). That is lower than the 3 A assumed in Session 1.
*   Power and ground go through breadboard rails and Dupont jumpers (new finding **F12**), which reinforces F3.
*   No mute switch, LED grid, or status strip is visible. F1 (LCD CS) and F4 (button diodes) can't be confirmed from the photo.

**Changed in `MODULAR_PCB_PROPOSAL.md`:**
*   Custom boards drop from four designs to **three: PWR, CORE-A, PAD**. SEG reuses the existing modules, and a custom SEG is optional.
*   eFuse limit goes to **~2.5 A** for the power bank (2.9 A with a 3 A USB-C adapter). The firmware LED budget goes to **≤2.0 A**.
*   Note added that CC-based source sensing can't work with USB-A banks.
*   PAD notes the current KY-040 encoder and self-locking switch modules.

**Still open:** Q2–Q9, especially Q3 (integrated vs tethered pad) and Q4 (LCD CS pin count, button diodes). A close-up photo of the LCD header and the underside of the button perfboard would answer Q4.

---

## Session 3: 2026-10-16, Portable Power

**Participants:** Bennett Dixon, Claude.

**Context:** Guillermo likely isn't attached to the current power bank, but portability is probably a feature he wants. Bennett asked whether a pre-certified battery module could be specced in.

**Proposed (`MODULAR_PCB_PROPOSAL.md` §6.1):**
*   **Recommended:** a certified USB-C PD power bank (5 V/3 A, ≥10,000 mAh) **docked** in an enclosure bay and removable. It carries all battery certification itself, and we design no charger or cell circuitry.
*   **Upgrade (PWR-2):** a second "wall" USB-C inlet with a priority power mux, so plugging in takes over without relying on the bank's pass-through charging. Many banks briefly drop their output during pass-through, which would reset the MCU.
*   **Rejected:** certified cells with our own charger and boost (brings in battery-safety design), and bare 18650s or uncertified power-bank boards.
*   Runtime is ~4 h typical on 10,000 mAh and ~8 h on 20,000 mAh.

**Decided:** the build photo is **not** committed to the repo.

**New open question:** Q10 (portability, and A vs B).

