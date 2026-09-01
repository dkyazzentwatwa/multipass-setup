# Troubleshooting

Run every command marked **on your Mac** in the macOS Terminal. Run every
command marked **in the VM** after `multipass shell lab`.

---

## The one thing to understand first

**cloud-init runs once, on the VM's very first boot.**

Editing `lab.yaml` does **nothing** to a VM that already exists. Not after a
`stop`/`start`, not after a reboot, not ever. The config was consumed at
creation and that machine is done with it.

To pick up a change to `lab.yaml`, you delete the VM and launch a new one. See
[Nuclear option](#nuclear-option) below.

---

## "Image hash mismatch" / the download fails or hangs

Symptoms, on your Mac, during `multipass launch`:

```
launch failed: Downloaded image hash does not match
```

Multipass cached a half-downloaded or stale Ubuntu image. The fix is to throw
the cache away and restart the Multipass service.

**Step 1 — look at what is cached.**

```bash
sudo ls -la /var/root/Library/Caches/multipassd/qemu/vault/images/
```

You will see one or more folders with dates in the name, like
`24.04-20250115` or a long hash. Those are cached images.

**Step 2 — delete the dated folder.** Substitute the real folder name you saw:

```bash
sudo rm -rf /var/root/Library/Caches/multipassd/qemu/vault/images/<DATED-FOLDER-NAME>
```

If you only ever launched 24.04 and you want to clear all of them:

```bash
sudo rm -rf /var/root/Library/Caches/multipassd/qemu/vault/images/*
```

This deletes cached *downloads*, not your VMs.

**Step 3 — restart the Multipass daemon.**

```bash
sudo launchctl kickstart -k system/com.canonical.multipassd
```

Wait about 10 seconds, then run your launch command again. It will re-download
the image from scratch.

> `sudo` will ask for your Mac login password. It shows nothing as you type —
> no dots, no stars. That is normal. Type it and press Return.

---

## The VM launched but the packages/folders are missing

So `git`, `jq`, or `htop` are not there, or `~/lab` does not exist. cloud-init
ran and something in it failed.

**Step 1 — get into the VM.**

```bash
multipass shell lab
```

**Step 2 — ask cloud-init how it went.** In the VM:

```bash
cloud-init status --long
```

- `status: done` — cloud-init finished cleanly. Your problem is elsewhere.
- `status: running` — it is still working. Wait a minute and check again.
- `status: error` — it failed. Keep reading.

**Step 3 — read the actual log.** In the VM:

```bash
sudo cat /var/log/cloud-init-output.log
```

This is long. The useful part is at the end:

```bash
sudo tail -50 /var/log/cloud-init-output.log
```

Look for lines containing `Error`, `Failed`, or `Traceback`. A failed
`apt-get install` usually means the VM had no network when it booted.

**Step 4 — for YAML/schema problems specifically**, in the VM:

```bash
sudo cloud-init schema --system --annotate
```

That points at the exact line of the config it did not like.

Most of the time the fix is just to rebuild the VM — see below.

---

## "Command not found: multipass"

On your Mac:

```bash
brew install --cask multipass
```

Then close Terminal completely, open it again, and try `multipass version`.
The `PATH` does not update in a shell that was already open.

---

## The VM is slow, or my Mac is slow while it runs

You gave the VM too much RAM. On an 8 GB Mac, relaunch with `--memory 2G`
instead of `--memory 4G`.

When you are not using the VM, shut it down and get the RAM back:

```bash
multipass stop lab
```

---

## "instance 'lab' already exists"

You already have a VM with that name. Either use it:

```bash
multipass shell lab
```

Or see everything you have:

```bash
multipass list
```

Or delete it and start over — next section.

---

## Nuclear option

When the VM is wrecked, or you edited `lab.yaml` and need the change applied,
throw it away and build a new one.

**This permanently erases everything inside the VM.** Copy anything you want to
keep out first:

```bash
multipass transfer -r lab:/home/ubuntu/lab ./lab-backup
```

Then, on your Mac:

```bash
multipass delete lab --purge
```

```bash
curl -fsSL https://raw.githubusercontent.com/dkyazzentwatwa/multipass-setup/main/lab.yaml | multipass launch 24.04 --name lab --cpus 2 --memory 4G --disk 20G --cloud-init -
```

Two minutes later you have a clean, identical lab.

---

## Still stuck

Bring these three things to class or paste them in the group chat:

1. The exact command you ran (copy it, do not retype it from memory).
2. The full error message.
3. The output of `multipass info lab` and `multipass version`.
