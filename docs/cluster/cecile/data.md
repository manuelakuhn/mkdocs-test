# Data organization

## Cecile data structure

The following schema represents how data are structured on Cecile. Please take a moment to look at it and read the explanation below, it will give you an overview of the cluster organization.  

```
/
├── data/
│   ├── archive/
│   │   └── projects/
│   │       ├── project_1/
│   │       └── project_n/
│   ├── groups/
│   │   ├── biopsy/
│   │   │   ├── archive/
|   |   │	- symlink /data/archive/project_1
│   │   │   └── projects/
│	│	|	- symlink /data/projects/project_1
│   │   ├── exppsy/
│   │   ├── methpsy1/
│   │   └── neuropsy/
│   └── projects/
│       ├── project_1/
|		|   └── scratch/
|       ├── ...
│       └── project_n/
│           └── scratch/
├── home/
│   └── user/
│       └── scratch/
└── software/

```

## Explaining Cecile data structure


- `/home/<username>`: *Your personal home folder*, as a rule of thumb this folder **must not** contain your main data and your main analysis. In `home` it is allowed a maximum storage space of **1GB**, see the section on data storage quotas for more information. <br>
- `/home/<username>/scratch`: *Your personal scratch folder*. `scratch` is a folder that does not get backed up, it is useful for testing and dumping temporary processed data that need to be delete soon. For a proper introduction on how to use `scratch` read the dedicated [paragraph]. 
- `/software`: *Stores the software stack*<br>  
- `/data/archive`: *Archived projects*. This folder is dedicated to archived projects.<br>  
- `/data/groups`: *Group folders*. These folders are dedicated to each group, for example `/data/groups/biopsy`. As for `home`, there is a maximum amount of storage allowed. See the section on data storage quotas for more information. <br>  
- `/data/groups/archive`: This folder contains **symlinks** (symbolic link), special files that refers to other files (see the tab below), that allow you to access archived projects directly from the group folder<br>
- `/data/groups/projects`: As for the `data/groups/archive`, this folder contains symlinks to access the projects from the group folder. </br>  
- `/data/projects`: *Projects folder*. The actual storage of all ongoing projects, which can be comfortably accessed and worked on from the `data/groups/projects` folder<br>
- `/data/projects/scratch`: *Project scratch folder*. There is a `scratch` folder also in each single `project`, please refer to the dedicated [paragraph]. 

[paragraph]: #scratch-what-it-is-and-how-to-use-it


??? note "What is a symlink?"

    ```bash
    # create an empty file
    $ touch my_file.txt

    # create a symlink of my_file.txt, we can give to the symlink the same name as the original file or a different one
    $ ln -s my_file.txt my_file_symlink.txt

    # now list the content of simlink_dir in the following way (use the flag -l)
    $ ls -l
    ```
    As you can see below, the symlink `my_file_symlink.txt` points to the original file `my_file.txt`

    ```
    lrwxrwxrwx 1 user 11 Jun 12 09:16 my_file_symlink.txt -> my_file.txt
    ```

    Now if you modify the symlink file, you can see that this change affects the original file 

    ```bash
    # write something into the simlink
    $ echo I am a symlink > my_file_symlink.txt
    # check the content of the symlink
    $ cat my_file_symlink.txt
    ```
    As expected, you can see the sentence inside `my_file_symlink.txt`
    ```
    I am a symlink
    ```
    If you check the content of the original file, you can see that also the original file contains the sentence

    ```
    $ cat file_1.txt
    ```
    ```
    I am a symlink
    ```


## How to create a project on Cecile

As mentioned in the research planning section, we adopt a specific project workflow, please follow the next steps to create a project:

