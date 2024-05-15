
# 2. Set up Python Environment on Levante

Python can be **environment-based**. Levante has Python environments that are maintained by DKRZ. For example, you can load a commonly used, trouble-free environment by running `module load python3/unstable`.

These environment are not allowed to install any new packages. If you wanna install a new package, the only way is to create a new environment by your self, and install the necessary packages in this new environment. `conda` is a tool to help you create new environment, and install new packages in the created environment (you could also install new package in the environment using `pip`). 


## 2.1 using existing `conda` on levante (*recommended*)
`conda` is already installed on levante. simply by `module load python3/unstable`, it comes with a conda. Type `which conda` , it tells you the location: `/sw/spack-levante/mambaforge-23.1.0-1-Linux-x86_64-3boc6i/bin/conda`
Then a new environment can be created like `conda create ...`, see [here](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#activating-an-environment).
A new package can be installed by `conda install ...`.

## 2.2 (Alternatively) install miniconda by yourself 
Alternatively, you can install `conda` or its minimal version `miniconda` .

1. Download miniconda and install it following the guide [here](https://docs.dkrz.de/doc/software%26services/machine-learning/conda.html).
2. If you say *yes* to all the options while installing, the conda initial info should already be in the *.bashrc* file. So you don't need to do anything here.
3. If you restart the terminal, check `conda list` and get the error message saying `no module conda found`, you may create (if you don't have one) or edit your *.bash_profile* like this:
```bash
vim ~/.bash_profile
```
then write the following command into this file (press *i* to enter the *insert* mode of vim)
```bash
source ~/.bashrc
```
then restart terminal and try `conda list` to see if conda is correctly installed.

## 2.3 Make a copy of the non-trouble python environment of `python3/unstable`

before we go:
- Do you really need a big environment like `python3/unstable`?
- `conda` version matters. the following tips only works with the `conda` that is loaded by `module load python3/unstable`. The one installed by miniconda doesn't work.
- The environment is very big (more than 28G). So don't do it in your $HOME directory. refer [here](https://docs.dkrz.de/doc/levante/code-development/python.html#location-of-conda-environments-and-packages) to change the location of the environment files to $WORK. 
1. change the location of the environment.
2. `cd` to a location for saving a .txt file which contains the information of packages in the `python3/unstable`. 
3. run the following code in a terminal:
```bash
module purge
module load python3/unstable
conda list --explicit > spec-file.txt
```
This saves the info of packages in the environment of `python3/unstable`. Note that two of packages are not from internet (lines starting not with "http://") It won't work in the following processes. my solution is to delete those lines. So that the new environment has most of the packages.

4. create a new environment based on the edited `spec-file.txt`:
```bash
conda create --name <you name> --file spec-file.txt
```
