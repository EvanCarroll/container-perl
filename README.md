`container_perl`
====

This is a wrapper around perl to recreate the environment in `podman`. What it
does is,

* Get the version of perl on the host, map it to an OCI image name.
* Get the current `@INC` on the host.
* Prepare a container that maps the host's `@INC` directories to the container
* Clobbers the `@INC` of the container perl with just the mapped dirs from the
	host using [`libreplace`](https://metacpan.org/pod/libreplace).
* `exec`'s a copy of a Perl using Podman in a container as above.

Security
---

The goal of this program is to allow arbitrary code executation of perl within
the context of a secure user-namespace.

Example
====

```sh
container-perl -E 'say "Hello World"';
container-perl ./test.pl
```

For the purpose of the demo, `test.pl` outputs the UID. This will change when
run inside and outside of `container-perl` because user namespaces allow 
perl running in the namespace to think it's root. While invoking this file with
regular perl will show it as the UID of the user.

Installation
====

Container perl requires,

* usernamespaces enabled in the kernel
* `podman` installed
* [`libreplace`](https://github.com/EvanCarroll/perl5-libreplace) installed

Notes
====

Currently, all the deps directory outside the directory are mounted read-only.
The working directory is mounted read-write.

Inspiration
----

The source of inspiration of this was [Brian Scannell's talk in The Perl
Conference 2022 on IDE and checking Perl
syntax](https://tprc2022.sched.com/event/11nfS/the-perl-navigator-code-intelligence-for-any-editor).
In that talk Brian puts forward two methods of dealing with the insecurity of
checking perl syntax, with `perl -c`

1. Trusting a project. Trust in this context would require you to audit the
	 code before testing the syntax. I doubt anyone will do this.
2. Executing the syntax checking on a remote machine, over SSH.

This approach uses podman to create a ephemeral container to syntax check Perl.
