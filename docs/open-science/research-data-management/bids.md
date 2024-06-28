# BIDS 

If you are following the project workflow, by now you should have a project set up on the cluster and you might be in the process of organizing your **raw data**. The next step is to convert your data into BIDS format.

[**BIDS**](https://bids.neuroimaging.io/get_started.html) is an acronym that stands for **Brain Imaging Data Structure**. It is a standardized format to organize and describe neuroscientific data (see the schema below).</b>  
In a nutshell, BIDS provides a framework for standardizing file names, folder structures and metadata for different kinds of data such as neuroimaging, behavioral, physiological etc.  
We recommend you to go through the [BIDS Starter Kit pages](https://bids-standard.github.io/bids-starter-kit/) to understand more in depth the concepts and the phylosophy behind BIDS.

```bash
.
├── CHANGES
├── dataset_description.json
├── participants.tsv
├── README
└── sub-01
    ├── anat
    │   ├── sub-01_T2starw.nii.gz
    │   └── sub-02_T1W.nii.gz
    ├── func
    │   ├── sub-01_task-mybrillianttask_bold.nii.gz
    │   └── sub-01_task-mybrillianttask_events.tsv
    └── task-mybrillianttask_bold.json

```

## Why BIDS?

- It provides a logical and intelligible structure for data of different kinds.
- It makes your data clear and reusable by other researchers, thus increasing the chance of collaborations and the chance of being cited.
- It helps you to organize your data without coming up by yourself with an efficient and flexible structure.
- It has been adopted by all the major tools for neuroscientific analysis.
- It improves science and also your science. 
  
## How to BIDS 

In the following tabs, we explain how to convert your data into BIDS for different modalities. We focused only on specific tools that are either widely used or provide some advantages for the users.
If any issue occurs, you need help or have a request for a specific tool that we do not cover here, please contact emanuele.porcu at ovgu.de.

=== "fMRI"

    ### Using heudiconv to convert fMRI data to BIDS

    [Heudiconv](https://heudiconv.readthedocs.io/en/latest/) is a python library which helps you to convert f/MRI data to BIDS with little effort. You do not need to be very proficient in python to use it. YOu can either follow the following tutorial or use the [tutorials](https://heudiconv.readthedocs.io/en/latest/tutorials.html) provided by the heudiconv developers.

    **Step 1**
    
    In order to convert correctly your data, heudiconv leverages on so called heuristics.</b>     
    Heuristics are rules that you provide to the software to specify which files to convert.
    
    You must specify these rules within a python script that is then passed to heudiconv. You do not need to create this script from scratch, you can generate a template by running the following line on the terminal:

    ```bash
    $ heudiconv --files your_dicom_file -o bids_output_dir -f convertall -s number_of_the_subject -c none
    ```

    The previous command does **not** convert any of your data, but it creates, within the output directory, a hidden folder, `.heudiconv`, containing a template of the script `heuristic.py` and a `dicominfo.tsv` which lists all the information about the dicom files necessary for specifing the heuristics. 
    
    
    !!! note "Check your dicoms"
        It is a good practice to extract your dicom files and check by yourself the file names and whether they match with the names stored by heudiconv in the `dicominfo.tsv`.


    !!! Danger "Delete .heudiconv" 
        Before running the actual BIDS convertion, make sure to delete the **.heudiconv** directory, otherwise the conversion will not produce a correct BIDS dataset.

    **Step 2**
    
    The following example represents a `heuristic.py` script taylored for a specific dataset. You can also use this template and adapt it to your dataset.


    ```python

    from __future__ import annotations

    import logging
    from typing import Optional

    from heudiconv.utils import SeqInfo

    lgr = logging.getLogger("heudiconv")


    def create_key(
        template: Optional[str],
        outtype: tuple[str, ...] = ("nii.gz",),
        annotation_classes: None = None,
    ) -> tuple[str, tuple[str, ...], None]:
        """
        This function allowes you to create
        the keys necessary to extract the files
        that you want to convert.
        """

        if template is None or not template:
            raise ValueError("Template must be a valid format string")
        return (template, outtype, annotation_classes)


    def infotodict(
        seqinfo: list[SeqInfo],
    ) -> dict[tuple[str, tuple[str, ...], None], list[str]]:
        """Heuristic evaluator for determining which runs belong where

        allowed template fields - follow python string module:

        item: index within category
        subject: participant id
        seqitem: run number during scanning
        subindex: sub index within group
        """

        # Create keys: This section is specific to each project
        # you need to create your own keys and the info dictionary

        t1w = create_key("sub-{subject}/anat/sub-{subject}_t1w")
        task = create_key("sub-{subject}/func/sub-{subject}_task-foodchoice_run-{item:01d}_bold")
        fmap = create_key("sub-{subject}/fmap/sub-{subject}_fieldmap")
        epi = create_key("sub-{subject}/fmap/sub-{subject}_epi")

        # the dictionary info must contain your keys
        info: dict[tuple[str, tuple[str, ...], None], list[str]] = { 
                t1w: [],
                task: [],
                fmap: [],
                epi: []
                }

        for s in seqinfo:
            """
            The namedtuple `s` contains the following fields:

            * total_files_till_now
            * example_dcm_file
            * series_id
            * dcm_dir_name
            * unspecified2
            * unspecified3
            * dim1
            * dim2
            * dim3
            * dim4
            * TR
            * TE
            * protocol_name
            * is_motion_corrected
            * is_derived
            * patient_id
            * study_description
            * referring_physician_name
            * series_description
            * image_type
            """

            # The following conditional statements constitute
            # the core of the heuristics. 
            # Modify them according to the dicom file names and 
            # other features (like data size) that 
            # can help select the correct files

            if "t1_mpr" in s.protocol_name:
                info[t1w].append(s.series_id)
            elif 'fMRI_SMS2_2.2iso_66sl_TR2_run' in s.protocol_name and s.dim4 == 200:
                info[task].append(s.series_id)
            elif 'field_mapping' in s.protocol_name:
                info[fmap].append(s.series_id)
            elif 'EPI' in s.protocol_name:
                info[epi].append(s.series_id)
            
        return info
    ```
    
    The essential parts of the script that you need to modify are the **keys** and the actual **heuristics**:

    - **Keys:** In the `dicominfo.tsv` (you can find it in `.heudiconv`) file you find the exact naming of the dicom files, you need to go through the list of files and decide which of them are important for your analysis. Once you have made your decision, you can create the proper keys by following the naming conventions explained in the [BIDS tutorial](https://bids-standard.github.io/bids-starter-kit/folders_and_files/files.html)
    
        Assume you have a T1 weighted image, you need to create a proper key for it:

        ```python
        t1w = create_key("sub-{subject}/anat/sub-{subject}_t1w")
        ```
        Note that we have specified the subject folder `sub-{subject}/` where `{subject}` is a placeholder that heudiconv fills in with the correct subject ID. We have specified the folder `anat/` dedicated to all the anatomical images and we have specified `tw1` which is the standard name for T1 weighted images. Keep in mind that you might have have different anatomical images, please refer to the BIDS main website.   

        Let's create another key for the bold images, one per run:

        ```python
        task = create_key("sub-{subject}/func/sub-{subject}_task-foodchoice_run-{item:01d}_bold")
        ```

        In this example it is important to notice that we follow the same pattern as for the T1 weighted image, but this time we have specified the folder `func`, which as you should know, stands for **functional**. Additionally we have specified the task name
        `task-taskname`, the runs and the `bold` suffix that indicate the kind of file we are dealing with.

        It is superfluous to say that we need to create as many keys as the types of files we need to include in our BIDS convertion.

    - **Heuristics:** Now we need to create a few basic rules to tell heudiconv which files to extract.
    
        In the following example we simply match the name of a `dicom` we would like to include in the BIDS conversion with the file names contained in `s.protocol_name`, if there is a match we add the file in the `info` dictionary.

        ```python
        if "t1_mpr" in s.protocol_name: # name matching
            info[t1w].append(s.series_id) # append to the dictionary
        ```
        Sometimes a file name matching is not enough to extract a the correct file, so we can combine more rules like the following one, in which we match the file name and the file size.

        ```python
        elif 'fMRI_SMS2_2.2iso_66sl_TR2_run' in s.protocol_name and s.dim4 == 200:
            info[task].append(s.series_id)
        ```

        As you might guess you need to create rules on a case by case basis.


    **Step 3**

    Now you just need to run heudiconv for the actual conversion with the following line, please for further information about heudiconv specs visit the software page or type `heudiconv --help` in the terminal:
    

    ```bash
    $ heudiconv --files <file_containing_dicoms> -o <directory_for_bids/> -f <heuristic.py> -s <subject_number> -c dcm2niix -b --overwrite
    ```

    Note that heudiconv creates the whole BIDS structure, the sidecar as `.json` files, and the events files as `.tsv`.</b>   `events.tsv` are just tabular files containing the header: `onset	duration	trial_type	response_time	stim_file`. Your task is to complete these files according to the bids conventions.

=== "EEG/MEG"

    ### EEG BIDS recommended formats

    Given the variety of EEG data formats, BIDS recommends two main formats (we also strongly suggest to use these two formats):

    1. European format: `.edf`
    2. BrainVision format: `.vhdr, .vmrk, .eeg`

    Other BIDS accepted formats, although not recommended, are:

    3. Biosemi: `.bdf`
    4. EEGLAB: `.fdt .set`

    ### EEG conversion with Fieldtrip (Matlab)



    ### EEG conversion with MNE (python)

=== "Behavioral"

    BIDS for behavioral data follow conventions similar to those of the events files for neuroimaging data. 
    We recommand to read carefully the [description](https://bids-specification.readthedocs.io/en/stable/modality-specific-files/behavioral-experiments.html#behavioral-experiments-with-no-neural-recordings) in the BIDS website and to try to adopt the BIDS conventions when creating the log files during data acquisition. We do not provide any example due to the large variety of structures and formats that researchers use for their experiments.

=== "Eye-tracking"

    Coming soon...

## Data Version Control

One of the most important aspects of data management is to keep track of the evolution of your data. This task might be particularly error prone if done manually, it is, thus, advisable to use a systematic and automated approach.</b>  
As for software development also data can be tracked via **version control**. There are not many version control tools dedicated to data, the main one is [Datalad](https://www.datalad.org/), which in the last years has been heavily adopted in cognitive neuroscience research.</b>   
Datalad does not simply track the changes of your data, but also records their relationships. Beyond that, the datalad ecosystem includes other tools that can help you to make your science reproducible and collaborative.


The best way to start with Datalad is to go through the [Datala-handbook](https://handbook.datalad.org/en/latest/).
Dalalad is natively used through the command line, like [git](https://git-scm.com/), but for those who are less comfortable with the terminal there is also a [**GUI version**](http://docs.datalad.org/projects/gooey/en/latest/). 