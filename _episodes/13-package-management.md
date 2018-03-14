---
title: "Managing software"
teaching: 30
exercises: 15
questions:
- "How do install and use software packages?"
objectives:
- "Understand how to install and use a software package."
keypoints:
- "GNU Guix is a user-space package manager aimed at reproducibility"
- "Install software with `guixr package -i software-name`"
- "Loading software environments with `guixr load-profile`"
- "You can edit your `.bashrc` file to automatically load software profiles."
---

The main interaction we will have with the high-performance computing system
(HPC), is running programs.  If we want to run a program, we must install it
first.

## Package managers

The HPC uses CentOS as its operating system.  This is a GNU/Linux distribution.
The main way to install programs is to use the distribution's package manager.
However, that is only available to super users.

Instead of resorting to compiling programs ourselves, the HPC provides another
package manager that can be used to install programs.  This package manager is
called GNU Guix.  In the remainder of this episode, we will look at how it can
be used to create and maintain software environments, and what others need to
reproduce our software environments.

## Enable GNU Guix

To use GNU Guix, add the following line to $HOME/.bashrc file:
```
export PATH=$PATH:"/gnu/profiles/base/bin"
```

## Finding programs

We can find packages by using the
[web interface's search function](https://hpcguix.op.umcutrecht.nl/),
or by using the command line:
```
$ guixr package --list-available | less
$ guixr package --search=samtools
$ guixr package -s ^samtools$
```

This output can be a bit verbose, so an alternative is to use the
`--list-available` option, or its short-hand `-A`:
```
$ guixr package -A ^samtools$
```

## Installing programs

To install packages, for example bwa, we can use the following command:
```
$ guixr package --install=bwa --profile=$PROJECT_ROOT/.guix-profile
```

Or its equivalent using “shorthand” notation:
```
$ guixr package -i bwa -p $PROJECT_ROOT/.guix-profile
```

To get an overview of the packages in the project’s profile, we can use:
```
$ guixr package --list-installed -p $PROJECT_ROOT/.guix-profile
```

## Running installed programs

Once the desired programs have been installed in a profile, we need to tell the
system to use the programs in that profile. We do this by “loading a profile”,
using:

```
$ guixr load-profile $PROJECT_ROOT/.guix-profile
```

This command will provide a new shell prompt from which those (and only those)
programs in the profile are available to us.

Note: You may notice that even basic commands like `ls` and `grep` may not be
available inside an environment created with the load-profile command. This
environment ensures that we are running exactly the software we expect to be
running (that inside the profile we loaded). If you need these programs,
install the `coreutils` and `grep` package into the profile.

A common task is to run one or more commands on a compute node using a specially
crafted job script. This is how it can be done in combination with the
`load-profile` command:

```
guixr load-profile $PROJECT_ROOT/.guix-profile --<<EOF
  Rscript $PROJECT_ROOT/scripts/analyse.R
EOF
```

So, any command available inside the profile environment can be placed between
the `EOF` markers, on separate lines.

## Software updates

To make it easy for someone else to reproduce the programs, and therefore, the
essential tools to reproduce the entire data analysis, we would like to have a
single set of instructions to perform, like:

> All software needed to perform this data analysis can be installed by running
> `guixr package -i bwa samtools delly gatk r r-mutationalpatterns r-ggplot2`,
> after installing GNU Guix.

This instruction implies all software was installed in a single point in time,
or more accurately, using a single snapshot of the GNU Guix package recipes
state.

So, when upgrading a single program, we should upgrade all programs in the
profile, so that the entire profile can be installed using a single installation
instruction.
