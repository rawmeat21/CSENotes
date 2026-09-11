### PAM: Pluggable Authentication Modules

-> User accounts are traditionally secured by passwords stored (in encrypted form) in the /etc/shadow or /etc/master.passwd file or an equivalent network database. Many programs may need to validate accounts, including login, sudo, su, and any program that accepts logins on a GUI workstation.

-> These programs really shouldn’t have hard-coded expectations about how passwords are to be encrypted or verified. Ideally, they shouldn’t even assume that passwords are in use at all. What if you want to use biometric identification, a network identity system, or some kind of two-factor authentication?

-> PAM is a wrapper for a variety of method-specific authentication libraries. Administrators specify the authentication methods they want the system to use, along with the appropriate contexts for each one. Programs that require user authentication simply call the PAM system rather than implement their own forms of authentication. PAM in turn calls the authentication library specified by the system administrator.


### Kerberos: network cryptographic authentication

Like PAM, Kerberos deals with authentication rather than access control. 

But whereas PAM is an authentication framework, Kerberos is a specific authentication
method. 

At sites that use Kerberos, PAM and Kerberos generally work together, PAM being the wrapper and Kerberos the actual implementation.

Kerberos uses a trusted third party (a server) to perform authentication for an entire network. You don’t authenticate yourself to the machine you are using, but provide your credentials to the Kerberos service. Kerberos then issues cryptographic credentials that you can present to other services as evidence of your identity.


### Filesystem access control lists (ACL)

Access control lists (ACLs) are a generalization of the traditional user/group/other permission model that permits permissions to be set for multiple users and groups at once.

**ACLs are part of the filesystem implementation, so they have to be explicitly supported by whatever filesystem you are using**. However, all major UNIX and Linux filesystems now support ACLs in one form or another.

### Linux capabilities

**Capability systems divide the powers of the root account into a handful (~30) of separate permissions.**

-> Capabilities can be inherited from a parent process. They can also be enabled or disabled by attributes set on an executable file, in a process reminiscent of setuid execution. Processes can renounce capabilities that they don’t plan to use.

-> The traditional powers of root are simply the union of all possible capabilities, so there’s a fairly direct mapping between the traditional model and the capability model.

-> The Linux capability called `CAP_NET_BIND_SERVICE` controls a process’s ability to bind to privileged network ports (those numbered under 1,024). Some daemons that traditionally run as root need only this one particular superpower. In the capability world, such a daemon can theoretically run as an unprivileged user and pick up the port-binding capability from its executable file.

As long as the daemon does not explicitly check to be sure that it’s running as root,
it needn’t even be capability aware.


### Linux namespaces

Linux can segregate processes into hierarchical partitions (“namespaces”) from which they see only a subset of the system’s files, network ports, and processes.

Instead of having to base access control decisions on potentially subtle criteria, the kernel
simply denies the existence of objects that are **not visible from inside a given box**.

Inside a partition, normal access control rules apply, and in most cases jailed processes are not even aware that they have been confined. 

Because confinement is irreversible, processes can run as root within a partition without fear that they might endanger other parts of the system.


