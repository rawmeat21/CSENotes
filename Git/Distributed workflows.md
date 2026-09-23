https://git-scm.com/book/en/v2/Distributed-Git-Distributed-Workflows

In Git, every developer is potentially both a node and a hub; that is, every developer can both contribute code to other repositories and maintain a public repository on which others can base their work and which they can contribute to.

### Centralized Workflow

In centralized systems, there is generally a single collaboration model — the centralized workflow. One central hub, or _repository_, can accept code, and everyone synchronizes their work with it. A number of developers are nodes — consumers of that hub — and synchronize with that centralized location.

![Pasted image 20260917235516](../assets/Pasted%20image%2020260917235516.png)


### Integration-Manager Workflow (what we do in fork)

-> It’s possible to have a workflow where each developer has write access to their own public repository and read access to everyone else’s. 

-> This scenario often includes a canonical repository that represents the “official” project. 

-> To contribute to that project, you create your own public clone of the project and push your changes to it. Then, you can send a request to the maintainer of the main project to pull in your changes. 

-> The maintainer can then add your repository as a remote, test your changes locally, merge them into their branch, and push back to their repository.

- The project maintainer pushes to their public repository.
    
- A contributor clones that repository and makes changes.
    
- The contributor pushes to their own public copy.
    
- The contributor sends the maintainer an email asking them to pull changes.
    
- The maintainer adds the contributor’s repository as a remote and merges locally.
    
- The maintainer pushes merged changes to the main repository.


![Pasted image 20260917235823](../assets/Pasted%20image%2020260917235823.png)


This is a very common workflow with hub-based tools like GitHub or GitLab, where it’s easy to fork a project and push your changes into your fork for everyone to see.


### Dictator and Lieutenants Workflow

It’s generally used by huge projects with hundreds of collaborators; one famous example is the Linux kernel.

-> Various integration managers are in charge of certain parts of the repository; they’re called _lieutenants_. 

-> All the lieutenants have one integration manager known as the benevolent dictator. The benevolent dictator pushes from their directory to a reference repository from which all the collaborators need to pull. 

The process works like this:

- **Regular developers** work on their topic branch and rebase their work on top of `master`. The `master` branch is that of the reference repository to which the dictator pushes.
    
- **Lieutenants** merge the developers' topic branches into their `master` branch.
    
- The **dictator** merges the lieutenants' `master` branches into the dictator’s `master` branch.
    
- Finally, the dictator pushes that `master` branch to the reference repository so the other developers can rebase on it.

![Pasted image 20260918000053](../assets/Pasted%20image%2020260918000053.png)


**The flow**

1. A developer works on a topic branch, rebasing it onto the current reference `master` to keep it up to date.
2. The relevant lieutenant (the one responsible for that subsystem) merges the developer's topic branch into their own `master`.
3. The dictator merges all the lieutenants' `master` branches into their own `master`.
4. The dictator pushes that combined `master` to the reference repository, and everyone else rebases their future work on top of it.


#### How a simple workflow looks like (private small team)

![Pasted image 20260918100515](../assets/Pasted%20image%2020260918100515.png)

Read the section "Private Small team" from here: https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project


#### How a more complex workflow looks like

See section "Private Managed team": https://git-scm.com/book/en/v2/Distributed-Git-Contributing-to-a-Project

How 