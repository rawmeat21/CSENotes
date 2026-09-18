
First, you need to select which protocols you want to use.

What is a **bare repository**? It's just the `.git` contents, no working directory, since nobody needs to check files out on it.

## The Protocols

Git can use four distinct protocols to transfer data: Local, HTTP, Secure Shell (SSH) and Git.

### Local Protocol

The remote repository is in another directory on the **same host**.

This is often used if everyone on your team has access to a shared filesystem such as an [NFS](https://en.wikipedia.org/wiki/Network_File_System) mount, or in the less likely case that everyone logs in to the same computer.

To clone such a repo:

```
$ git clone /path/to/project.git
```

Note: If you use:

```
$ git clone file:///path/to/project.git
```

Git fires up the processes that it normally uses to transfer data over a network, which is generally much less efficient. The main reason to specify the `file://` prefix is if you want a clean copy of the repository with extraneous references or objects left out, generally after an import from another VCS or something similar (see [Git Internals](https://git-scm.com/book/en/v2/ch00/ch10-git-internals) for maintenance tasks).


### The HTTP Protocols

### Smart HTTP

Runs over standard HTTPS ports and can use various HTTP authentication mechanisms. You can use things like username/password authentication rather than having to set up SSH keys.

### Dumb HTTP

If the server does not respond with a Git HTTP smart service, the Git client will try to fall back to the simpler _Dumb_ HTTP protocol. The Dumb protocol expects the bare Git repository to be served like normal files from the web server.

You literally drop a bare repo under your web server's document root, enable the `post-update` hook (runs `git update-server-info`), and the repo is served as static files.

How to setup:

```console
$ cd /var/www/htdocs/
$ git clone --bare /path/to/git_project gitproject.git
$ cd gitproject.git
$ mv hooks/post-update.sample hooks/post-update
$ chmod a+x hooks/post-update
```

What each step does:

1. `cd` into your web server's document root (the directory it serves files from).
2. `git clone --bare` makes a bare copy of your project (no working directory, just the `.git` contents) directly inside that document root, named `gitproject.git`.
3. Enable the `post-update` hook by renaming the sample file that ships with Git. This hook runs `git update-server-info` automatically whenever someone pushes to the repo (over SSH, say), which regenerates the auxiliary info files that dumb HTTP clients need to fetch/clone.
4. Make the hook executable with `chmod a+x`.

Now anyone with access to the web server can now clone read-only over plain HTTP:

```console
$ git clone https://example.com/gitproject.git
```

A couple of things worth noting:

- This only gives **read-only** access, there's no push support over dumb HTTP.
- The repo is served as plain static files by whatever web server you're already running (Apache, nginx, etc.), there's no special Git server process involved, which is why it's so easy to set up.
- If you ever push new commits into that bare repo via another route (e.g., SSH), the `post-update` hook fires and keeps the static info files current so dumb HTTP clients can see the new data.


### The SSH Protocol

### Cloning over SSH

```console
$ git clone ssh://[user@]server/project.git
```

or the shorter scp-like form:

console

```console
$ git clone [user@]server:project.git
```

If you omit the username, Git just uses whatever user you're currently logged in as on your local machine.

Example:

`git@github.com:rawmeat21/fragarach.git`:

- `git` → the user (GitHub uses a shared `git` account for all SSH access, and identifies _you_ individually via which SSH public key you've registered)
- `github.com` → the server
- `rawmeat21/fragarach.git` → the "project" path (here, `owner/repo.git`)

It's the exact same thing as writing:

```
ssh://git@github.com/rawmeat21/fragarach.git
```

