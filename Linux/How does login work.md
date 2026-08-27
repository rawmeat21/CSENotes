## The `login` Program

This passage ties directly into the UID discussion from before — it's showing you a concrete, everyday example of privilege dropping in action, and an important limitation on it.

### What's happening step by step

**1. You sit down and try to log in.** The `login` program (or your GUI's equivalent — the login screen you see on a desktop) is what greets you and asks for your username and password.

**2. This `login` process itself runs as root.** Before you've even typed anything, the process handling your login prompt already has full root privileges — UID 0, GID 0.

Why does it need to be root at this stage? Because it has to do things an ordinary user can't:

- Read `/etc/shadow` (the password hash file — root-only, as we discussed)
- Check your typed password against the stored hash
- Potentially set up your terminal/session
- Prepare to switch identity to _any_ user on the system (it doesn't know in advance if you're `qing` or `alice` or anyone else)

**3. If your credentials check out**, `login` does something significant: it **changes its own UID and GID** from root to _your_ UID and GID (say, 1000/1000 for `rawmeat`).

**4. Once that switch happens, it starts your shell or desktop environment** — and that shell now runs as you, not as root.

Only a process that's **already root** has the power to change its UID/GID to arbitrary values. This is exactly the "superuser powers" the heading is about.

So `login` is a privileged bootstrapping tool: it starts as root specifically _so that_ it can hand off control to the correct unprivileged user afterward.

### The important one-way street

Here's the critical detail in the last sentence: **once `login` has changed itself from root down to your normal user, it cannot go back.**

This is different from the saved-UID "parking spot". In that case, a process could stash root privileges in the saved UID and reclaim them later. But `login`'s privilege drop here is **permanent and irreversible** for that process — there's no saved UID being kept around for it to climb back up with.

Why does this matter for security? Because it means once your shell is running as you, there's no backdoor left in that process for it to silently re-become root. If a process _could_ freely climb back to root after dropping privileges, that would defeat the whole purpose of dropping them in the first place — malicious code could exploit that path to escalate back to full power.