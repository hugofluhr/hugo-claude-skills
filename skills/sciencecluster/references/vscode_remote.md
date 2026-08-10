# VS Code Remote-SSH via a compute-node tunnel

**When to load:** the user wants to develop/run Jupyter notebooks on
sciencecluster from local VS Code without using the ScienceApps
browser UI, or is debugging `uzh.cluster.node` / `vscode_tunnel`.

## Why this exists

Login nodes are shared and not meant for dev work (see golden rule 1
in the main skill). This workflow runs an `sshd` *inside* a SLURM
allocation on a compute node; VS Code's Remote-SSH then tunnels
through the login node into that job. All indexing, extensions, and
the Jupyter kernel run on the compute node — login-node-safe.

Source: https://docs.s3it.uzh.ch/cluster/vscode/ (adapted — the doc's
`ProxyCommand` example uses double quotes around the remote command,
which makes `$(squeue ...)` expand on the **local** shell instead of
the remote one, since command substitution isn't blocked by double
quotes. Fixed below with single quotes.)

## One-time setup (already done for hfluhr, 2026-08-10)

**`~/.ssh/config`** — two blocks:

```
Host uzh.cluster.cmd
  ...
  ControlMaster auto
  ControlPath ~/.ssh/cm/%r@%h:%p
  ControlPersist 60m

Host uzh.cluster.node
  HostName vscode-tunnel-node
  User hfluhr
  IdentityFile ~/.ssh/id_rsa
  ProxyCommand ssh uzh.cluster.cmd 'nc $(squeue --me --name=vscode_tunnel --states=R -h -O NodeList,Comment)'
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  IdentitiesOnly yes
  PreferredAuthentications publickey
```

`ControlMaster` avoids the login-host connection rate limit being hit
by the polling loop in the start script (see
`references/ssh_multiplexing.md`).

**Cluster side:** `~/slurm_jobs/vscode_tunnel.sbatch` — starts `sshd`
on the allocated node, publishes its port via
`scontrol update JobId=$SLURM_JOB_ID Comment=$PORT`, logs to
`~/logs/vscode_tunnel_<jobid>.txt`. Default: `standard` partition, 4
CPUs, 16G, `--time=4:00:00`, GPU line commented out.

**Local helpers:** `~/dotfiles/bin/cluster-vscode-start [remote-path]`
and `cluster-vscode-stop` (on `PATH` via `~/bin` symlink).

## Day-to-day usage

```
cluster-vscode-start                 # opens $HOME on the cluster
cluster-vscode-start ~/repos/myproj  # or a specific folder
```

This submits `vscode_tunnel.sbatch` if none is running (reuses an
existing RUNNING one otherwise), polls `squeue` until it's RUNNING,
sanity-checks the SSH tunnel, then runs
`code --remote ssh-remote+uzh.cluster.node <path>`.

When done: `cluster-vscode-stop` (cancels the job — don't just close
the VS Code window, the allocation keeps burning walltime/fairshare).

## Jupyter notebooks specifically

1. Open the `.ipynb` — it's on the same NFS home/`/shares` filesystem
   as everywhere else on the cluster, no extra mounting needed.
2. First connection: VS Code prompts to install the **Python** and
   **Jupyter** extensions on the remote. They land in
   `~/.vscode-server/extensions`, which is on NFS `$HOME` — persists
   across job restarts, so this is a one-time cost, not per-session.
3. Select kernel → "Select Another Kernel" → point at
   `<conda-envs>/<env>/bin/python` (the project's cluster conda env,
   not a login-node system Python).
4. The kernel process runs on the compute node the job landed on —
   satisfies golden rule 1 automatically.

## Gotchas

- **Don't `sbatch`/`squeue` from the VS Code integrated terminal on
  the remote.** Per the source doc, job submission from inside that
  terminal is unreliable on this setup — use a normal
  `ssh uzh.cluster.cmd` terminal locally for job management, keep the
  VS Code terminal for editing/running code only.
- **Only run one `vscode_tunnel` job at a time.** If a stale one is
  still RUNNING from a previous session, `squeue --me --name=vscode_tunnel`
  can return multiple `NodeList Comment` pairs, which breaks the `nc`
  call in `ProxyCommand`. `cluster-vscode-stop` before starting fresh
  if unsure, or `ssh uzh.cluster.cmd squeue --me`.
- **Port-not-set race:** right at the RUNNING transition there's a
  brief window before the job's `scontrol update ... Comment=$PORT`
  has landed. `cluster-vscode-start` sleeps 3s after seeing RUNNING to
  dodge this; a manual `ssh uzh.cluster.node` immediately after
  `squeue` shows R may need a retry.
- **4h wall time (default).** Job dies at the limit — no warning
  inside VS Code beyond a dropped connection. Save notebooks
  frequently; bump `--time` in `vscode_tunnel.sbatch` for longer
  sessions (subject to QoS max, see main skill's Constants table).
- **File watcher:** for large data dirs, add excludes in VS Code
  Settings (remote scope) — search "exclude", add patterns like the
  project's data/venv/`.git` dirs — to keep the remote extension host
  responsive.
- **GPU tunnel:** uncomment `#SBATCH --gpus=1` in the sbatch file if
  the notebook needs one; goes through the same `standard`/`lowprio`
  partition tradeoff as any other job (see main skill's partition
  convention — a preempted `lowprio` tunnel just drops the VS Code
  connection, resubmit).
