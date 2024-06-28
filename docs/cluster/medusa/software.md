# Software stack

The software stacks are shared for the whole cluster and are generated using the [Spack package manager](https://spack.io/)

## Types of available software stacks

Currently there are two types of stacks available.

### Current 

- stable stack throughout the semester
- new stack every semester (build from stratch)
- "old" stacks still valid with `current_<date>`
- 3 ways to access (further details see [Usage]):
  - directly with spack commands
  - via modules
  - via environment
[Usage]: #Usage
Paths:
- stack: `/home/data/software/current`
- environment: `/home/data/software/current/ipsy_env`

### Experimental

- stack that will be updated regularly and on short notice user requests
- updating instead of building from scratch can lead to multiple versions of software (e.g. `python ~gcc@11.0` vs `python ~gcc@11.1`)
- same access methods as in `current` are available, but due to multiple versions not unambiguous\
  -> recommended method: use environment to load stack

Paths:
- stack: `/home/data/software/experimental`
- environment: `/home/data/software/experimental/ipsy_env`

## Usage
*******

There are multiple ways to use the software stacks, depending on how much control you want over what packages are loaded (and in which way):
- [ipsy env]
  [ipsy env]: #load-ipsy-environment
- [directly via spack]
- [via modules]

[directly via spack]: #directly-via-spack
[via modules]: #via-modules
<!--
- [via environment](#via-environment)
- [via custom environment](#via-custom-environment)
-->

### Load ipsy environment

Instead of loading every package individually an evironment can be activated containing only the newest version of each package.\
This is the recommended way for most users.

```
$ . /home/data/software/current/ipsy-env/activate
```

all packages inside of the environment are now loaded and can be used directly.\
e.g.: to start python:
```
$ python
```

To see what software is available run:
```
module avail
```

<!--
### directly via spack

***This currently does not work because the python on medusa ist too old run work with spack directly***

to enable shell support, source
```
$ . /home/data/software/current/spack/share/spack/setup-env.sh
```

to see what packages are installed run
```
$ spack find -x
```

to load a package, e.g python in version 3.8.10 (assuming it was installed and
can be seen in the `spack find` list)
```
$ spack load python@3.8.12
```
(if only one version is installed, the @3.8.12 is not needed)

now you can use this python version by simply calling
```
$ python
```

if you do not want this version anymore you can unload it again
```
$ spack unload python@3.8.12
```

Remark: the main python and r packages are kept as seperate packages and thus
have to be loaded separately. e.g. `spack load r-rstan`
-->


### Via modules

For every installed package spack generates a module file in addition meaning
that classic module commands can be used as well.

Again shell support has to be enabled
```
$ . /home/data/software/current/env.sh
```

to see what packages are installed run
```
$ module avail
```

to load a package
```
$ module load python
```
or if a specific version is needed, load the entry shown in `module avail`

<!--
### via environment

**This currently does not work** because of old package versions in the stack `view` had to be disabled to prevent the whole stack to be pulled down by them.

a different way of loading the environment is using the spack environment directly (the above method uses modules)

```
$ . /home/data/software/current/spack/share/spack/setup-env.sh
$ spacktivate /home/data/software/ipsy_env
```
who likes to see the activate enviroment in the command line can use the `-p` option

all packages inside of the environment are now loaded and can be used:
```
$ python
```

To leave the environment use
```
despacktivate
```

### via custom environments

Instead of using a predefined enviroment, one can also specify a custom environment for, e.g., a specific project.

The initial setup of the environment consists of four steps:

1. Creating & activating the custom enviroment

```
$ spack env create -d <my-env-dir>
$ spacktivate -p <my-env-dir>
```

2. Adding all packages that one wants to use in the enviroment

For example, all pre-installed R-packages

```
$ spack add r
$ spack add r-rstan
```

One can verify the list of added packages with
```
$ spack find # returns the following:
==> In environment <my-env-dir>
==> Root specs
r r-rstan
```

3. Concretize the environment.

This step makes all the `add`ed packages actually available within the environment

```
$ spack concretize
```

Note: This command can the a while to run.

4. Create load-script for loading all packages

We will use a script which spack generates for us:

```
$ spack env loads -r
```

Whenever you want to activate the environment and the installed packages, simply run the suggested command:

```
$ . <my-env-dir>/loads
```

for more infos on spack environments, see [the docs](https://spack.readthedocs.io/en/latest/environments.html) or the [tutorial](https://spack-tutorial.readthedocs.io/en/latest/tutorial_environments.html)
-->