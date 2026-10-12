# agent-watch

A Claude Code mod that shows what your subagents are up to: a progress bar above the prompt box, and a live panel listing every subagent with its model, context, tokens, cost and time.

It's a fork of johnnyvizz's [savvy-progress](https://github.com/johnnyvizz/claude-kit/tree/main/plugins/savvy-progress). savvy-progress only shows its bar for the savvy-flow workflow. agent-watch shows it for any subagent Claude starts, so you get it with Claude's own subagent work.

## What it does

- **The bar.** When Claude starts a subagent, a "Subagents" bar appears above the prompt box. It shows how many have finished out of how many started (`1/3`), a percentage, and a crew button (`×3`) that opens the panel. When the last one finishes it reads Done, and it goes at your next message, or when you press its ✕.
- **The panel.** Every subagent in the session, running and finished, with its model, effort, how full its context is, tokens, an estimated cost and its time. It opens by itself the first time subagents start in a session. After that, the crew button or `/agents-info` opens and closes it.
- **savvy-flow.** Its `savvy-*` workers show in the panel like any other subagent. Its plan bar is another matter: the savvy-flow skill reports its plan to savvy-progress's tools by name, so that bar needs savvy-progress itself (see Install).

The costs are estimates from token counts and a price table in `hooks/register.tsx`, not a bill.

## Install

Mods install from a terminal. The desktop app's Code tab doesn't offer the `/plugin` command, so run these once in PowerShell, Terminal, or any shell where `claude` works:

```bash
claude plugin marketplace add Barncore/agent-watch
```

```bash
claude plugin install agent-watch@agent-watch --scope user
```

Then restart the desktop app, or start a new `claude` session.

If you have savvy-progress installed, turn it off, or you'll get two bars and two panels:

```bash
claude plugin disable savvy-progress@claude-kit
```

The one exception: if you use savvy-flow itself, keep savvy-progress. The savvy-flow skill reports to savvy-progress's tools by name, and agent-watch's tools have their own names.

To update later:

```bash
claude plugin marketplace update agent-watch
```

```bash
claude plugin update agent-watch@agent-watch --scope user
```

Then restart again. A running session keeps the copy it started with.

### What it needs

- **Claude Code 2.1.287 or later,** the release mods arrived in. They're on by default, but local settings, safe mode or an organisation policy can turn them off. See [Anthropic's mod docs](https://code.claude.com/docs/en/plugins/mods/overview#turn-mods-on-or-off).

## Tested on

- **Windows 11, the Claude desktop app's Code tab, Claude Code 2.1.295.** The bar has been seen drawing there. The rest is covered by the tests: the bar's counts, the panel opening once, the shared band, and the batched updates.
- **macOS: untested.** It should work. Mods are JavaScript run by Claude Code itself, and agent-watch touches no files, processes or OS features. But the desktop app draws mods itself, and a few drawing bugs have only been reported on Mac. If you try it on a Mac, an issue saying how it went would help.
- **Terminal:** savvy-progress's text version of the bar and panel is still there, unchanged, but nobody has watched agent-watch in a real terminal yet.

## What changed from savvy-progress

- **Any subagent starts the bar,** not only savvy-flow's `savvy-*` workers.
- **It shares the band above the prompt.** savvy-progress drew its bar on its own, which hid any other mod's rows in that band while it showed. agent-watch draws them above its bar.
- **Live numbers update every 30 seconds.** savvy-progress saved new numbers every second while anything ran, and again on every subagent request. In the desktop app's Code tab, any mod saving state redraws every mod's drawing, and a button press that lands during the redraw gets lost ([anthropics/claude-code#99211](https://github.com/anthropics/claude-code/issues/99211)). So with per-second saves, buttons across every mod got unreliable while subagents ran. agent-watch holds a subagent's numbers and saves them in one batch every 30 seconds, which also moves the panel's clocks. A subagent that finishes is saved straight away.
- **Its own name and tools:** `/agents-info` is unchanged, and the tools are `mcp__agent-watch__progress` and `mcp__agent-watch__step`.

The changes are marked with `agent-watch` comments in `hooks/register.tsx`.

## Settings

`language`: `auto` (the default), `en` or `ru`. `auto` follows Claude Code's `language` setting, then the system locale, and falls back to English.

## What it can reach

From `claude plugin validate` on Claude Code 2.1.295:

```
hooks: session.start, command.run{command=agents-info}, tool.call{tool=mcp__agent-watch__progress}, tool.call{tool=mcp__agent-watch__step}, tool.call{tool=Agent}, agent.spawn, turn.step, turn.complete, prompt.submit, ui.render{component=Pane, requestId=savvy-agents}, ui.render{component=AbovePrompt}
calls: $.clock.after (via startTicking, tick), $.clock.now, $.command.register, $.env.get (via detectLang), $.settings.read (via detectLang), $.state.get, $.state.set, $.tool.register, $.ui.close (via togglePane), $.ui.invalidate (via tick), $.ui.open (via autoOpen, togglePane), $.ui.panes (via togglePane), $.ui.resolve
env reads: LANG, LC_ALL, LC_MESSAGES
```

In plain words: it sees when subagents start and finish, and the token counts of their requests. It reads your settings and locale only to pick the panel's language. It keeps its numbers in the session's own state and uses no store, so nothing carries over into other sessions. It has no file, network or process access.

## Develop

```bash
claude plugin validate .
```

```bash
claude plugin test .
```

## Licence

MIT, see [LICENSE](LICENSE). agent-watch is built on [savvy-progress](https://github.com/johnnyvizz/claude-kit/tree/main/plugins/savvy-progress) by johnnyvizz, also MIT, and his copyright notice is kept.
