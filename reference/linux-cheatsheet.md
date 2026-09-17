# My Linux Cheatsheet

Rule: **only add a command here after I've used it and understood it.**
A copied cheatsheet is worthless. One I built myself is a memory aid.

---

## Commands I keep having to look up

| Command | What it does | When I needed it |
|---|---|---|
| sshd -T | Dumps the effective configuration after all includes are resolved. It's the difference between what you wrote and what the daemon actually believes. |Lab01 for disabling passwor authentication|
| stat -f '%A %N' ~/.ssh ~/.ssh/* |  Checking the file permissions of SSH keys and files. MacOS only  |  End of Lab01  |
| stat -c '%a %A %n' ~/.ssh ~/.ssh/* |  Linux version of file permissions. Differs slightly between Mac and Linux  |  End of Lab01  |
|   |    |    |
|   |    |    |
|   |    |    |
|   |    |    |
|   |    |    |

## Things that bit me

| Symptom | Actual cause | Fix |
|---|---|---|
| | | |

## Mental models worth keeping

<!-- e.g. "NAT vs bridged is the same distinction as Docker's default bridge network
     vs host networking" — connections between phases go here. -->

- First matches win when it comes to config files. If there is a includes at the top of a file with the property you want to change, change that one first, the ones that come after it don't matter. 
