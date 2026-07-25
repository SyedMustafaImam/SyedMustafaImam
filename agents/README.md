# DevOps Agent Fleet

A set of **persistent, stateful DevOps agents** for Claude Code. Each agent is a
Claude Code *skill* with three things that make it feel like a real teammate:

1. **A persona** (`persona.md`) — a named identity, voice, and set of operating principles.
2. **Persistent memory** (`memory/`) — the agent reads its own history at the start of
   every run and appends to it at the end, so it remembers what it did and why.
3. **A domain protocol** (`SKILL.md`) — how the agent works and its checklist.

Memory is stored **inside each agent's own folder**, in **two formats**:

- `memory/journal.md` — human-readable, dated narrative entries.
- `memory/history.jsonl` — one machine-readable JSON record per action (append-only).

## The fleet

| Agent | Persona | Domain |
|---|---|---|
| `devops-infra-architect` | **Terra** | Terraform / IaC, provisioning, state, drift |
| `devops-cicd-engineer` | **Flux** | GitLab CI / GitHub Actions pipelines |
| `devops-container-ops` | **Helm** | Docker images, Kubernetes, rollouts |
| `devops-cloud-ops` | **Nimbus** | AWS / Azure / GCP / Oracle day-2 ops, IAM, cost |
| `devops-observability` | **Argus** | Prometheus, Grafana, ELK, checkMK, PRTG, SLOs |
| `devops-incident-responder` | **Sentinel** | Triage, mitigation, blameless postmortems |

## Folder layout (per agent)

```
agents/<agent-name>/
├── SKILL.md            # frontmatter + persona summary + domain protocol + memory protocol
├── persona.md          # full persona charter (identity, voice, principles, boundaries)
└── memory/
    ├── journal.md      # append-only Markdown narrative
    └── history.jsonl   # append-only JSON records
```

## How memory works

On **every invocation** the agent, before doing any work:
1. reads `persona.md` and adopts the persona,
2. reads recent `memory/journal.md` entries and tails `memory/history.jsonl`,
3. acknowledges relevant prior context in its persona's voice.

After **completing work**, before ending its turn, the agent appends:
1. one JSONL record to `memory/history.jsonl`, and
2. one dated entry to `memory/journal.md`.

History is **append-only** — past entries are never edited or deleted.

## Installing / using

These agents are mirrored into `~/.claude/skills/<agent-name>/` so Claude Code can
invoke them. Because the environment is ephemeral, this repo copy under `agents/` is
the **durable source of truth** — to persist accumulated memory between sessions,
commit the updated `memory/` files back to this repo.

To (re)install into the live skills directory:

```bash
for a in agents/devops-*; do
  cp -r "$a" "$HOME/.claude/skills/$(basename "$a")"
done
```
