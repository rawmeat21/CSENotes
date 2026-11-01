A bare repository is a repository that doesn't have a working directory. It only contains the `.git` directory, the directory in which Git stores all its internal data. The main purpose of these repositories is to be a central repository that developers can push to and pull from, so there's no need in having a working directory. Bare repositories are also used in Git hosting services like GitHub and GitLab.

It basically means, we cannot have project files there like regular repositories but we can choose a different location for storing files.

A regular git repository consists of both a working tree as well as the “.git” directory which contains all the information that necessary for the git repository including the history of the repository.

### Create a bare repository

```
git init --bare
```

This will initialize the directory as a git bare repository. Then you will see a strange list of directories and files. These are the repository data. If the repo was a regular one, these things are included inside the “.git” subdirectory.

https://thilinatlm.medium.com/quick-web-deployments-using-git-bare-repositories-2e6e1ceaa06e
https://www.youtube.com/watch?v=8aZW9mYOxhc
