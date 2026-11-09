# Zellij — Colemak-DH / Dygma Defy

Lock mode is gone. `Ctrl+g` no longer switches modes and nothing can enter LOCKED.

Direction keys are **`n e i o`** = left / down / up / right (Colemak-DH right home
row, index→pinky) in every mode that has direction.

## Always live — no prefix

| Chord | Action |
|---|---|
| `Alt+n` `Alt+e` `Alt+i` `Alt+o` | focus left / down / up / right |
| `Alt+←↓↑→` | same, arrow fallback |
| `Alt+s` | split right |
| `Alt+v` | split down |
| `Alt+[` `Alt+]` | previous / next tab |
| `Alt+1`..`Alt+9` | jump to tab N |
| `Ctrl+Alt+g` | **prefix** — enter/exit PREFIX mode |

`Alt+n` and `Alt+o` use `MoveFocusOrTab`: at the edge of a tab they roll into the
adjacent tab.

## PREFIX mode — `Ctrl+Alt+g`, then:

Status bar shows `PREFIX` (zjstatus) — zellij calls the mode `tmux` internally. Repeatable keys stay in the mode; one-shot keys drop you
back to NORMAL. `esc` or `enter` always leaves.

| Key | Action | Exits? |
|---|---|---|
| `n e i o` | move focus | stays |
| `N E I O` | move the pane in that direction | stays |
| `;` | last-focused pane | stays |
| `tab` | cycle pane focus | stays |
| `space` | next swap layout | stays |
| `r` | → RESIZE mode | stays |
| `m` | → MOVE mode | stays |
| `s` / `d` / `k` | new pane right / down / stacked | exits |
| `t` | new tab | exits |
| `b` | break pane into its own tab | exits |
| `[` / `]` | break pane to left / right tab | exits |
| `x` / `X` | close pane / close tab | exits |
| `f` | fullscreen focused pane | exits |
| `w` | toggle floating panes | exits |
| `p` | embed ⇄ float this pane | exits |
| `y` | pin floating pane on top | exits |
| `z` | toggle pane frames | exits |
| `g` | toggle sync-input on tab | exits |
| `1`..`9` | jump to tab N | exits |
| `H` / `L` | move tab left / right | stays |
| `c` / `,` | rename pane / rename tab | → rename |
| `v` | → SCROLL mode | stays |
| `/` | search scrollback | → search |
| `u` | open scrollback in $EDITOR | exits |
| `?` | this cheatsheet in a floating pane (`q` closes) | exits |
| `a` | session manager | exits |
| `P` / `C` | plugin manager / configuration | exits |
| `D` | detach | — |
| `Q` | quit session | — |

RESIZE: `n e i o` grow, `N E I O` shrink, `+ - =` uniform, `esc` out.
MOVE: `n e i o` move pane, `tab` / `p` cycle forward / back, `esc` out.
SCROLL: `n o` page up/down, `e i` line down/up, `d u` half page, `[ ]` jump prompts,
`c` copy last command output, `/` search, `Ctrl+c` bottom+exit.
SEARCH: `e` next match, `i` previous, `c` case, `w` wrap, `x` whole word.

## Dygma Defy — SuperKey build sheet

Bazecor → **Superkeys** → **+ New Superkey** → assign the 5 actions → then
Keyboard layout → select the thumb key → key picker → **Superkeys** → pick it.

### Pane SuperKey — right thumb

| Bazecor slot | Send | Zellij does |
|---|---|---|
| Tap | `Alt` + `S` | split right |
| Hold | `Alt` + `V` | split down |
| Tap & Hold | `Control`+`Alt`+`F` | fullscreen toggle |
| 2Tap | `Control`+`Alt`+`X` | close pane |
| 2Tap & Hold | `Control`+`Alt`+`W` | toggle floating panes |

### Tab SuperKey — left thumb

| Bazecor slot | Send | Zellij does |
|---|---|---|
| Tap | `Alt` + `]` | next tab |
| Hold | `Control`+`Alt`+`T` | new tab |
| Tap & Hold | `Control`+`Alt`+`R` | rename tab |
| 2Tap | `Alt` + `[` | previous tab |
| 2Tap & Hold | `Control`+`Alt`+`K` | close tab |

