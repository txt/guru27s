<p align="center">
  <a href="https://github.com/txt/guru27s/blob/main/README.md"><img 
     src="https://img.shields.io/badge/Home-%23ff5733?style=flat-square&logo=home&logoColor=white" /></a>
  <a href="https://github.com/txt/guru27s/blob/main/docs/lect/policies.md"><img 
      src="https://img.shields.io/badge/Policies-%230055ff?style=flat-square&logo=openai&logoColor=white" /></a>
  <a href="#"><img
      src="https://img.shields.io/badge/Teams-%23ffd700?style=flat-square&logo=users&logoColor=white" /></a>
  <a href="#"><img 
      src="https://img.shields.io/badge/Lectures-%238a2be2?style=flat-square&logo=panopto&logoColor=white" /></a>
  <a href="#"><img 
      src="https://img.shields.io/badge/Moodle-%23dc143c?style=flat-square&logo=moodle&logoColor=white" /></a>
  <a href="#"><img 
      src="https://img.shields.io/badge/Chat-%23008080?style=flat-square&logo=discord&logoColor=white" /></a>
  <a href="https://github.com/txt/guru27s/blob/main/LICENSE"><img 
      src="https://img.shields.io/badge/©%20timm%202027-%234b4b4b?style=flat-square&logoColor=white" /></a></p>
<h1 align="center">:cyclone: guru <br>NC State, Spring '27</h1>

# TUIs

A TUI is a state machine over a keystroke stream. Every principle below
follows from taking that sentence literally.

Two audiences read what you build: people at a terminal, and other
programs (including AIs) pulling your code apart for the useful bits.
Design for both.

## Screen layer

- **Keystroke cost ∝ frequency.** Top ten operations get three keys or
  fewer; the rest can be long. This repo's `,/push` beats `make push`
  for exactly this reason — a directory named `,` makes the prefix two
  keys, and `,/sh` puts `,` on `PATH` so inside that shell it is zero.
- **State visible, not remembered.** Nobody should ask "what mode am I
  in?" If the answer isn't on screen, the screen is wrong.
- **Modeless by default.** If modal, the mode name is always displayed.
- **`?` and `help` always live.** Self-listing beats a hand-maintained
  list: `,/help` scans `,/` for each file's `## ` line, so adding a
  command adds its own help.
- **Degrade.** No color, no unicode, 80x24, non-tty output: still works.
- **Under 100ms, or show progress.** No silent pauses.
- **Reversible by default.** Confirm only what cannot be undone. Offer a
  dry-run flag.
- **Fail loud.** Exact error text, real exit codes.
- **Zero config to start.** Config file optional, never required.

## Scavenge layer

The unit of theft is one file. Assume someone lifts a single file out of
your project and runs it elsewhere. Make that work.

- **One file, one job, runs standalone.** No shared mutable globals, no
  initialization order to get right.
- **Doc in-file, machine-greppable.** One convention, two consumers: the
  `## ` line is help text for humans and metadata for `,/help`.
- **Convention over registry.** Discovery by scanning, not by a central
  index that drifts out of date.
- **Plain standard library.** Stolen code must run unchanged. A
  framework dependency makes the file unstealable.
- **Pure core, thin shell.** Logic separate from screen-drawing. The
  logic is the useful bit; keep it liftable.
- **Text in, text out.** Every function reachable from the TUI is also
  reachable non-interactively. No TUI-only path to anything.
- **Line-oriented plain text data,** stable field names. Greppable,
  diffable, pipeable.
- **Names are search keys.** Name things for what someone will grep.
- **Idempotent; print what changed.**
- **Tests beside the code,** runnable alone — a spec that travels with
  the stolen file.
- **Keep each file small enough to read whole.** An AI steals what it
  can read whole, and this one constraint forces most of the rules
  above. "Small" is measured in live variables and branches, not lines
  — see below, because the usual line-count rules do not survive
  contact with the evidence.

### One screen, with docs in the file

Budget the code, not the bytes. One screen means the logic you must hold
in your head at once; documentation does not count against it. That is
not a loophole, because:

- **Docs buy code lines back.** A named invariant at the top of the file
  replaces three defensive `if` statements. A usage example replaces an
  essay on argument parsing. A doc that does not shorten the code is
  decoration — cut it.
