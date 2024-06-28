# Software stack

Inside Cecile softwares are provided in the whole system through a so called **software stack**, a collection of softwares, which is generated using the [**Spack package manager**](https://spack.io/).

A software stack contains a number of softwares necessary for your analysis and many more. If any software you might need is missing from the stack you need to contact `manuela.kuhn at ovgu.de`.


## Type of stacks

Currently there are **two kinds of stacks** available: 

1. **Current stack:** A stack that is kept stable throughout one semester. Each semester a new stable stack is created from scratch, while the old current stacks are still in place and can be used by specifying the the date in the following way: `current_<date>`.
2. **Experimental stack**: A flexible stack that is going to be regularly updated also by user's request.

!!! Danger "Multiple software versions in Experimental stack"
    Continuous updating in the `experimental stack` can lead to multiple versions of the same software (e.g. `python ~gcc@11.0` vs `python ~gcc@11.1`), therefore be careful to load the correct version when using the experimental stack. Usully back compatibility between software versions is mantained, but  sometimes this might not be the case and some feature might change. 



!!! note "Why two different stacks (use case):"
    If you need to use a software that is not yet installed on Cecile (a.k.a the `current stack`), after your request, the missing software is going to be installed in the `experimental stack`

## How to use the stacks

There are multiple ways to access the stack depending on the how much of control you want over what packages you would like to use:

### Load the Ipsy environment (recommended method)

Instead of loading every package individually, an evironment (containing only the newest version of each package) can be activated.

=== "Current stack"

    ```bash
    $ . /software/current/ipsy-env/activate
    ```

    Now that the stack has been loaded, you can start each single software.

    ```bash
    $ python
    ```

    In order to see which softwares are available in the environment type the following command:

    ```bash
    module avail
    ```

=== "Experimental stack"

    ```bash
    $ . /software/experimental/ipsy-env/activate
    ```
    
    Now that the stack has been loaded, you can start each single software.

    ```bash
    $ python
    ```

    In order to see which softwares are available in the environment type the following command:

    ```bash
    module avail
    ```

### Via modules

For every installed package spack generates a module file in addition, this allows to use the `module` cammand to load specific softwares

```bash
$ . /software/current/spack/share/spack/setup-env.sh
```

To see what packages are installed run
```bash
$ module avail
```

To load a package (in case a specific version is needed, load the version of the software shown by `module avail`)
```bash
$ module load python
```