### Prefix key — any third thumb key (not a SuperKey, a plain assignment)

`Control` + `Alt` + `G`

Notes:
- In Bazecor, "Alt" is left-Alt. Do not use AltGr — macOS maps it to right-Option
  and Ghostty's `macos-option-as-alt = true` covers both, but left-Alt is the
  tested path.
- Lower the Superkey tap/hold timeout to ~180–200 ms in Bazecor's advanced
  preferences, otherwise Tap-for-next-tab feels laggy while it waits for a 2Tap.
- The two SuperKeys are on opposite thumbs so their double-tap windows can never
  overlap.

## Status bar hints

Two surfaces, because the built-in bar cannot be configured.

**1. Mode hint line (zjstatus).** `layouts/default.kdl` replaces the built-in
status bar with `plugins/zjstatus.wasm` (dj95/zjstatus v0.25.0, requires
zellij >= 0.45.0). The bar's entire text is keyed off the current mode, so each
mode spells out its own keys:

| Mode | Bar reads |
|---|---|
| NORMAL | `Alt n/e/i/o focus · Alt s/v split · Ctrl+Alt+g prefix` |
| PREFIX | `r resize · m move · s/d split · t tab · x close · f full · ? help` |
| RESIZE | `n e i o grow · N E I O shrink · +/- all sides · esc done` |
| MOVE | `n e i o move pane · tab cycle fwd · p cycle back · esc done` |
| SCROLL | `e/i line · n/o page · d/u half · / search · y editor · esc done` |
| SEARCH | `e next · i prev · c case · w wrap · x whole word · esc done` |

Permission: already granted. The prompt renders *inside* the status-bar pane,
which never takes keyboard focus, so `y` would land in your shell instead. It was
granted out-of-band by loading the plugin once in a focusable pane; the grant now
lives in

    ~/Library/Caches/org.Zellij-Contributors.Zellij/permissions.kdl

    "/Users/q/.config/zellij/plugins/zjstatus.wasm" {
        ReadApplicationState
        ChangeApplicationState
        RunCommands
    }

If the prompt ever returns (moved .wasm, cleared cache), re-grant with:

    zellij -l /tmp/permgrant.kdl   # a layout whose only pane is the plugin

then press `y` in that pane and close the session. Do not combine `-s` and `-l`:
zellij treats that as attach-to-existing, not create.

Why not the built-in bar: its hints are hardcoded per mode in
`default-plugins/status-bar/src/second_line.rs`. NORMAL mode shows a rotating
tip and never keybindings; TMUX/PREFIX mode renders a fixed 9-item list that has
no slot for resize or move. RESIZE and MOVE modes do derive their hints from the
real keymap, so those two were already correct - zjstatus just makes the other
modes match.

To revert: delete `~/.config/zellij/layouts/default.kdl`.

**2. Full cheatsheet - PREFIX then `?`.** Opens this file in a floating pane via
`bat --paging=always`, closes on `q`. Bound in the `tmux` block of config.kdl.

## Which physical modifier is "Alt" on macOS

`Alt` means **Option**, not Control.

    Bazecor "Alt" -> HID Left-Alt -> macOS Option -> terminal option-as-alt -> zellij Alt

| Bazecor label | macOS | Use |
|---|---|---|
| `Alt` | left Option | yes |
| `AltGr` | right Option | works, but left Alt is the tested path |
| `OS` | Command | no — never reaches the terminal as Alt |
| `Control` | Control | yes, for the `Ctrl+Alt+g` prefix |

Verified on this machine: Defy enumerates as vid `13807` pid `18`; Karabiner's
selected profile has no rule for that device and no global modifications, and
macOS has no modifier remap. Defy modifiers pass through clean.

Karabiner *does* remap the built-in Apple keyboard (`1452:591`) with
`left_control -> left_command`, so left Control does not exist there. Typing the
prefix off-Defy needs right Control plus the key labeled Command (Karabiner turns
it into Option).

