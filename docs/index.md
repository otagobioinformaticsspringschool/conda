# Conda/Mamba
!!! overview "On this Page"
      - About Conda and alternatives
      - How to install Conda
      - How to do additional configurations
      - What are Conda Environments
 
  <!-- TODO See if overview is in line with content -->


## What is Conda

Conda is a cross‑platform package, dependency and environment manager that makes it easy to install binary packages (both Python and non‑Python) into isolated environments. By encapsulating exact package versions and their dependencies in a named or prefix environment, and by exporting an environment specification (for example with conda env export > environment.yml), you can recreate the same software stack on another machine or share it with collaborators. To enhance the reproducibility of your research, commit the environment.yml to version control, and recreate environments with conda env create -f environment.yml to ensure analyses remain reproducible over time.

!!!note "Mamba"
    Mamba is a drop in replacement for conda which has a faster package and dependency resolver.

The `Miniforge3` environment module provides the
[Conda](https://docs.conda.io/projects/conda/en/latest/) package and
environment manager. Conda lets you install packages and their
dependencies in dedicated environment, giving you more freedom to
install software yourself at the expense of possibly less optimized
packages and no curation by the REANNZ team.

Documentation about using miniforge on REANNZ is found at [https://docs.nesi.org.nz/Scientific_Computing/Supported_Applications/Miniforge3/](https://docs.nesi.org.nz/Scientific_Computing/Supported_Applications/Miniforge3/)

## Using miniforge and conda on the REANNZ HPC

Using miniforge on the HPC differs slightly from using it on your local computer in that miniforge itself is loaded using the `module` commands, and has some extra configurations to change some default behaviours due to the way the REANNZ HPC is set up - namely to avoid many default locations being your home directory, and instead to use project storage locations.

### Installing conda on your PC

The miniforge installer nad instructions are found at [https://conda-forge.org/download/](https://conda-forge.org/download/)

!!! note

    Some bioinformatic software requires an UNIX shell to run - which involves additional steps such as installing WSL2 in Windows.


### Module loading and conda environments isolation

When using the Miniforge3 module, we recommend using the following
snippet to ensure that your conda environments can be activated and are
isolated as possible from the rest of the system:

``` sh
module purge && module load Miniforge3
source $(conda info --base)/etc/profile.d/conda.sh
export PYTHONNOUSERSITE=1
```

Here are the explanations for each line of this snippet:

- `module purge && module load Miniforge3` ensures that no other
    environment module can affect your conda environments. In
    particular, the Python environment module change the `PYTHONPATH`
    variable, breaking the isolation of the conda environments. If you
    need other environment modules, make sure to load them after this
    line.
- `source $(conda info --base)/etc/profile.d/conda.sh` ensures that
    you can use the `conda activate` command.
- `export PYTHONNOUSERSITE=1` makes sure that local packages installed
    in your home folder `~/.local/lib/pythonX.Y/site-packages/` (where
    `X.Y` is the Python version, e.g. 3.8) by `pip install --user` are
    excluded from your conda environments.

!!! warning
     We **strongly** recommend against using `conda init`. It inserts a
     snippet in your `~/.bashrc` file that will freeze the version of conda
     used, bypassing the environment module system.

!!! warning "Defaults Channel"
     If you are using a `environment.yml` file, you will have to remove the
     `defaults` channel, or you will receive an error.
     
     ``` out
     Failed to create Conda environment
     The channel is not accessible or is invalid.
     ``` 
     
     The `defaults` channel is blocked due to Anaconda's licensing requirements.





### Configurations

#### Prevent conda from using /home storage

Conda environments and the conda packages cache can take a lot of
storage space. By default, Conda uses your home directory (`~/`),
which is restricted to 20GB on the REANNZ HPC. Here are some techniques to avoid
running out of space when using Conda.

First, we recommend that you move the cache folder used for downloaded
packages on the `nobackup` folder of your project:

``` sh
conda config --add pkgs_dirs /nesi/nobackup/<project_code>/$USER/conda_pkgs
```

where `<project_code>` should be replace with your project code. This
setting is saved in your `~/.condarc` configuration file.
!!! prerequisite Note
     Your package cache will be subject to the nobackup autodelete process
     (details available in the [Nobackup autodelete](https://docs.nesi.org.nz/Storage/File_Systems_and_Quotas/Automatic_cleaning_of_nobackup_file_system/)
     support page). The package cache folder is for temporary storage so it
     is safe if files within the cache folder are removed.


!!! tip "Reduce prompt prefix"
     By default, when activating a conda environment created with `-p` or
     `--prefix`, the entire path of the environment is be added to the
     prompt. To remove this long prefix in your shell prompt, use the
     following configuration:
     ``` sh
     conda config --set env_prompt '({name})'
     ```

--------

Loading miniforge

- REANNZ docs
- setup config
- create environment
  - activate environment
  - install into environment
  - save requirements
- load environment
- 









#### Self installing

If you would prefer to install and manage your own version of conda you can do so, we recommend using _[Miniforge](https://conda-forge.org/miniforge)_ to manage conda environments and packages.
Miniforge is a community-led, minimal conda/mamba installer that uses _[conda-forge](https://conda-forge.org/)_ as the default channel.


To install Miniforge under your user account, you can use the following commands:

!!! terminal
    ```bash
    wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
    bash Miniforge3-Linux-x86_64.sh -b -u
    ```

Once installed (the default location is `/home/<user>/miniforge3`), load conda using:

!!! terminal

    ```bash
    source ~/miniforge3/etc/profile.d/conda.sh
    ```

!!! note

    Installing miniforge and loading as above will also give you access to `mamba` which is a drop in faster replacement to `conda`.

### Extra Configuration


#### Bioconda

Bioconda ([https://bioconda.github.io](https://bioconda.github.io)) is a popular repository for bioinformatic software. To be able to make use of the bioconda repository you must configure conda to know about it. The following commands are from [https://bioconda.github.io/#usage](https://bioconda.github.io/#usage) and will configure conda to search and download from the bioconda repositiory when installing into enivronments.

!!! terminal
    ```bash
    conda config --add channels bioconda
    conda config --add channels conda-forge
    conda config --set channel_priority strict
    ```

These commands modify your `~/.condarc` file.

## Conda environments

Conda environments let you manage software (and it's dependencies). Ultimately, conda install software into specific directories and then alters your `PATH` for you to make them accessible. To utilise an environment it must first exist, and then be activated. 

There are two types of environments: _named_ and _prefix_. 

- _Named_ environments are installed within your home directory (subdirectories within `~/.conda/envs`) and will let you activiate by `conda activate <environment_name>`.

- _Prefix_ environments are installed into the location you specify at creation. If this location is accessible by others, they too can use the environment. For reproducibility it is useful to create an environment in a project directory and use that for operating on data there. When you change to a different project, you can activate the corresponding environment. This lets you manage your software at the project level.

!!!note
    As named environments are stored in your home directory these can take up a large portion of your home directory storage quota. They are also not accessible to others.

    The use of _prefix_ environments is encouraged as you can specify the location to store these (ideally in your project directory)


!!! warning
    We strongly recommend against using `conda init`. It inserts a snippet in your `~/.bashrc` file that will freeze the version of conda used, bypassing the environment module system.


## Creating and activating an environment


!!! information "Managing Conda Environments to Conserve Home Directory Storage"

    To save home directory storage space, it is recommended to create Conda environments in a shared project directory. 
    This approach allows you to manage your Conda environments within your project directory and if needed share them with collaborators.


Lets create an environment to practice with


!!! terminal "Creating Conda Environments"

    Using the prefix method (REANNZ preferred method)
    
    ```bash
    # ensure we have a driectory
    mkdir -p ~/obss_2025/conda_envs/

    conda create -p ~/obss_2025/conda_envs/bioTools
    ```


It will show the proposed installation location, and once you answer the prompt to proceed, will do the installation.  If you have followed these instruction, this location should be `/home/users/<your_username>/obss/conda_envs/bioTools`. 


Once you have created your environment, you can activate it using `conda activate <path>` for example:

!!! terminal

        ```bash
        conda activate ~/obss_2025/conda_envs/bioTools
        ```


The command prompt will then change (e.g. to start with "(myenv) ") to reflect this.  Typing conda deactivate once will return you to the base environment; typing it a second time will deactivate conda completely (as above).
 | To List your conda environments type the following:

!!! terminal
    ```bash
    conda env list
    ```

#### Installing conda packages

Once you have activated an environment, you can install packages with the conda install command, for example:

!!! terminal
    ```bash
    conda install gcc
    ```

You can also force particular versions to be installed.  See the conda cheat sheet for details.

To list the packages installed in the currently activated environment, you can type conda list.

#### Cleaning up

##### Cleaning Cache

Once you've made your environments it can be a good idea to clean up your cache 

!!! terminal

    ```bash
    # remove index cache, lock files, unused cache packages, tarballs, and logfiles
    conda clean --all
    ```

#### Migrating an Existing Conda Environment


To move an existing Conda environment to a new location:

1. Export your current environment to a YAML file:

    !!! terminal

        ```bash
        conda env export --name existing_env > environment.yml
        ```

2. Create a new environment from the exported YAML file at your chosen location:

    !!! terminal

        ```bash
        conda env create --prefix /path/to/project_directory/env/conda_envs/myenv --file environment.yml
        ```

3. Activate the newly created environment:

    !!! terminal
    
        ```bash
        conda activate /path/to/project_directory/env/conda_envs/myenv
        ```

#### Creating an Alias for Easy Activation


To simplify environment activation, consider adding an alias to your shell configuration file (e.g., ``.bashrc`` or ``.bash_profile``):

!!! terminal

    ```bash
    alias activate_myenv="conda activate /path/to/project_directory/env"
    ```

Activate your environment using the alias:

!!! terminal

    ```bash
    activate_myenv
    ```

This method is Python-version agnostic and provides a convenient way to manage Conda environments in shared or collaborative project directories.

##### Removing environments

If you no longer need an environment the easiest way to remove it is:

!!! terminal

    === "Prefix"
        ```bash
        conda env remove -p /path/to/env
        ```

    === "Named"
        ```bash
        conda env remove -n env_name
        ```

#### Running packages from your conda environment

In order to run packages from a conda environment that you installed previously, you will first need to activate the environment in the session that you are using.  This means repeating some of the commands typed above.  Of course, you will not need to repeat the steps to create the environment or install the software, but the following may be needed again:


!!! terminal

    ```bash
    conda deactivate

    conda activate myenv
    ```

#### Installing pip packages

Many python packages that are available via PyPI are also available as conda packages in conda-forge, and it is generally best to use these via "conda install" as above.

Nonetheless, you can also install pip packages (as opposed to conda packages) into your conda environment.  However, first you should type:

!!! terminal
    ```bash
    conda install pip
    ```

before typing the desired commands such as


!!! terminal
    ```bash
    pip install numpy
    ```

If you do not install pip into your sub-environment, then either:

your shell will fail to find the pip executable, or
your shell will find pip in your base environment, which will lead to pip packages being installed into the base environment, resulting in potential interference between your conda environments
Explicitly installing pip into your sub-environment will guard against this.    


## Using conda with SLURM


In order to use conda environments within your slurm script you need to source the conda profile script so that the conda paths get set.

!!! terminal
    
    === "Modules"

        ```bash
        #SBATCH lines go here

        module purge && module load Miniforge3
        source $(conda info --base)/etc/profile.d/conda.sh
        export PYTHONNOUSERSITE=1

        conda activate /path/to/env/

        # code/programs to run go here
        ```



## Exercise - redo the variant calling

1. create a new environment in `~/obss_2025/genomic_dna/env
2. activate and then install into it the software that is found in the `modload.sh` we used.
    - Note that the names might be slightly different and you might need to google "<program> bioconda" to find out the actual package name
3. make a copy of the variant calling script  
4. 
