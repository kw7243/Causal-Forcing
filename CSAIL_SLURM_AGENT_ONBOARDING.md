# Remote Machine Usage/ CSAIL Slurm Cluster Usage

This repo may be developed/run on the CSAIL Slurm cluster. Do **not** assume direct GPU access on login nodes.

- Main remote environment: CSAIL cluster-style setup.
- When the user says `ssh into csail` verbatim, open an interactive SSH session to:
  `ssh kwen1@slurm-login-1.csail.mit.edu`
- If `slurm-login-1` is unavailable, try `slurm-login-0` and then `slurm-login-2` with the same fully-qualified domain pattern.
- Do not store the CSAIL password in this file. Password is needed and the prompt is not visible to the user, so ask the user for it in the current session.
- If prompted for Duo, choose option `1` for Duo Push.
- Use Slurm from the login node to get onto GPU nodes.
- If user asks for a generic GPU (ex. H100), only consider allocating from nodes with "torralba" in the name, as those are the nodes from the user's lab
- Before running code, be sure the current GPU node can support it (ex. check memory available, CUDA avail, etc)

## Cluster etiquette

Do NOT modify anything that belongs to the system or login nodes. Isolate any installs to my personal storage units as much as possible. Do NOT touch any directory marked with a different username than "kwen1" (ex. "/data/scratch-fast/kswain"). Refer to storage notes for more details.

### Example

kwen1@torralba-3090-2:/data/scratch-fast/kwen1/Causal-Forcing$ which python3
/usr/bin/python3

This indicates python3 is a system-level thing, so do NOT install any packages on top of this but rather use a separate Python for my purposes. 

## Storage notes

Prefer working under scratch rather than AFS home. Known scratch path used:

```bash
/data/scratch-fast/kwen1
```

AFS home path:

```bash
/afs/csail.mit.edu/u/k/kwen1
```

Avoid installing large environments or VS Code server files into AFS home when possible.

- GPU nodes and login nodes can both see `/afs` and `/data`.
- `/afs/csail.mit.edu/u/k/kwen1` is the persistent home-backed location.
- `/tmp/home/kwen1` may appear as `$HOME` or `~` on some GPU nodes; treat it as node/session-local and not reliably persistent.
- Do not assume `~` means the AFS home on every node.
- Store persistent SSH/GitHub material on AFS, not `/tmp`.
- Current GitHub SSH setup should live under:
  `/afs/csail.mit.edu/u/k/kwen1/.ssh-github`
- SSH config path:
  `/afs/csail.mit.edu/u/k/kwen1/.ssh-github/config`
- GitHub key path:
  `/afs/csail.mit.edu/u/k/kwen1/.ssh-github/github_ed25519`
- Test GitHub auth with:
  `ssh -F /afs/csail.mit.edu/u/k/kwen1/.ssh-github/config -T git@github.com`
- Clone repos onto `/data`, e.g.:
  `/data/scratch-fast/kwen1/Causal-Forcing`
- Clone with:
  `GIT_SSH_COMMAND='ssh -F /afs/csail.mit.edu/u/k/kwen1/.ssh-github/config' git clone git@github.com:kw7243/Causal-Forcing.git`
- After cloning, persist SSH config inside the repo:
  `git config core.sshCommand 'ssh -F /afs/csail.mit.edu/u/k/kwen1/.ssh-github/config'`
- Then normal `git pull`, `git push`, and `git fetch` should work from inside the repo.
- After each fresh login/session, unlock the key once:
  `eval "$(ssh-agent -s)"`
  `ssh-add /afs/csail.mit.edu/u/k/kwen1/.ssh-github/github_ed25519`
- `/data` is for active coding and large working files; less reliable than AFS for persistent config/secrets.

## Correct account / QoS / partition

The user’s valid Slurm account for Torralba Vision resources is:

```bash
--account=vision-torralba-urops-meng
```

For interactive Torralba GPU jobs, the working QoS is:

```bash
--qos=vision-torralba-interactive
```

A known-working partition is:

```bash
--partition=vision-torralba-rtx3090
```

Example of allocation

```bash
salloc \
  --account=vision-torralba-urops-meng \
  --qos=vision-torralba-interactive \
  --partition=vision-torralba-rtx3090 \
  --nodes=1 \
  --cpus-per-task=8 \
  --gres=gpu:1 \
  --mem=32G \
  --time=01:00:00
```

### Useful diagnostics

Show current user’s Slurm associations:

```bash
sacctmgr show assoc user=kwen1 -P
```

Readable version:

```bash
sacctmgr show assoc user=kwen1 format=User,Account,Partition,QOS%100
```

Expected relevant association:

```text
vision-torralba-urops-meng
```

Expected relevant QoS values include:

```text
lab-free
normal
shared-if-available
tig-debug
tig-main
vision-torralba-interactive
vision-torralba-urops-meng
```
