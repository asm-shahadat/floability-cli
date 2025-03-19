# Floability CLI


Floability uses Conda for managing dependencies and environments. If you do not have Conda installed, you can install it via Miniconda or Miniforge. They are both light versions of Anaconda, with Miniforge having conda-forge as the default channel. Follow the instructions provided in the links below:

- [Miniforge Installation](https://github.com/conda-forge/miniforge)
- [Miniconda Installation](https://docs.anaconda.com/miniconda/install)

All conda specific dependencies for floability are specified in the `environment.yml` file. To create the environment, use the following command:

```bash
conda env create -f environment.yml
```

Then activate the new environment:

```bash
conda activate floability-env
```

Install Floability as a package:

```bash
pip install -e .
```


Now you are ready to run the `floability` command-line tool. You can examples as `backpak`s like:

```bash
floability run --backpack example/matrix-multiplication
```

## Structure of a Backpack
A backpack is a directory that contains all the necessary components to run a workflow. It typically includes a workflow file, an environment file, and any necessary data or compute-related files. The idea is to encapsulate everything needed to run a specific task or example in one place, making it easy to share and reproduce.

The matrix multiplication example has the following structure:

```
example/matrix-multiplication/
├── compute
│   └── compute.yml
├── software
│   └── environment.yml
└── workflow
    └── matrix-multiplication.ipynb
```

The compute file defines the compute resources and environment needed to run the workflow. For example, it might specify the number of workers, the type of compute resources, and any other relevant settings. Here is an example `compute.yml` file:

```yaml
vine_factory_config:
  min-workers: 2
  max-workers: 4
  cores: 1
```

The  software directory contains the environment file that specifies the dependencies needed to run the workflow. This is a standard conda environment file. Here is an example `environment.yml` file:

```yaml
name: matrix
channels:
  - conda-forge
dependencies:
  - python
  - numpy
```

The workflow file in this case is a Jupyter notebook that contains the actual code to be executed. In the matrix multiplication example, the worklow is to do the matrix multiplication on the distributed workers. We run the following python funaction as distributed tasks on the workers. 

```python
def multiply_pair(A, B):
    import numpy as np  # Only the worker environment needs numpy
    
    A_np = np.array(A, dtype=float)
    B_np = np.array(B, dtype=float)
    C_np = A_np @ B_np
    return C_np.tolist()
```
We can then run the notebook using the `floability` command:

```bash
floability run --backpack example/matrix-multiplication
```

To deppoy the workers on a batch system, we can use the `--batch-type` flag. This will submit the workers to a job scheduler like HTCondor, UGE or SLURM. 
For example:

```bash
floability run --backpack example/matrix-multiplication --batch-type condor
```