- **One block, at the top.** Long documentation goes in the module
  docstring, not sprinkled through the body. The reader scrolls past it
  once, then reads uninterrupted logic. An editor folds it; a program
  strips it with one pattern.
- **Doc lines that execute.** Doctests, and `--keys` examples in the
  usage text, are documentation and test and specification at once.
- **Doc longer than code is a smell with a fix.** It means the prose is
  really a lecture. Move it to `docs/lect/*.md` and leave a one-line
  pointer. The file stays thin and the prose gets room to be good.
- **Two columns.** `pycco` renders comment beside code, so the budget is
  per paragraph of code, not per file.

The real target is that the whole control flow fits in one head.

### What "small" actually means, and what nobody has shown

Be careful here: the famous numbers are weaker than their reputation.

- The one durable industrial rule is the Linux kernel's, and it is
  *relative*: the allowed length of a function shrinks as its complexity
  and indentation depth grow, so a long simple `case` statement is fine
  while a deeply nested function is not. It also caps local variables at
  roughly five to ten. Older versions of that text said "one or two
  screenfuls" of an 80x24 terminal, which is a hardware artifact of 1978
  and not a result about people. Current versions dropped the number.
- The strongest empirical study points the *other* way. Hatton (1997)
  found defect density lowest for components in the low hundreds of
  lines, not the low tens, and explained it by working memory: a
  component that fits the short-term cache uses it efficiently.
  Replications then weakened even that — several found no size to
  fault-density relationship at all, which is what you would expect if
  faults are simply linear in size.
- The memory figure behind all such arguments is about four chunks
  (three to five), not Miller's famous seven, and it is still disputed.
  The unit that matters is the *chunk*, and a chunk of code is not a
  line: a forty-line table initializer is one chunk, while a twelve-line
  nest of conditionals is five.

So count live variables and branches, not lines. That at least has a
mechanism behind it, and it matches the only rule industry actually
follows. Any specific line budget in this course is a house convention
for keeping files stealable — useful, but not a finding. Do not cite it
as one.

## Testable TUI

Terminal binding is why most TUIs go untested. Break the binding:

- **Three pure pieces:** `key -> reduce(state, key) -> state`, then
  `render(state) -> text`. Reducer and renderer are pure and never touch
  a terminal. The terminal driver is twenty lines you never test.
- **Input is data, not typing.** `tui --keys 'jjdq'`, or keys on stdin.
  If a session cannot come from a file, it cannot be tested. This single
  rule buys scripting, tests, fuzzing and bug repro at once.
- **Render to a string.** Golden/snapshot files; the diff is the oracle.
- **Inject every source of nondeterminism:** clock, random seed, cwd,
  environment, terminal size. A hidden `time.now()` is a flaky snapshot.
- **Record and replay.** Log keys plus a per-step state hash. A bug
  report becomes the key log: free repro, and it seeds the fuzzer.
- **Headless exit.** Run the keys, print final state as parseable text,
  let the exit code carry the assertion.

## Input generation

Keys are a small alphabet, so generating strings over it is cheap — and
it finds the sequences humans never type.

- **Random walk first.** Uniform random keys, ten thousand sequences.
  Catches most crashes for about twenty lines of code.
- **Model-based next.** Write the spec state machine (tiny, no
  rendering). Walk both, compare. That is a metamorphic oracle: it needs
  no expected output.
- **Shrink, always.** A 400-key failure is useless; shrink it to five
  keys or do not bother. Delta-debug: drop halves, then singles, keep
  dropping while it still fails.
- **Mutate real logs.** Your record/replay corpus plus mutation beats
  uniform random — it reaches deep states a random walk never enters.
- **Sweep the hostile dimensions:** sizes 1x1, 80x24, 500x3; wide
  characters, emoji, combining marks; no-color; non-tty stdout; EOF
  mid-session; window resize mid-render.
- **Coverage means (state, key) pairs tried,** not lines executed. Line
  coverage lies about state machines.

## Invariants

Random input has no expected output, so the oracle has to be a property
that holds for every state. This list is the real asset — it is the
specification, and it survives being pulled out of your code.

