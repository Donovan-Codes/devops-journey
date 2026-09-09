# Lab 1 — Key-Based SSH & Disabling Password Auth

**Phase:** 1 — Linux fundamentals
**Time:** ~45 minutes
**Prerequisite:** VM booted, you can SSH in with a password

---

## Objective

Log into `devbox` from your Mac without typing a password, then turn password
authentication off entirely so keys are the only way in.

## Why this matters

This is the most common single piece of server hardening there is. Passwords can be
brute-forced; a 256-bit key can't. Every cloud provider defaults to key-only auth on
new instances, every CI system authenticates to git with a key, and every production
server you ever touch will be configured this way.

You'll also meet the exact same mechanic again in Phase 6 (EC2 key pairs), Phase 8
(cluster access), and every time you push to GitHub.

---

## ⚠️ Before you start: the safety net

You are about to change the config of the service you're using to connect. If you get
it wrong, you lock yourself out.

Two protections, use both:

1. **Keep your current SSH session open the entire time.** Do all editing in it. Test
   the new configuration from a *second, separate* terminal window. If the new window
   fails, the old session is still alive and can undo the change.
2. **The UTM console window is your out-of-band access.** Even if SSH is completely
   broken, you can open the VM console in UTM and log in directly. That's the VM
   equivalent of walking to the datacenter — it's why real servers have console access.

If you do lock yourself out, don't panic and don't delete the VM. Use the console.
And write it up in your log, because "I locked myself out of SSH and recovered via
console" is a genuinely good story.

---

## Tasks

Figure out the exact commands yourself — the point is the reading, not the typing.
Command names are given; flags and arguments are yours to find with `man`.

### 1. Generate a keypair on your Mac

Command: `ssh-keygen`

- Use the `ed25519` algorithm, not RSA (it's shorter, faster, and modern default)
- Add a comment identifying the key — usually your email
- Decide whether to set a passphrase. Either is defensible; know the tradeoff.

**Understand before continuing:** two files were created. One is secret and never
leaves your machine. One is public and gets copied to servers. Know which is which
and what each is named. Getting this backwards is a classic beginner mistake with
real consequences.

### 2. Copy the public key to the VM

Command: `ssh-copy-id`

This is the last time you'll type your VM password.

**Then go look at what it actually did.** On the VM, find the file it wrote to. Check
the permissions on that file and on the directory containing it with `ls -la`. Note
what they are — you'll need this in step 5.

### 3. Verify passwordless login works

From a **new** terminal window, SSH in. You should land at a prompt without a password
challenge (or be asked for your *key passphrase*, if you set one — that's different,
and understanding why is the point).

**Do not proceed until this works.** If it's still asking for your account password,
something in step 2 didn't take. Fix it now, while password auth is still your
fallback.

### 4. Disable password authentication

File: `/etc/ssh/sshd_config`
Editor: `sudo nano /etc/ssh/sshd_config`

Find and change these directives:

- `PasswordAuthentication` → `no`
- `PermitRootLogin` → `no`

Notes: lines starting with `#` are comments and have no effect — a commented-out
setting is not a setting. Some values may appear more than once; the file is read
top-down. Look for an `Include` line near the top too, and understand what it means
for where your changes should go.

Then reload sshd. Use `systemctl` — you'll need to know whether the service is called
`ssh` or `sshd` on Ubuntu. Check with `systemctl status` before guessing.

### 5. Verify the lockdown

From a third terminal, try to force a password login:

```
ssh -o PubkeyAuthentication=no youruser@<vm-ip>
```

This should be **refused**. If it prompts you for a password, the change didn't apply
— re-read step 4.

### 6. Make it convenient

Create `~/.ssh/config` on your Mac so you can type `ssh devbox` instead of the full
user@ip. Look up the `Host`, `HostName`, `User`, and `IdentityFile` directives.

Check the permissions on that file when you're done — see the next section.

---

## Verification checklist

- [ ] `ssh devbox` logs you in with no account password
- [ ] Forcing password auth is refused by the server
- [ ] Root cannot log in over SSH
- [ ] You can explain which of your two key files is safe to share and why
- [ ] You can state the correct permissions for `~/.ssh` and `authorized_keys`

---

## Troubleshooting — symptoms only

Diagnose these yourself. That's the skill.

**"Permission denied (publickey)" after disabling passwords**
Your key isn't being accepted and there's no fallback. Check the server's view of
what happened: `sudo journalctl -u ssh -n 50`. The log will usually tell you outright.

**Key auth silently doesn't work, falls back to password prompt**
Almost always permissions. sshd refuses to use a key if the directory or file is
readable by anyone other than the owner — it will not tell you this at the client end.
`~/.ssh` should be `700`, `authorized_keys` should be `600`. This is Break-and-Fix
drill #3, and you may have just done it accidentally.

**Connection refused / connection timed out**
Different failure entirely. Refused means nothing is listening; timed out means
nothing answered. Check the service is running and the IP is still what you think it
is — DHCP may have moved it.

**Changes to sshd_config appear to do nothing**
Either the service wasn't reloaded, the directive is duplicated further down the file,
or your edit is in a file that's being overridden by an `Include`.

---

## For your log

Write these up while it's fresh:

- What did `ssh-copy-id` actually do? (Not "copied my key" — what file, what format?)
- What's the difference between a key passphrase and an account password?
- Did anything fail? What was the *symptom*, and what command revealed the cause?
- Why does sshd refuse to use a key with loose file permissions?

---

## Going further (optional)

- Set up the same key auth against a second machine and use `ssh-agent` so you only
  unlock your passphrase once per session
- Look at `~/.ssh/known_hosts` — what is it, and what is it protecting you from?
- Change the SSH port to something non-standard, and reason about whether that's
  actually security or just noise reduction
