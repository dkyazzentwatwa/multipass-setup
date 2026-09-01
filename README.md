# multipass-setup

One command gives every student in the class an identical Ubuntu 24.04 Linux
machine on their Mac. Same packages, same folders, same starting point.

If yours breaks, delete it and run the command again. That is the point.

---

## What this actually is

**Multipass** is a free tool from Canonical (the company that makes Ubuntu). It
runs a real Ubuntu system as a **virtual machine** — a whole computer simulated
in software, running on top of macOS.

**cloud-init** is the standard way Linux machines configure themselves the first
time they boot. The file [`lab.yaml`](lab.yaml) in this repo is a cloud-init
config. It tells the new VM which packages to install and which files to create.
That is why everyone's machine comes out the same.

Two words you will see constantly:

- **host** — your Mac.
- **guest** (or VM) — the Ubuntu machine running inside it.

### Why not just use a browser-based terminal?

Free web terminals are fine for a ten-minute demo. They fall apart in a class:

- **Your work persists.** Stop the VM, close the laptop, come back Thursday —
  your files are still there. Web terminals reset when the tab closes.
- **You get real root.** `sudo` actually works. You can install kernel headers,
  edit `/etc`, break the network, and fix it. Sandboxes block all of that, and
  breaking things is how you learn what they do.
- **It works offline.** No wifi at the venue, no problem. The VM is on your disk.
- **It is disposable.** One command destroys it, one command rebuilds it. There
  is nothing you can do inside the VM that harms your Mac.

---

## 1. Install Multipass

Open the **Terminal** app on your Mac. It is in Applications → Utilities, or
press `Cmd+Space`, type `terminal`, and hit Return.

Paste this and press Return:

```bash
brew install --cask multipass
```

If that says `brew: command not found`, install Homebrew first from
<https://brew.sh>, close Terminal, open it again, then run the line above.

Check it worked:

```bash
multipass version
```

You should see a version number. If you do, you are ready.

> **Windows or Linux?** Download the installer from
> <https://canonical.com/multipass/install>. The `multipass` commands in this
> README are identical once it is installed.

---

## 2. Launch your lab VM

### Option A — the one-liner (do this one)

This downloads `lab.yaml` from this repo and pipes it straight into Multipass:

```bash
curl -fsSL https://raw.githubusercontent.com/dkyazzentwatwa/multipass-setup/main/lab.yaml | multipass launch 24.04 --name lab --cpus 2 --memory 4G --disk 20G --cloud-init -
```

The trailing `-` means "read the config from what was just piped in."

It takes 2–5 minutes the first time — it is downloading an Ubuntu image and
installing packages. It will look frozen. It is not. Wait for it.

When it finishes you will see a line saying the lab VM is ready.

### Option B — download the file first (fallback)

Use this if the one-liner fails, if your network blocks piping, or if you want
to read the config before running it. Which you should.

```bash
curl -fsSL -o lab.yaml https://raw.githubusercontent.com/dkyazzentwatwa/multipass-setup/main/lab.yaml
```

```bash
cat lab.yaml
```

```bash
multipass launch 24.04 --name lab --cpus 2 --memory 4G --disk 20G --cloud-init lab.yaml
```

### About those flags

| Flag | Meaning |
| --- | --- |
| `24.04` | Ubuntu 24.04 LTS, pinned. Never run a bare `multipass launch` — it picks whatever is current and the class stops matching. |
| `--name lab` | The VM is called `lab`. Every command below uses that name. |
| `--cpus 2` | Two CPU cores. |
| `--memory 4G` | 4 GB of RAM, taken from your Mac only while the VM is running. |
| `--disk 20G` | 20 GB of disk. It is allocated as you use it, not all at once. |

> **If your Mac has 8 GB of RAM**, use `--memory 2G` instead of `--memory 4G`.
> Click the Apple menu → About This Mac to check. Handing 4 GB to a VM on an
> 8 GB machine makes macOS itself crawl.

---

## 3. The five commands you need

Run all of these **on your Mac**, not inside the VM.

**Get inside the VM:**

```bash
multipass shell lab
```

Your prompt changes to `ubuntu@lab:~$`. You are now in Linux. Type `exit` to
come back to macOS — the VM keeps running.

**Shut the VM down** (frees your Mac's RAM; your files are kept):

```bash
multipass stop lab
```

**Boot it back up:**

```bash
multipass start lab
```

**See its status, IP address, and memory use:**

```bash
multipass info lab
```

**Destroy it permanently:**

```bash
multipass delete lab --purge
```

### Which of these deletes my work?

Only the last one. **`multipass delete lab --purge` erases the VM and
everything in it, with no confirmation and no undo.** `stop` does not. Closing
your laptop does not. Restarting your Mac does not.

Reach for `--purge` on purpose: when your VM is wrecked and you want a clean
one. Then re-run the launch command from step 2.

---

## 4. Moving files between your Mac and the VM

`multipass transfer` copies files across. The VM side is always written as
`lab:` followed by a path.

**Mac → VM:**

```bash
multipass transfer ./notes.txt lab:/home/ubuntu/lab/notes.txt
```

**VM → Mac:**

```bash
multipass transfer lab:/home/ubuntu/lab/notes.txt ./notes.txt
```

**A whole folder** (note the `-r` for recursive):

```bash
multipass transfer -r ./myproject lab:/home/ubuntu/lab/
```

The `.` in `./notes.txt` means "the folder my Terminal is currently in." Run
`pwd` on your Mac if you are not sure where that is.

---

## 5. What this cannot do

**There is no USB passthrough. None.** Multipass on macOS does not forward USB
devices into the VM, and there is no flag that turns it on.

Concretely, from inside this VM you **cannot**:

- Flash an ESP32, ESP8266, Arduino, or any other board over USB
- Use a Proxmark3, Flipper Zero, or any USB RFID/NFC reader
- Run an RTL-SDR, HackRF, or any other SDR dongle
- Talk to a USB serial adapter, JTAG probe, or logic analyzer
- Read a USB drive you plugged into your Mac

Plugging the device in and running `lsusb` in the VM will show nothing. This is
not a misconfiguration and it is not something the class can fix. It is how the
hypervisor works.

**Hardware work happens on the host — on macOS directly.** Install `esptool`,
the Arduino IDE, `proxmark3`, or `rtl-sdr` on your Mac and use them there.

Use the VM for what it is good at: Linux itself, the shell, scripting, package
management, servers, networking, compiling, and breaking things safely.

Also outside its scope: GUI applications (this is terminal only), and anything
needing GPU acceleration.

---

## Something went wrong?

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md). The image hash mismatch error and
"my cloud-init changes did nothing" are both covered there.

---

## What's in this repo

| File | Purpose |
| --- | --- |
| [`lab.yaml`](lab.yaml) | The cloud-init config. This is what builds the VM. |
| [`README.md`](README.md) | This file. |
| [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) | Fixes for the errors that actually come up. |
| `.github/workflows/validate.yml` | CI. Checks `lab.yaml` on every push so a typo never reaches a student. |

---

## License

MIT — see [LICENSE](LICENSE). Copyright ProsperLift LLC.

---

Build like a hacker. Ship like a pro. Stay legal.