- `render` never throws, for any reachable state.
- Rendered output fits the declared width and height.
- The cursor is always within bounds.
- `undo(do(s)) == s`.
- `deserialize(serialize(s)) == s`.
- No key sequence produces a blank screen or a dead end: every reachable
  state has a path back out.
- Quit always quits, from any state.

Honest failures here are findings. A reported broken invariant scores;
a hidden one costs more.

## Checklist

1. Can a session be driven from a file of keys?
2. Does `render` return a string?
3. Is clock/seed/size injected?
4. Is there a model to compare against?
5. Does the fuzzer shrink?
6. Is the invariant list written down, separate from the code?
7. Can one file be lifted out and still run?

## Does this survive LLMs?

The fair sarcastic question: surely code generation supersedes all of
this? Brooks' 1986 split sorts it. Generators attack *accidental*
complexity — typing, boilerplate, remembering an API. They barely touch
*essential* complexity — deciding what the thing should do, and knowing
whether it does it. Sort every rule on this page into those two bins and
the answer falls out.

### Superseded

- **Keystroke economy.** `,/push` saves *your* hands. An agent types for
  free and does not care. The most obsolete idea on this page is the one
  it opens with.
- **Discoverability by design.** `,/help` listing itself is for people.
  An agent reads the whole directory anyway.
- **"Hold it in your head."** A model has a context window, not four
  chunks. The cognitive-load argument for small files is about the
  *human* reader, so it weakens when the reader is a machine. Small
  files survive for a different reason: a person still approves the
  diff, and review does not scale.

### Stronger than before

Generating code now costs almost nothing. Checking it costs the same as
it always did. So the bottleneck moved into the half of this page you
would expect to be obsolete.

- **The oracle problem got worse.** An agent writes five hundred lines
  of TUI in ten seconds and you still do not know if it is right.
  Invariants, a fuzzer and a shrinker are the only verification that
  scales with generation speed. Everything else you read by hand, and
  now there is far more to read.
- **The invariant list outranks the code.** Code is becoming
  disposable — regenerate it. The invariant list is what you maintain,
  review and argue about. It was the real asset before; now it is the
  whole game.
- **Input as data went from nice to mandatory.** `--keys 'jjdq'` is how
  an agent drives a TUI. With no key-stream interface, an agent cannot
  test your program at all. Same for headless exit codes and text
  output: that is the agent's entire sensory channel.
- **Record and replay.** An agent cannot watch your screen. The key log
  is the only bug report it can act on.
- **Determinism.** A flaky snapshot burns agent loops forever, because
  the agent cannot tell a flake from a bug and will "fix" both.
- **One file, standard library, greppable.** This part was already
  written for machine readers. Unaffected.

### Unknown

- Every size-and-defects result quoted above was measured on
  human-written code. Whether defect density behaves the same way in
  generated code is an open question — the datasets do not exist yet.
  Treat the earlier section as evidence about human code only.
- "Generation supersedes design" has a long losing record: fourth
  generation languages, CASE tools, UML round-tripping,
  model-driven architecture. Each attacked typing. None attacked
  "is it correct?" or "what should it do?". That record is a prior, not
  a proof — this time could differ, and saying so is not an argument
  that it will.

## Provenance

Separate what is evidence from what is taste — the same rule this course
applies to generated exam questions.

- **Evidence, with caveats:** the size to defect-density literature
  (Hatton 1997 and its contested replications); working-memory capacity
  of about four chunks (Cowan 2001, disputed); the Linux kernel coding
  style as observed industrial practice.
- **Craft consensus, not measured:** separating pure logic from
  rendering; input as data; snapshot and golden-file testing;
  record and replay; shrinking failing inputs. These are standard
  practice in property-based testing and terminal-UI tooling, and they
  work, but no study here is being cited for them.
- **House convention:** every specific number on this page, the `,/`
  keystroke argument, and the "unit of theft is one file" framing.
- **Argument, not measurement:** Brooks (1986) on essential versus
  accidental complexity, used above to sort what generators change.
  It is a famous argument that has held up, not an experiment.
- **Open:** whether any of the size-and-defects evidence transfers to
  generated code.

If a claim on this page is going to appear in a quiz, check it first.
