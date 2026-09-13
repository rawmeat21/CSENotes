### List tags

```console
$ git tag
v1.0
v2.0
```

### Create tags

2 types of tags: _lightweight_ and _annotated_.

A lightweight tag is very much like a branch that doesn’t change, it’s just a pointer to a specific commit.

Annotated tags, however, are stored as full objects in the Git database. They’re checksummed; contain the tagger name, email, and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (GPG). 

It’s generally recommended that you create annotated tags so you can have all this information.

#### Annotated Tags

Use `-a`:
```console
$ git tag -a v1.4 -m "my version 1.4"
```

The `-m` specifies a tagging message, which is stored with the tag.

To see tag data along with the commit that was tagged, use `git show` command:

```console
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number
```

#### Lightweight Tags

This is basically the commit checksum stored in a file, no other information is kept.

```console
$ git tag v1.4-lw
$ git tag
v0.1
v1.3
v1.4
v1.4-lw
v1.5
```

```console
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    Change version number
```


### Tagging Later

```console
$ git log --pretty=oneline
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
a6b4c97498bd301d84096da251c98a07c7723e65 Create write support
0d52aaab4479697da7686c15f77a3d64d9165190 One more thing
6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc Add commit function
4682c3261057305bdd616e23b64b0857d832627b Add todo file
166ae0c4d3f420721acbb115cc33848dfcc2121a Create write support
9fceb02d0ae598e95dc970b74767f19372d61af8 Update rakefile
964f16d36dfccde844893cac5b347e7b3d44abbc Commit the todo
8a5cbc430f1a9c3d00faaeffd07798508422908a Update readme
```

Now, suppose you forgot to tag the project at v1.2, which was at the “Update rakefile” commit. To tag that commit, you specify the commit checksum (or part of it) at the end of the command:

```console
$ git tag -a v1.2 9fceb02
```

### How to share tags

**By default, the `git push` command doesn’t transfer tags to remote servers.

You will have to explicitly push tags to a shared server after you have created them. This process is just like sharing remote branches: you can run `git push origin <tagname>`.

```console
$ git push origin v1.5
Counting objects: 14, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (12/12), done.
Writing objects: 100% (14/14), 2.05 KiB | 0 bytes/s, done.
Total 14 (delta 3), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.5 -> v1.5
```

If you have a lot of tags that you want to push up at once, you can also use the `--tags` option to the `git push` command. This will transfer all of your tags to the remote server that are not already there.

```console
$ git push origin --tags
Counting objects: 1, done.
Writing objects: 100% (1/1), 160 bytes | 0 bytes/s, done.
Total 1 (delta 0), reused 0 (delta 0)
To git@github.com:schacon/simplegit.git
 * [new tag]         v1.4 -> v1.4
 * [new tag]         v1.4-lw -> v1.4-lw
```


### How to delete tags

Use `git tag -d <tagname>`

```console
$ git tag -d v1.4-lw
Deleted tag 'v1.4-lw' (was e7d5add)
```

**Note that this does not remove the tag from any remote servers.**

#### How to delete remote tags?

Way 1:

`git push <remote> :refs/tags/<tagname>`

```console
$ git push origin :refs/tags/v1.4-lw
To /git@github.com:schacon/simplegit.git
 - [deleted]         v1.4-lw
```

Read as: null value before the : is being pushed to the remote tag name, deleting it.

Way 2:

```console
git push origin --delete <tagname>
```


### Checking out tags

If you want to view the versions of files a tag is pointing to, you can do a `git checkout` of that tag, although this puts your repository in “detached HEAD” state, which has some ill side effects:


It's better to refer this now: 
https://git-scm.com/book/en/v2/Git-Basics-Tagging