- As a first step, you need to fill in a **questionnaire** about your project, this needs to be done on Cecile:
  
  - Go to your `home` on Cecile, on the command line type the following command and then press ENTER:
   
  ```bash
  $ ./run_quest
  ```
  - This command starts an interactive questionnaire on the terminal, if you need more information about the usage of the questionnare prompt, please refer to the [documentation](https://github.com/ipsy-md/Quest/blob/main/README.md).</b>  
  Please try to answer all the questions, even if you do not know the exact answer to some questions, for example: `Indicate the size of your raw data`, you should provide a reasonable estimate and your answer can be updated later. In case you are not able to answer some questions by yourself, try asking your PI or more expert members of your group, alternatively contact us and we will try to help you.

If your questionnaire has been properly completed your project will be created.

### Project structure

The project structure follows a specific BIDS structure (see examples in the tabs below). Such a structure will be generated upon project creation. In the following examples, `derivatives` and `code` are populated with potential sub-folders, these are just examples, the naming of such sub-folders are up to you and entirely project dependent, nonetheless keep the general structure as we suggest it.</b>  

!!! note "Mirroring derivatives and code structure"
    We **strongly recommend** to mirror the sub-folder names between `derivatives` and `code` to keep an intuitive relationship between code and data.

!!! tip "Naming good practices"
    Regarding naming good practices, we suggest you to look into this very helpful [page](https://book.the-turing-way.org/project-design/filenaming) 

For modalities, such as **eye-tracking**, for which there is not yet a consensus for BIDS, we recommend to use the same structure as for the other modalities and to convert the data into BIDS format anyway. [Here](https://github.com/bids-standard/bids-specification/pull/1128) you can follow (or even contribute to) the discussion around BIDS for eye-tracking 

!!! Danger "Sides effects of using a different structure"
    Failing to mantain this structure might create future issues for assigning less canonical permissions. Not adopting BIDS would make your dataset not easily interoperable and ultimately it would not be following the FAIR principles.

 

=== "fMRI project"


    A prototypical fMRI dataset:

    - `project-metadata.json`: This file is created by the cluster admin, it contains general metadata about the project.</b>  
    - `sourcedata`: It comprises the raw data (unprocessed data), in this case raw dicom files.</b>  
    - `rawdata`: It comprises the BIDS converted niftis, the relative `.json` files and `.tsv` files.</b>  
    - `derivatives`: It comprises different sub-folders which represent the macro-steps of you analysis, e.g. preprocessing, first_level  etc. These folders should follow the BIDS convention as well, and **must contain only data and no code**.</b>  
    - `code`: It includes sub-folders that **mirror** the `derivatives` sub-folder names. They **must contain only the code relative to each step, and no data at all**. It must also include a sub-folder that contains the code for the BIDS convertion.</b>   
    - `stimuli`: It includes a subfolder for the experimental code and the stimuli (e.g. images), if any.</b>  

    ```
    /data/project_name/

    └── dataset-1/
        ├── project-metadata.json
        ├── sourcedata
        ├── rawdata/
        │	├── dataset_description.json
        │	├── participants.tsv
        │	├── sub-01/
        │	├── sub-02/
        │	└── ...
        ├── derivatives/
        │	├── preprocessing/
        │	├── first_level/
        │	├── second_level/
        │   ├── classification/
        │   ├── rsa/
        │   └── ...
        ├── code/
        |   ├── bids_convertion/
        │	├── preprocessing/
        │	├── first_level/
        │	├── second_level/
        │   ├── classification/
        │   ├── rsa/
        │   └── ...
        └── stimuli/
            ├── experiment
            └── ...
    ```

=== "EEG/MEG project"

    A prototypical EEG/MEG dataset:

    `project-metadata.json`: This file is created by the cluster admin, it contains general metadata about the project.</b>  
    `sourcedata`: It comprises the raw data (unprocessed data), for example .bdf files.</b>  
    `rawdata`: It comprises the BIDS converted data, the relative `.json` files and `.tsv` files.</b>  
    `derivatives`: It comprises different sub-folders which represent the macro-steps of you analysis, e.g. preprocessing, erp etc. These folders should follow the BIDS convention as well, and **must contain only processed data and no code**.</b>  
    `code`: It includes sub-folders that **mirror** the `derivatives` sub-folder names. They **must contain only the code relative to each step, and no data at all**. It must also include a sub-folder that contains the code for the BIDS convertion.</b>   
    `stimuli`: It includes a subfolder for the experimental code and the stimuli (e.g. images), if any.</b>  

    ```
    /data/project_name/

    └── dataset-1/
        ├── project-metadata.json
        ├── sourcedata
        ├── rawdata/
        │	├── dataset_description.json
        │	├── participants.tsv
        │	├── sub-01/
        │	├── sub-02/
        │	└── ...
        ├── derivatives/
        │	├── preprocessing/
        │	├── erp/
        │	├── time_frequency/
        │   ├── classification/
        │   └── ...
        ├── code/
        |   ├── bids_convertion/
        │	├── preprocessing/
        │	├── erp/
        │	├── time_frequency/
        │   ├── classification/
        │   └── ...
        └── stimuli/
            └── experiment
    ```

=== "Behavioral project"

    A prototypical behavioral dataset:

    `project-metadata.json`: This file is created by the cluster admin, it contains general metadata about the project.</b>  
    `sourcedata`: It comprises the raw data (unprocessed data) in whatever file type you have acquired them.</b>  
    `rawdata`: It comprises the BIDS converted data, the relative `.json` files.</b>  
    `derivatives`: It comprises different sub-folders which represent the macro-steps of you analysis, e.g. preprocessing, erp etc. These folders should follow the BIDS convention as well, and **must contain only processed data and no code**.</b>  
    `code`: It includes sub-folders that **mirror** the `derivatives` sub-folder names. They **must contain only the code relative to each step, and no data at all**. It must also include a sub-folder that contains the code for the BIDS convertion.</b>   
    `stimuli`: It includes a sub-folder for the experimental code and the stimuli (e.g. images), if any.</b>  

    ```
    /data/project_name/

    └── dataset-1/
        ├── project-metadata.json
        ├── sourcedata
        ├── rawdata/
        │	├── dataset_description.json
        │	├── participants.tsv
        │	├── sub-01/
        │	├── sub-02/
        │	└── ...
        ├── derivatives/
        │	├── reaction_times/
        │	├── curve_fitting/
        │   └── ...
        ├── code/
        |   ├── bids_convertion/
        │	├── reaction_times/
        │	├── curve_fitting/
        │   └── ...
        └── stimuli/
            └── experiment
    ```

## How related projects are handled

In case you want to create a project that is related to an existing project, for example projects that are part of the same grant, project names will share a **common prefix** that is decided by the project owner upon project creation (see example below).

```
data/
├── dfg_eeg_exp/
└── dfg_fMRI_exp/

```

If two experiments are part of the same project and need to be together in the same project folder, two sub-directories containing the project structures, previously discussed, will be created:

```
data/
└── project_name/
    ├── eeg/
    └── fMRI/
```



## How to transfer data from/to Cecile

=== "Linux"


    **From your computer to Cecile:**

    ```bash
    $ rsync -rltoDvh --progress </my_computer/some_data> <user@cecile.ovgu.de:~/target_directory/>

    ```

    **From Cecile to your computer:**
    Also in this case the following command should be typed on your machine.

    ```bash
    $ rsync -rltoDvh --progress <~/some_directory_on_cecile/some_data> <~/my_computer/>
    ```

=== "macOS"

    If you want to use a gui based application, you may use [cyberduck](https://cyberduck.io/), if you are comfortable with the terminal use `rsync`

    **From your computer to Cecile:**

    ```bash
    $ rsync -rltoDvh --progress </my_computer/some_data> <user@cecile.ovgu.de:~/target_directory/>

    ```

    **From Cecile to your computer:**
    Also in this case the following command should be typed on your machine.
    ```bash
    $ rsync -rltoDvh --progress <~/some_directory_on_cecile/some_data> <~/my_computer/>
    ```

=== "Windows"

  - If you are using a [WSL](/cluster/cecile/accessing-cecile/#windows-subsystem-for-linux-wsl-option") on your Windows machine you can simply transfer data by using `rsync`:
  ```bash
  $ rsync -rltoDvh --progress </my_computer/some_data> <user@cecile.ovgu.de:~/target_directory/>

  ```

  **From Cecile to your computer:**
  Also in this case the following command should be typed on your machine.

  ```bash
  $ rsync -rltoDvh --progress <~/some_directory_on_cecile/some_data> <~/my_computer/>
  ```

  - In case you feel more comfortable with a GUI you may use the following SFTP clients:
    [Winscp](https://winscp.net/eng/index.php) and [cyberduck](https://cyberduck.io/) are GUI based applications, please refer to their documentation for proper usage. 


## How to retrieve lost data

=== "Data lost in the last 7 days"

    In case you have lost data within the last seven days, you can retrieve them by yourself. It is sufficient to go the following folder called `.zfs/snapshot` within the folder in which your data were previously hosted and transfer the data back to their previous location. Data transfer can be easily done using `rsync`.

=== "Data lost more than 7 days ago"

    You cannot retrieve such data by yourself, please contact `manuela.khun at ovgu.de`

=== "How backups work"

    Backups of `home`, `group`, `project` and `archive` are done daily by moving the `snapsthots` to the backup storage. The only folder that is never backed up is `scratch`.

## Data storage quotas

Cecile is a shared resource, as such users are subjected to certain restrictions to guarantee a fair access to resources for everybody.

**Data storage quotas set the maximum amount of storage space that can be utilized in a given folder (e.g. your home folder)**

Such restrictions are enforced to ensure that users adopt a reasoned and parsimonious choice when storing data, thus reducing that unnecessary data pollute the cluster. 
As a byproduct, quotas help you to keep your project much more tidy and force you to keep organized folders.



|  Folder  |   Quota    |
|----------|------------|
| `home`   |  **1GB**   |
| `group`  |  **?GB**   |
| `scratch`|  **1TB**   | 
| `project`|  **500GB** |



## Scratch: What it is and how to use it

`Scratch` is a particular folder that is not backed up as the other data folders (refer to How backups work for further details). For this reason is meant to be used for temporary data, for example intermediate outputs, that can be deleted with no consequences or code that you are testing. 

**How to decide which data should go on scratch:**</b>  
The following are not absolute rules, please use your common sense and domain knowledge when applying them.

- It is an intermediate output.
- It is easily reproducible, meaning that such an output is not generated by long lasting jobs.
- It is an accessory output from a specific software that you are not going to use and that is essential for your dataset.
  
!!! Danger "No important data on scratch"
    Do not store important data and code in scratch. Clean your scratch periodically. 


## Fair usage of resources

- Always remember that, once you access Cecile you are in the `head node` which is a shared space/resource. If you run a heavy computation you could either slow down or prevent others from doing regular activities on the cluster. Use it as entry point where you can run light tasks and if you need to run heavy computations, like fMRI analyses, use **Slurm**.
- If you need to run computationally heavy tests/analyses in an interactive fashion, use **Slurm interactive jobs**. By doing so you will be assigned a specific resource and you will not interfere with any other users. 
- Always think before polluting a shared space like the `group` folder with your data or other unecessary files. Do not store data or analyses in `groups/<group_name>/`, `project` is the right place to store them.