## Zed: Ctrl+Alt chords need a shim (Zed <= 1.17.x)

Zed's terminal drops `ctrl+alt+<letter>` before it reaches the pty. In
`crates/terminal/src/mappings/keys.rs` (v1.17.2, line 213) the ESC-prefix branch
fires only when the modifier set is **exactly** Alt:

    modifiers == TerminalModifiers::Alt && keystroke.key.is_ascii()

Ctrl+Alt matches no branch, `to_esc_str` returns `None`, nothing is written.
Ghostty encodes it correctly, which is why the same key works there.

Upstream fix: zed-industries/zed#62891, merged 2026-08-19 - NOT in v1.17.2
(released 2026-08-26 off an earlier branch). Verified via GitHub compare: the
commit is not an ancestor of the v1.17.2 tag.

Workaround, in `~/.config/zed/keymap.json` under `"context": "Terminal"` -
send the bytes a real terminal would (ESC + the control byte):

| Chord | SendText | Bytes |
|---|---|---|
| `ctrl-alt-g` | `\u001b\u0007` | ESC BEL |
| `ctrl-alt-f` | `\u001b\u0006` | ESC ACK |
| `ctrl-alt-x` | `\u001b\u0018` | ESC CAN |
| `ctrl-alt-w` | `\u001b\u0017` | ESC ETB |
| `ctrl-alt-t` | `\u001b\u0014` | ESC DC4 |
| `ctrl-alt-k` | `\u001b\u000b` | ESC VT |
| `ctrl-alt-r` | `\u001b\u0012` | ESC DC2 |

Delete the shim block once `zed --version` reports 1.18 or later.

Not affected in 1.17.2: plain `Alt+<key>` passes the exact-Alt check with
`is_ascii()` (not lowercase-only), so `Alt+n/e/i/o/s/v`, `Alt+1..9` and
`Alt+[` / `Alt+]` all transmit.

## Encoding rules — read before editing config.kdl

Terminals send `Alt+X` as `ESC` + the byte for X. Some bytes are control-sequence
introducers, so those chords are unusable:

| Form | Safe? |
|---|---|
| `Alt` + lowercase / digit / punctuation below `@` | safe |
| `Alt` + UPPERCASE, or `[ ] \ ^ _ @` | **C1 introducer** (CSI/OSC/SS3) — avoid |
| `Ctrl+Alt` + `a b e f k l n p r t w x` | safe |
| `Ctrl+Alt` + `i m j h [` | = Tab / Enter / LF / Backspace / Esc — avoid |
| `Ctrl+Alt` + `q s z c d v y o` | = XON/XOFF/SUSP/INTR/EOF/LNEXT/DSUSP — avoid |
| `Ctrl+Shift+…`, `Super+…` | needs the Kitty keyboard protocol. Ghostty yes, Zed unverified |

`Alt+[` and `Alt+]` are the one deliberate exception: they are C1 introducers, but
zellij ships them by default and its parser resolves them by timing. If next/prev
tab ever misfires, move them to `Alt+,` / `Alt+.` — that costs zsh's
`Alt+.` yank-last-arg, which you can relocate with
`bindkey '\e/' insert-last-word`.

## Reserved vs freed

Zellij captures 28 chords in NORMAL mode: 21 `Alt+…`, 6 `Ctrl+Alt+…`, and the
prefix. **No plain `Ctrl` chord is captured in NORMAL mode at all.**

Returned to your shell / vim / Claude Code: `Ctrl+b f g h n o p q s t`.

## Host terminal requirements

| Host | Setting | File |
|---|---|---|
| Ghostty | `macos-option-as-alt = true` | `~/.config/ghostty/config` |
| Zed | `"option_as_meta": true` | `~/.config/zed/settings.json` → `terminal` |

Cost of both: Option-composed characters (å ∂ ƒ ©) no longer type in terminals.

## Verify

```sh
zellij setup --check          # config parses
zellij kill-all-sessions      # old sessions keep the old keybinds
```
Then in a fresh session press `Ctrl+Alt+g` — the status bar must flip to `TMUX`.
If nothing happens, the host terminal is eating Option; recheck the table above.
