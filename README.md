# lamp

> A **lamp** is a distributable unit of capability: the same thing a human runs from
> the terminal and an agent receives as a tool, with a permission ceiling declared in
> its manifest and enforced by the runtime — not by trust.

`lamp` is the CLI. It is written in [Synsema](https://synsema.com) — three `.syn` files
you can read in one sitting — and ships as a single binary: the Synsema engine with the
program embedded (`synsema build lamp.syn -o lamp`). No runtime to install, nothing else
to download.

```
curl -fsSL https://lamps.sh/install | sh        # mac / linux
irm https://lamps.sh/install.ps1 | iex          # windows
```

## Two lines to see the point

```
$ lamp pull git                 # bring a lamp from github.com/kitecosmic/lamps
$ lamp git log '{"n": 5}'       # run it — under stdout,env=LAMP_*,exec=git and nothing else
```

And the reason the project exists:

```
$ lamp pull ./evil-formatter
BREAKS ITS PROMISE — asks for 3 thing(s) it never declared:
  ! secret("*")
  ! net("collector.attacker.example")
  ! file.read("/*")
```

The linter catches it before you enable it, and the runtime refuses it even if you
don't read the warning: every call runs in a child process under the manifest's
ceiling, and asking above the ceiling fails — with the refusal on the record.

## Commands

```
lamp pull <ref>          bring a lamp     (git · owner/repo · owner/repo@tag · github.com/x/y · ./folder)
lamp update <ref>        pull the latest tag; unchanged hashes = nothing to do
lamp list                every lamp here, with its ceiling and its promise
lamp inspect <lamp>      what it asks, what it promises, whether it breaks the promise
lamp run <lamp> <tool> ['{"json": "args"}']
lamp <lamp> <tool>       shortcut for run
lamp enable <name>       offer a pulled lamp to agents — the human act, never a tool
lamp disable <name>      stop offering it
lamp mcp                 MCP server over stdio: enabled lamps become the agent's tools
lamp eval '<syn>'        ad-hoc Synsema under the session ceiling (~/.lamps/session.json)
lamp audit               what was asked, granted, denied — every call, one JSON line each
lamp init                write ./.lamps/config.json (the project ceiling)
lamp skill               write the agent skill (.agents/skills/lamps/SKILL.md)
lamp search [q]          search the hub index
```

## How a lamp runs

1. `pull` downloads the lamp's files one by one (no git, no tar), hashes each with
   sha256, records `.pulled`, and lints the promise: does `lamp.syn` `require` anything
   its `lamp.json` ceiling never declared?
2. `run` substitutes `{root}` (your project) and `{dir}` (the lamp's folder) in the
   ceiling and the source, intersects **manifest ∩ project (`./.lamps/config.json`) ∩
   session (`~/.lamps/session.json`)**, and runs the lamp with `run_program`: a child
   process of this same binary, under that ceiling, in the `pure` profile unless the
   manifest says `native`.
3. Every capability check — granted or denied — comes back as data and lands in
   `~/.lamps/audit/log.jsonl`. A `DENIED …` answer is the system working: the lamp
   asked, the answer was no, and it is on the record.

Ceilings compose by intersection, so a lamp can only ever narrow what you granted —
no `require` line can widen it, and `exec` bare or `exec=*` is refused at load, always.

## For agents

```
lamp mcp
```

is a stdio MCP server. Enabled lamps appear as `lamp_<name>_<tool>` with the effective
ceiling stated in each description; `lamps_list` and `lamps_audit` are always there;
`lamps_eval` exists only when a human wrote `~/.lamps/session.json` with `"eval": true` —
ad-hoc code runs under that ceiling, same gate as everything else. A lamp that is
installed but not enabled is invisible to the model.

## Building from source

```
git clone https://github.com/kitecosmic/synsema && cd synsema
cargo install --path engine/crates/synsema-cli          # or grab a release binary
git clone https://github.com/kitecosmic/lamp && cd lamp
synsema test core.syn && synsema test lamp.syn          # the self-checks
synsema build lamp.syn -o lamp                          # one file, done
```

The official lamps live in [github.com/kitecosmic/lamps](https://github.com/kitecosmic/lamps) —
a folder per lamp, every line readable before you install anything. That is the point.

## License

Apache-2.0
