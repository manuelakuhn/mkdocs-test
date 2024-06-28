
!!! Warning "Coding is a prerequisite"
    Data analysis is inextricably linked to coding, therefore if you are new to coding we recommend you to start reading the section about [good coding practices] before continuing with the current section." 
    [good coding practices]: open-science/code-management/good-practices/good-coding-practices
 
Data analysis constitutes one of the core aspects of your research and also the phase of your research that requires the most intense management. 

## How to build a well structured analysis

Analyses can easily become cumbersome and difficult to mantain in ways that we have not predicted.
To avoid mistakes and to waste time deciphering our data structure, we recommend a simple and intuitive workflow that you might have already adopted before:

1. Chunk your analysis according to sensible, logical macro-steps (e.g. preprocessing, first-level, second-level etc.). 
2. Create folders for each macro-step and store in there all the scripts relative to each micro-step (in the example below, the preprocessing folder constitutes a macro-step and the scripts, `preproc_step_1.py`, `preproc_step_2.py` etc., are micro-steps).
3. Each macro-step folder should also contain a pipeline script, `preprocessing_pipeline.py` in the example, that is used to combine the micro-steps allowing to run those steps all at once.
4. Each micro and macro-step should be implemented to run one subject/instance at the time. This enables you to run all the subjects in parallel in the cluster.
5. You should have a general pipeline script that combines all the macro-steps together.

```bash
my_analysis/
├── general_pipeline_script.py
├── preprocessing
│   ├── preprocessing_pipeline.py
│   ├── preproc_step_1.py
│   ├── preproc_step_2.py
│   └── preproc_step_3.py
├── signal_analysis
│   ├── signal_analysis_pipeline.py
│   ├── signal_step_1.py
│   └── signal_step_2.py
└── statistics
    └── stats.py

```

### How to build a pipeline
*******
At its core an analysis pipeline is nothing else than a collection of steps executable in a logical sequence. In this instance, we explicitly mean a script that combines all the steps in a very clear and intelligible way. 
You can create a pipeline in many ways, we will suggest you a few options with examples that you might use. Keep in mind that often analysis libraries or toolboxes offer the option of creating a pipeline (Nilearn, SPM), if this is the case, we suggest to use them.

=== "nypipe"

    [Nypipe](https://nipype.readthedocs.io/en/latest/index.html) is the most sophisticated and powerful tool to create a pipeline for cognitive neuroscience. Nypipe is a python library that enables you to create an analysis pipeline by combining different external tools (e.g. SPM, FSL) and/or custom scripts from different languages such as Python, Matlab and shell.

    ```python
        Coming soon...
    ```

=== "python"

    The following example represents only a very essential scaffolding for a potential pipeline

    ```python
    """
    Import all the necessary steps
    from your other scripts. Imagine that
    the script preproc_step_1.py contains 
    a function or a class called step_1 and
    so on for the other steps.
    """

    from preproc_step_1 import step_1
    from preproc_step_2 import step_2
    from preproc_step_3 import step_3


    class Pipeline:
        """
        Toy preprocessing Pipeline scaffolding.
        It can be used and modified to accomodate
        different kinds of analyses.

        Arguments
        ---------
        input : In this case is numpy array
                it could be any kind of data
        """

        def __init__(self, input):
            self.input = input

        def _load_data(self):
            """
            This method simply load your starting data,
            in this toy example just loading a (1xn) numpy array
            """
            return np.load(self.input)

        def _check_io(self, data, condition=None, error=None):
            """
            This method checks
            whether inputs or outputs
            meet specific criteria,
            e.g. data type, shape
            or something else.
            It throws an error in case
            data do not meet even one of criterion
            """
            for check, error in zip(condition, error):
                print(check, error)
                if check(data):
                    print("Data are fine")
                else:
                    raise ValueError(error)

        def __call__(self, pl_dict):
            """
            This method makes it callable and
            we use it to run through the for loop
            each step, while checking inputs and outpus

            Arguments
            ---------
            pl_dict : dict, it is a dictionary that contains
                    dictionaries with the name of each step as keys,
                    as many as they are the steps,
                    in each dictionary we specify the name of the function
                    or method (in case we have them in a class) that we
                    need for that step. The parameters needed by
                    the function/method, the criteria for checking
                    the correcteness of input/output and the error message
                    we would like to throw in case the input/output does
                    not satisfy certain criteria. The criteria requires
                    either a lambda function or any other function that
                    returns a boolean

            data : np.array, in this toy example it returns a numpy array,
                in a actual scenario, can return whatever type you need

            """
            data = self._load_data()
            for k, v in pl_dict.items():
                print(f"running {k}")
                # check whether criteria are met
                if v["criteria"] is not None:
                    self._check_io(data, v["criteria"], v["error_message"])
                # extract function/method from the dictionary
                func = v["method_name"]
                # extract parameters from the dictionary
                par = v["par"]
                # run step
                data = func(data, par)

            return data


    if __name__ == "__main__":
        import numpy as np

        DATA_POINTS = 20
        PL_DICT = {
            "step_1": {
                "method_name": step_1,
                "par": (0, 3),
                "criteria": [
                    (lambda x: x.shape[1] == DATA_POINTS),
                    (lambda x: x.any().min() >= 0 and x.any().max() <= 1),
                ],
                "error_message": [
                    "data points do not match",
                    "data are not with the range min >= 0 and <= 1",
                ],
            },
            "step_2": {
                "method_name": step_2,
                "par": ("mask", "ROI"),
                "criteria": None,
                "error_message": None,
            },
            "step_3": {
                "method_name": step_3,
                "par": (0.5, 1),
                "criteria": None,
                "error_message": None,
            },
        }

        PREPROC_PL = Pipeline("data.npy")
        print(PREPROC_PL(PL_DICT))

    ```

=== "make file/shell script"

    Coming soon...


=== "EEG/MEG specific pipeline"

    List of pipelines by the major EEG/MEG analysis libraries:

    - [Fieldtrip](https://www.fieldtriptoolbox.org/tutorial/scripting/) pipeline (Matlab based)
    - [EEGlab](https://eeglab.org/tutorials/11_Scripting/automated_pipeline.html) pipeline (Matlab based)
    - [MNE](https://mne.tools/mne-bids-pipeline/stable/features/overview.html) pipeline (Python based)


=== "fMRI specific pipeline"

      - [Nilearn](https://nilearn.github.io/stable/building_blocks/index.html) pipeline for Machine Learning can be extended to include also GLMs (Python based - it works with BIDS)
      - [fmriprep](https://fmriprep.org/en/stable/) is a pipeline for fMRI preprocessing which combines steps from different tools (Python and shell based - it works with BIDS)


## Why do I want to adopt this workflow?

This approach, besides being very intuitive, provides a modular and logical structure. Each micro-step can create temporary outputs that can be systematically checked for correcteness, it thus allows to identify more easily potential mistakes.
The adoption of a pipeline helps you to keep track of inputs and outputs.

