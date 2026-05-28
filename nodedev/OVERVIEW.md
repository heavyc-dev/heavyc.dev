# node.dev — Overview

> Your development machine, in your pocket. A secure SSH terminal and AI coding
> companion for iPhone that connects to your real PC over a private network —
> so you can code, run commands, and drive an AI agent from anywhere.

## What it is

node.dev turns your phone into a first-class window onto your development
machine. It is a polished, mobile-native SSH client built for one job done
exceptionally well: reaching your own computer from anywhere and getting real
work done on it — including delegating that work to an AI coding agent.

The phone is the control surface. Your PC does the heavy lifting. node.dev
keeps the two in sync over an encrypted private link, so the full power of your
desktop — its files, its tools, its compute — is always one tap away.

## The problem it solves

Real development happens on a real machine: your files, your toolchain, your
compute, your environment. But you are not always sitting at that machine. You
are on the couch, on a train, away from your desk, or simply want to check on a
long-running task without opening a laptop.

Existing mobile SSH apps treat the phone like a tiny, awkward desktop terminal.
node.dev takes the opposite approach: it is designed from the ground up for the
phone, with a clean, tactile interface, secure key handling that fits how phones
actually work, and — most importantly — an AI agent that lets you describe what
you want in plain language instead of thumb-typing commands.

## How it works

1. **Install a private network.** node.dev connects to your PC over Tailscale,
   a zero-config encrypted mesh network. Your devices talk directly and
   privately — nothing is exposed to the public internet.
2. **One-time PC setup.** A single guided script installs an SSH server on your
   Windows PC, locks it down to your private network, and accepts a key from
   your phone automatically. No passwords, no manual file editing.
3. **Connect and work.** From then on, your phone has a secure, key-only
   connection to your machine. Open a terminal, browse files, check git status,
   forward a port — or hand the work to Codie, the built-in AI agent.

Sessions are kept alive with tmux, so a dropped connection or a locked phone
never kills your work. Pick up exactly where you left off.

## Key features

- **Full terminal.** A genuine xterm-compatible shell with proper key handling
  and scrollback — not a stripped-down toy. Your real shell, your real tools.
- **Files.** Browse, view, and edit files on your machine over SFTP, straight
  from the phone.
- **Git at a glance.** See the current branch, working-tree status, and recent
  commits without typing a command.
- **Port tunnels.** Forward a port from your machine to your phone to preview a
  local dev server or test an API on the go.
- **Persistent sessions.** tmux keeps long-running jobs and shells alive across
  disconnects, sleeps, and reconnections.
- **Projects as rooms.** Your home screen is organized around projects, each
  grouping its live sessions — so context, not a flat list of connections, is
  what you navigate.

## Codie — the on-board AI coding agent

Codie is what sets node.dev apart. It is a chat-based AI coding agent that lives
on your phone but runs on your machine. You describe what you want in natural
language; Codie carries it out on your actual codebase — reading files, running
commands, making edits, working with git — and reports back in a clean,
conversational thread.

- **Real work on the real machine.** Codie operates on your live project, not a
  sandbox or a copy. The results are real changes you can review and ship.
- **You stay in control.** Codie asks for approval before sensitive actions, and
  you choose the level of access it has per project — from read-only, to edit
  and git, to full autonomy.
- **Transparent.** Each turn shows what it did, how long it took, and what it
  cost, so there are no surprises.
- **Personable.** Codie has an animated character and a conversational voice. On
  supported iPhones, Apple Intelligence runs entirely on-device to make Codie
  sound natural and friendly — purely for the conversation, never to make
  decisions about your code.

The effect is that complex tasks you would normally need a keyboard and a desk
for become things you can start, supervise, and finish from your phone.

## Security and privacy

Security is a foundation, not a feature bolt-on.

- **Private by default.** All traffic runs over your own encrypted Tailscale
  network. Your machine is never exposed to the public internet.
- **Keys never leave the device.** node.dev generates modern Ed25519 SSH keys
  and stores the private key in the iPhone's hardware-backed Keychain, locked
  behind Face ID / Touch ID. The private key is never uploaded or synced.
- **No passwords.** The PC is configured for key-only authentication. Only your
  phone can connect.
- **No cloud, no telemetry.** node.dev has no backend. Your sessions, history,
  and diagnostics stay on your device. Nothing is collected or phoned home;
  crash logs are local and shared only if you choose to.

## Design philosophy

node.dev is built to feel like a high-touch cockpit, not a flat utility. Depth,
motion, and tactility make it feel alive and precise — while always respecting
the system's reduce-motion setting for those who prefer calm. The guiding rule
is that clean and intuitive beats feature-rich: every screen earns its place,
and the result should feel simpler than the power it puts in your hands.

## Who it's for

- **Developers who want their machine with them** — to check on builds, fix a
  quick bug, or kick off a task without sitting down at a desk.
- **People exploring AI-assisted coding** who want an agent working on their
  real projects, with real guardrails, from a device they always have.
- **Anyone who values privacy** and wants a powerful remote-development setup
  that runs on their own hardware and their own network, with no third-party
  cloud in the middle.

## In one line

node.dev is the secure, mobile-native way to reach your development machine and
its AI agent from anywhere — your computer's power and your AI pair-programmer,
in your pocket.
