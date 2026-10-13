# Token Gesture agent machine

The machine your CTO's agents work on, when you run it yourself.

Token Gesture's CTO plans the work, writes the cards and decides what merges. An agent does the
coding: it claims a card, clones your repository, runs a coding tool, pushes a branch and reports
back. This is the program for a machine you run yourself: a Mac Mini under a desk, a VM, or one
button on a platform. It is the same runner Token Gesture's own machines will run, so a card is done
the same way wherever it is done.

**It connects outward only.** No inbound port, no VPN, no static address. A machine under a desk, a
cloud VM and a platform worker all work the same way, behind NAT.

## One is enough

The platform decides, job by job, whether a machine writes a change or reviews one, so there is no
pair to get right. A second machine, or more slots on one, means more jobs at once and nothing else.

## Two ways to run it

Make a set-up code first, in Token Gesture: **Technology, Settings, Agents**, then **Change** beside
"Runs on", **Your own machines**, name the machine and **Make a set-up code**. It is shown once, works
once, and lasts 24 hours.

### 1. On a platform, with one button

**Best if you do not want to own a machine.**

[![Deploy to Render](https://render.com/images/deploy-to-render-button.svg)](https://render.com/deploy?repo=https://github.com/Token-Gesture/agent-machine)

Render asks for the set-up code as it creates the machine. It runs as one background worker with a
small persistent disk. The disk is not optional: Render wipes the filesystem on every deploy, and
the machine keeps the credential it got for its code there. Without it, the first redeploy leaves a
machine asking for a code that has already been spent.

### 2. In Docker, on a machine you own

**Best if you already run containers**, or want the agents' work kept apart from the rest of the
machine. The image carries `git` and the coding tools, and runs as its own user.

```bash
docker run -d --restart unless-stopped \
  --name tg-agents \
  -v tg-agents-state:/state \
  -e TG_CODE=<your set-up code> \
  ghcr.io/token-gesture/agent-machine:1
```

The volume matters for the same reason Render's disk does.

## What it needs from you

| Variable       | What it is                                                                     |
| -------------- | ------------------------------------------------------------------------------ |
| `TG_CODE`      | The set-up code. Spent once, on first boot                                     |
| `TG_STATE_DIR` | Where the credential is kept. `/state` in the image. Must survive a restart    |
| `TG_SLOTS`     | Jobs at once, 1 to 4. Defaults to 1                                            |
| `TG_URL`       | The platform. Defaults to `https://api.token-gesture.com`; leave it unless told |

No model key and no GitHub token. Each job is handed the ones it needs, for that job only, and they
are gone when it ends.

## Why you never handle a credential

Give it a code and it does the rest: it spends the code on first boot, keeps the credential it gets
in `TG_STATE_DIR` where only it can read it, and starts. The credential can check in and claim your
organisation's waiting work, and nothing else. It is never printed.

Each job it claims comes with a token of its own, good for that one job. The job gets that token,
the platform's address and a `PATH`, and nothing else of the machine's.

## What keeps a job in its place

- **A folder of its own.** Each job runs in a new, empty folder that is its home and its temp, and is
  deleted when it ends.
- **The container is the wall.** In Docker or on Render, a job is inside this machine's container
  and cannot reach the computer it runs on. It is not given the Docker socket.
- **A time limit.** A job is stopped at its agent's time ceiling, whatever it is doing: asked first,
  which hands its work back, then stopped.
- **Removing it is immediate.** Remove the machine in Token Gesture and its credential stops working
  at once. The program says so and exits.

## It worked when

The log says `machine ready: 1 slot(s), process`, and the machine shows as **Ready** in Token Gesture
within a minute. **Set up, never started** means the code was spent and the program is not running.

## Licence

Apache-2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
