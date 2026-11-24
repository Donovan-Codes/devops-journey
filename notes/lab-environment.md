# Week 02 — 2026-09-09 to 2026-09-15

**Phase:** 1 — Linux fundamentals
**Hours this week:** ~

---

## Goals for this week

### Carried over from Week 1

- [ ] **Lab 1** — key-based SSH, password auth disabled, `~/.ssh/config` alias working
- [ ] Week 1 log filled in and committed

### Bandit

- [ ] Levels 0–10 complete
- [ ] Notes written for each level in my own words (not the level page's words)
- [ ] Passwords kept in the gitignored file, not in `bandit-notes.md`

### Labs

- [ ] **Lab 2** — create `alice` and `bob`, a `developers` group, and `/srv/shared`
      writable by the group only. Verify by logging in as each user.

### Concepts to be solid on by Friday

- [ ] Octal permissions — say what `644`, `755`, and `600` mean without looking it up
- [ ] The difference between a user, a group, and "other" in `ls -l` output
- [ ] What `sudo` actually does and where its rules live
- [ ] `chmod` vs `chown` — when each is the right tool

### Vim survival

- [ ] Open a file, enter insert mode, type, escape, save, quit
- [ ] `:q!` to abandon changes
- [ ] `dd` to delete a line, `/` to search

  Goal is not fluency. Goal is: I land on a server with no nano and I can still
  edit a config and get out.

### Habits

- [ ] At least 3 commits this week (not one big one on Sunday)
- [ ] Log filled in as I go, not from memory at the end
- [ ] Repo topics added on GitHub

---

## Stretch (only if the above is genuinely done)

- [ ] Bandit 11–14
- [ ] Lab 3 — write a systemd unit file

---

## Built

- Created a private and public SSH key called devbox to login to my remote Ubuntu server without my password.
- 

## Broke

1.  Got a permission denied by not specifying the user when using ssh-copy-id.
2. Couldn't get the password authentication to disable after multiple tries

## Fixed

1. Make sure to do user@hostname at the end of ssh-copy-id command so that it tries to authenticate to an actual user on the server.
2. At the top of my sshd_config file under /etc/ssh/ there was a Include line that superseded the config file edits I made. It was a file called: /etc/ssh/sshd_config.d/50-cloud-init.conf. Had to change the PasswordAuthentication to no because it was still yes and reload to fully disable password authentication.

## Learned

- ssh-keygen -t ed25519 to create a ssh public and private key in ~/.ssh
- When it's prompted to "Enter file in which to save the key:" ... provide the entire filepath of where you want your key to provide.
- You can use "man {command}" in order to see a guide manual on how to use a command properly and see its description.
- "ssh-copy-id -i ~/.ssh/devbox. {VM IP} will copy your ssh key to your remote server. Use the '-n' flag beforehand to see a test-run and what will happen if you were to run it.
- The authorized keys on the remote server are here: .ssh/authorized_keys. When you do an ssh-copy-id command it gets placed here.
- In order to search a config file with vim, use '/' to search forward or '?' to search backwards. The 'n' key can be used to look for any more matches ahead and 'N' can be used to search for any matches behind. Needed this for the lab01 SSH keys so that I could find PasswordAuthentication and PermitRootLogin values.
- When using vim, use 'i' to enter INSERT mode to make your edits and ':wq' to write and quit.

- On Mac OS do a "stat -f '%A %N' ~/.ssh ~/.ssh/*" to pull the file permissions on ssh keys and files.
- sshd -T helped reveal that my Include file at the top of my sshd config file was preventing me from turning off PasswordAuthentication.


## Still fuzzy

-

## Next week

<!-- Fill in Friday, not next Monday. -->

-

---

## Expanding the VM disk (UTM + Ubuntu LVM)

**Situation:** VM disk was 11 GB, increased to 31 GB in UTM settings (VM shut down first).
Ubuntu didn't see the extra space on its own.

**Why:** Ubuntu Server uses LVM by default, so the space has to be passed up through
every layer before the filesystem can use it:

```
disk → partition (vda3) → LVM physical volume → volume group → logical volume → ext4 filesystem
```

Growing the disk in UTM only grows the first layer.

**Steps (all done with the VM running, no reboot):**

```bash
lsblk                     # confirm the disk is bigger
sudo parted /dev/vda resizepart 3 100%    # grow partition 3 to the end of the disk
sudo pvresize /dev/vda3                   # grow the LVM physical volume
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv   # grow the LV + filesystem (-r)
df -h /                   # verify the new size
```

**Gotchas:**

- parted warned "Not all of the space available to /dev/vda appears to be used" → answer
  **Fix**. That rewrites the GPT backup header at the new end of the disk.
- At the `End? [32.2GB]?` prompt, press **Enter** to accept the default. Typing `Yes`
  gives "Error: Invalid number."
- "You may need to update /etc/fstab" is an info message, not an error. fstab uses UUIDs,
  and resizing doesn't change them.
- The Ubuntu installer only gives the logical volume about half the disk by default, so
  `lvextend +100%FREE` can claim more space than was just added.

**Connects to later:** growing an EBS volume in AWS follows the same layered pattern:
grow the volume, then the partition, then the filesystem.

---

## VM couldn't reach GitHub (UTM Shared → Bridged)

**Problem:** `git clone` over SSH on the VM timed out. Port 443 through `ssh.github.com` timed out too. YouTube, Wikipedia, and Yahoo loaded fine. Only GitHub failed, in Firefox as well.

**How I found it:**
- `curl -v --connect-timeout 5 https://github.com` resolved the IP, then timed out on `Trying 140.82.x.x:443...`. It never reached TLS, so nothing was answering the connection request.
- `nc -vz -w 5 140.82.112.4 443` on my Mac connected fine. The Mac could reach GitHub and the VM couldn't.
- `ping 1.1.1.1` showed 100% loss, but that only tests ICMP, so it sent me down the wrong path for a while.

**Fix:** Changed the VM's network mode in UTM from Shared to Bridged. GitHub worked right away. Bridged skips my Mac's NAT. Tailscale's network extension is installed on my Mac and might be the cause, but I didn't confirm it.

**After:** The VM's IP changed. I got the new one with `hostname -I` and updated `HostName` for `devbox` in my Mac's `~/.ssh/config`. To do later: set a DHCP reservation in the router so it stops changing.

**Learned:**
- "Timed out" means nothing answered. It's different from "refused" or "denied."
- `curl -v` shows where a connection stops: resolve, connect, TLS, or HTTP.
- To narrow down a network problem, test from a different spot (VM, then Mac, then skip the Mac).
- `nc -w` needs a number: `nc -vz -w 5 host 443`.
