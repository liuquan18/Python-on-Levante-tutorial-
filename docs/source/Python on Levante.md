# Python on Levante

[TOC]

# 0. Introduction
This is a tutorial from a user's perspective. It shows how to code in python on [Levante](https://docs.dkrz.de/doc/levante/index.html), the latest super computer of the [DKRZ](https://docs.dkrz.de), with efficiency. With an dkrz account (account numbers start with *m* for MPI users, followed by a 6 digit number) you will be able to login into levante, get access to the data and use the software. For most use cases and if computing power is needed, this account has to be associated with a project whose computing resources are being used.

This tutorial focuses specifically on Python. The tutorial recommends VS Code as a tool for coding. However, with the exception of the first section, the tips apply to other coding-tools as well. 

# 1. Where to Python: VS code

There are several ways to get started coding on levante, and it's your choice which tool to use. The most easy way is [jupyterhub](https://jupyterhub.dkrz.de/), where we can use the [jupyter notebook/ jupyterlab](https://docs.dkrz.de/doc/software%26services/jupyterhub/index.html) in the same way as we would on our own computer. Notebooks are a useful tools to quickly hack together a script, and the integration of text, code and images, making it a nice tool for sharing ideas and plots. But there are limitations to use jupyter notebook (e.g. you can not import the functions defined in the other notebooks). Software developers usually use IDEs (Integrated development environment), that provide more functionality and more convenience, than a notebook. One of the more popular IDEs of the last few years is Microsofts [Visual Studio code (VScode)](https://code.visualstudio.com)

**Why VS code rather than jupyterhub**

- A more local-pc like file explorer.
- Flexible to allocate the compute resourses ('compute node').
- Many available extentions (e.g. git support or an AI powered copilot)
- Easier to import your own functions as packages.
- Efficient workflow: process data --> plot results --> create document
- For notebook fans: notebooks can be run inside VScode!

## 1.1 Install VS code and connect to levante

The workstations with **linux** OS in the office already have Visual Studio code software installed. Just go the application list and search for it. **Mac** book users can go to the app store or the VScode website to download and install it. 
VS code connects to levante via `ssh` (like the terminal) with support by [extentions](https://code.visualstudio.com/docs/editor/extension-marketplace). Several useful functions are only avaiable after installing the corresponding extentions like python or jupyter notebook.

### 1.1.1 Install 'Remote - SSH' from Extensions

Install the 'Remote - SSH' via the left panel "extentions".

![](https://pad.gwdg.de/uploads/b3f7db2d-de21-4ee8-b303-68d10f25de18.png)

### 1.1.2 Login levante via SSH

After you install the extention, a green button resembling `><` appears in the lower left corner. 

![](https://pad.gwdg.de/uploads/9adcfc5f-41ac-4872-a090-5c5d8b7f5784.png)

Click it, and then the `Connect to Host` button. Type in your dkrz account`m*****@levante.dkrz.de` enter and then type in your corresponding `password`. The VS code is now connected with levante! If you did set up a SSH key, it will be used for the connection. More on this in Section 3.

![](https://pad.gwdg.de/uploads/fd39ab8a-43df-474b-9f40-299427bde52e.gif)

## 1.2 Get started with VS code

Open a directory where you already have some codes with `Open`(we show further in `1. Open File or Folder`). However, we recommond `clone a repository` from GitHub (GitLab) (we show further in `2 Start a project with GitHub`). 
![](https://pad.gwdg.de/uploads/c85f63f8-ace8-4f1f-b3d3-5129722f7773.png)

### 1.2.1 Open File or Folder

You can find your working space via 'Explorer' on the left banner.
You can open any folder by type `commond + o` or `File-Open folder` at anytime.
![](https://pad.gwdg.de/uploads/0d44eb7b-09cf-40c9-8ee3-a427e7f6b91f.png)

### 1.2.2 Clone an existing project from GitHub.

It's recommonded to start a new project from GitHub. [Here](https://github.com/lkluft/example-python) is a good example of a new Python project. After creating or forking the project into your own GitHub profile, go to the project, click `code` then copy the link.
![](https://pad.gwdg.de/uploads/d03b6f54-635e-402d-b4d4-fe56e1277b14.png)

Go back to VS code, click the `Clone Repository` in the Getting started interface, paste the link in, and choose the place on levante where you wanna store the project.

## 1.3 Enable the necessary tools on VS code/Levante

there are some necessary or fancy tools avaiable on levante/VS code that can speed up your coding.

### 1.3.1  Login to Levante without password

Login Levante on vs code without password, making vs code on Levante more and more similar to local Mac. (Actually nothing matters with vs code, just make Levante recognise your local Mac.)

1. On VS code
   click the green button `><` in the left lower corner, → `Connect to Host` → `configure SSH Hosts` → `../.ssh/config`, (you can also find the config file in your local terminal in ~/.ssh/config), include the following text:

```bash
Host levante.dkrz.de
    HostName levante.dkrz.de
    ForwardAgent yes
    User m3*****
    IdentityFile ~/.ssh/levante
    IdentitiesOnly yes
    ForwardX11 yes
    ForwardX11Trusted yes
```


2. Secure way to create a key withouth being prompted for password all the time:
   Go to the *local terminal* of your mac, Create your ssh key with a save passphrase

```bash
ssh-keygen -t ed25519 -f ~/.ssh/levante
```

a file called 'levante.pub' will be generated.

3. Open the levante.pub file 

```bash
cd ~/.ssh
vim levante.pub
```

copy all the content here. 

4. Go to website https://luv.dkrz.de/, User - publick_keys, paste the text copied above.
   Type in the password (DKRZ password) once, then the VS code doesn't  ask for the password again. Levante may ask you update the code again (The validity or lifetime of the keys is six weeks). 
5. Go back to *local Terminal* of your mac

```bash
ssh-add ~/.ssh/levante
```

6. Login Levante and enter your password once before using vscode. This adds your key to an identitiy managed by the ssh agent. When using ssh you will only enter your passphrase after a restart.It seems that the step 5 is also needed after the restart. 

For other quenstions, also refers to the [2nd tip](https://pad.gwdg.de/#2-Login-levante-problem-Keep-asking-for-password) in this recipe. 

### 1.3.2 Get access to a compute node via VScode

With section 1.2, your vs code is connected to the "login node". However, the "login node" should not be used for heavy workloads, since it's shared with any users. You could use dedicated resources instead of fighting with others on shared nodes. These hardware resources named [interactive](https://docs.dkrz.de/doc/levante/running-jobs/partitions-and-limits.html#levante-partition-interactive). Starting an interactive session (here we mean "compute node" specificlly) is accompished with a single command `salloc`. use of salloc please refer [here](https://docs.dkrz.de/doc/levante/data-processing-on-levante.html):

1. On the terminal of your mac, allocate a compute node on levante, e.g,. 

```bash
salloc --partition=compute --nodes=-1 --time=08:00:00 --account mh00XX
```

2. connect VS code to the compute node. 
   Once you node is ready include your node number eg. `l20549` to the second block in the script below. Here we introduce a new host called `levante-compute`. If you connect now to levante-compute (green button left corner), it will use 'ProxyJump' to ssh on 'levante' and subsequently ssh to your compute node eg. `l20549`. 

```bash
Host levante.dkrz.de
    HostName levante.dkrz.de
    ForwardAgent yes
    User m3*****
    IdentityFile ~/.ssh/levante
    IdentitiesOnly yes
    ForwardX11 yes
    ForwardX11Trusted yes
    

Host levante-compute
     HostName l20549 #insert here the node you allocated
     ProxyJump levante.dkrz.de
     User m3*****
     IdentityFile ~/.ssh/levante
     IdentitiesOnly yes
     ForwardX11 yes
     ForwardX11Trusted yes
```

Example useage:
On levante login nodes it is not possbile to start a dask slurm cluster (no permissions), now you can start a slurm cluster from your compute nodes! see the [next topic](https://pad.gwdg.de/#5-Dask-cluster-with-VS-code). 

! Note: As a IDE, VS code is a nice platform to *edit* the code. It's also nice to *run* the code somehow. But if your code is really heavy, you can still [Submitting a batch job](https://docs.dkrz.de/doc/levante/data-processing-on-levante.html#). 

**Automatic Job Allocation**
The process of allocating resources, updating the config file with the allocated node, and connecting VS-Code to it can be automated using the 
[code-remote](https://github.com/Gordi42/code-remote) terminal application. More details on how to use the application can be found in the README file of the repository.


### 1.3.3. AI-paired programming

[**GitHub Copilot**](https://github.com/features/copilot) can be seen as code 'ChatGPT'. Given a description of the purpose in the comment, it can generate the code automatically. `tab` to accept the suggestion from AI.
![](https://pad.gwdg.de/uploads/dc6ac0af-3927-419e-a0de-fa0ea3d4cc99.gif)

Such a nice tool is not for free for sure. However, we students can use it for free.

1. Go to [website](https://education.github.com/discount_requests/pack_application) to get the students benifit. You will receive the confirmation email from GitHub after several minutes.
   ![](https://pad.gwdg.de/uploads/ae09eed8-b608-4c47-96d5-56b7435062e1.png)
2. Go to the `extension` on VS code, search for 'GitHub copolit' and install it.
3. login the GitHub to enjoy the AI-paired-coding.

[**GitHub Copilot X**](https://github.com/features/preview/copilot-x) is an even amazing AI tool. Now it's open to join the waitlist. Apart from the typical auto-code function of previous copilot, which is powered by codex model, copilot x adds the Chat option powered by GPT4. 
![](https://pad.gwdg.de/uploads/8b2445fa-f95b-4ca0-9c03-7ac3edce8294.gif)

Apart form the chat option shown above, you can also select piece of your own code, right click -> `copilot`, ask it to either debug, accelerate, or in a more elegent way... coding just becomes a work of imagination. 
if you are interested, go to the [website](https://github.com/features/preview/copilot-x) to join the waitlist, some months later, you may get the email from OpenAI saying you are off the waitlist, and a guide showing how to use it. 

### 1.3.4 Track and Synchronize with `git`

After making some modifications to the (existing) project, it's recommonded to use [`git` ](https://git-scm.com) to track the modification. You can use `git` on levante with terminal by  `module load git`. However, VS code provide much friendly interface. 
To use `git` on vs code:

1. Go to the menu panel in the left and click the Git coin, the vs code can not find the location for git yet. So help it by hand.
2. Go to the terminal in the bottom of the vs code, type `module load git`. 
3. continue type `which git` shows the path for git on levante, copy the path. 
4. Go to `Preference` - `Settings`, click the `Remote[SSH:levante.dkrz.de]` tab at the top. Search for `git.path` at the search bar at the top, paste the path copied before. Then go to the git coin again, `reload git`. 
5. Commands like `git branch`, `git add`, `git commit -m ` now are replaced by clicking.
   ![](https://pad.gwdg.de/uploads/6b8b6d00-ac6f-4568-92b1-b967b0085e89.png)
   **Note**: to make easy use of `git`, it's recommonded to login the VS code with GitHub account. For that, extention of "GitHub" maybe needed.
   For more info of `git`, refer this [summer school](https://github.com/JuliaDynamics/GoodScientificCodeWorkshop). 

### 1.3.5 Format the code with `Black`

You don't really need to write the codes looks perfectly. If you use [`Black`](https://github.com/psf/black), everyting just becomes perfect by  <kbd>Ctrl</kbd> + <kbd>s</kbd>. 

It's simple. Install the [Black Formatter Extension](https://marketplace.visualstudio.com/items?itemName=ms-python.black-formatter)


![](https://pad.gwdg.de/uploads/d8d0db72-65eb-4337-a9fc-f007c743dbac.gif)

### 1.3.6 Interactive mode and notebook

On vs code, the lines below the `#%%` will automatically be seen as a cell like in jupyter notebook. With `crtl + enter` or `shift + enter`, you can run only the code inside the cell. good to plot and debug. 

![](https://pad.gwdg.de/uploads/b6daaea8-4109-42b3-9723-8ed8c5ec993b.gif)

create a new file with `.ipynb` will generate a **notebook** file.
![](https://pad.gwdg.de/uploads/b736a120-d2af-4636-a917-14a4b21cdd9a.png)

**Use new interactive window for each python file**

"Code cells from Python scripts by default will still be executed in a same interactive window. However, you can now configure the Python extension to run separate files in separate interactive windows. Just open the settings page (File > Preferences > Settings), search for 'interactive window mode' and change the setting value to 'perFile.' Now when you run cells from different files, they will each run on their own separate window."

![](https://pad.gwdg.de/uploads/30bf8977-5543-4542-be3c-6e0c9a9aee2a.png)

### 1.3.7 debug with VS code

VS code is easy to debug. you can even debug a cell in the interactive mode. the variable values can be seen in the left panel called `run and debugger`, you can also use the `debug console` in the bottom to do some calculations. 

 > however the debugging is running in the terminal and not the interactive window

### 1.3.8 markdown in vs code

vs code can also support markdown (of course). A new file named with `.md` start a markdown file. `shift + ctrl + v` could open the preview of the markdown file. you can also plug the preview to the other half the screen to make it simultaneously visible. 

------

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

------

# 3. An example python project

This repository is a minimal example to show how to structure a Python project, how to use document function to visulize your prelimilary results and how to use GitHub Actions to automatically run tests and build documentation for your code.
**To run the example project here, go to [this GitHub project](https://github.com/liuquan18/python_example), copy the link under the 'code' and open the example project in VS code with `Clone Repository` in the welcome page.**

A well structured python project contains the following folders:
```bash
\docs               # the directory to generate the documents
\scripts            # your scripts to do processing or ploting.
\src                # the codes you wanna make as your own package. 
\test               # the codes to test the functions in `src`
.gitignore          # tell git changes of what files are not tracked.
environment.yml     # the files contain the environment configrations
LICENSE             # license file (MIT)
README.md           # this file
requirements.txt    # the packages required to install your own package.
setup.py            # to install your own package
```


## 3.1 create the environment

> NOTE: to use `conda`, please `module load python3/unstable` or  install the minconda following the guide in section 2.1.


Create a new environment with all dependencies and make it the active environment
> change the file[ "environment.yml"](https://github.com/liuquan18/python_example/blob/main/environment.yml) here to have the proper name, and include all the packages your project needed. 
```sh
conda env create -f environment.yml
conda activate python_example   # the name in the file
```
> In VS code, the environment for the scripts can be changed by type `shift+command+p`, and type `Python: select interpreter`. The environment for the jupyter notebook can be changed with the 'phone' button on the right upper. 

Inspect information about the environment and list all installed packages
```sh
conda info
conda list
```

## 3.2 make your own code in `\src` as package

Making the frequently used codes as package. In this example, `data_generator.py` and 
`md_generator.py` are made as the source code, by including a file `__init__.py` in the 
same directory and run the following bash in the root directory. `-e` make the source 
code editable. 
```sh
python3 -m pip install -e .
```
Now whenever the functions in `src` are changed, just reload the package in `script` file to update:
```python
import importlib
importlib.reload(NAME_of_Package)
```

## 3.3 run tests

'No test no code', simple example should be checked in `\test` directory. 

Run the test suite
```sh
pytest .
```

## 3.4 run your script in `\scripts`
In the example, two files are included. `make_plot.py` generate the example plots. which saves the plots under the `docs\source\plots'. 
```bash
cd scripts
python make_plot.py
```

`generate_md.py` generate the markdown file with python, inclxuding the plots just created
in the last step into a markdown note. The markdown files can also be hand made by creating the new file ending with `.md`. In order to convert the markdown files into html, all markdown files neeed to be under the `\docs` directory. 
```bash
python generate_md.py
```

## 3.5 build documentation

Auto-generate documentation. 
`conf.py` already incldued the necessary extentions to convert markdown files into html files. 
`index.rst` files should be changed to include all the markdown files that you wanna convert 
into html files. 
```sh
# activate the environment first
conda activate python_example # the name of your env
cd docs/
make html
```
The generated HTML files are stored in ` docs/build/html`.

## 3.6 upload html and build webpage

Two ways to build your webpage with your prelimilary results.

**The first way** is to use `GitHub Page`. This project include a [ci.yml](https://github.com/liuquan18/html_example/blob/393fdd8b83b339c7f0ea48b9a4388d8efb2678b2/.github/workflows/ci.yml#L1) to aumatically build the webpage. When commits are pushed or merged to the remote repository, a new version of the documentation is build and published on [GitHub pages](https://liuquan18.github.io/python_example/).
if you want to disable this more *pulic* way, you can change it in the settings of the repository. 

**The second way** for the DKRZ users, who has the swift account on levante. the following bash files named as `upload_html.sh` could be included in the `docs` directory
```bash
#!/bin/bash
# activate environment
conda activate TelSeason
module load py-python-swiftclient

# go to 
cd /docs
# upload
swift upload python_example build # change the name of the container here
```
then run the script on levante to upload the html files to swift. 
```bash
cd docs
./upload_html.sh
```
You can use swiftbrowser to check the files. To make the html as a webpage, the container need be chaged into a public. Then the webpage will be published [here](https://swift.dkrz.de/v1/dkrz_e2efc288-3ba0-499b-9bd8-d1a2580fd987/python_example/build/html/index.html) 
> Note that the link the different from just copying the link from swift_browser. Update the link above with the correct URL when you set the container on swift as public. 

---

Further information
* [Setuptools - library to facilitate packaging Python projects](https://setuptools.pypa.io)
* [Sphinx - Python documentation generator](https://sphinx-doc.org)
* [pytest - A full-featured Python testing framewor](https://docs.pytest.org)
* [GitHub Actions - automate your software workflows](https://github.com/features/actions)
* [GitHub Pages - Host web sites directly from your repository](https://pages.github.com)
* [DKRZ swift object storage](https://docs.dkrz.de/doc/datastorage/swift/index.html)

We strongly recommend the [Good Scientific Code Workshop](https://github.com/JuliaDynamics/GoodScientificCodeWorkshop) to make your code more productive. 

-----

# 4. Make python Fasterrrr:fire:
Speed comes from parallel computing. 

In general, there are two types of parallel computing that we can implement:
1. If the issue of slow speed is because of the very **big size** of a single file, you should consider [**dask**](https://docs.dask.org/en/stable/array-best-practices.html) (4.1). This also applies to the case where you read multiple files into a single dataset with `xr.open_mfdataset`. Dask do two things: 
    - enable to process data whose size is larger than the memory. 
    - parallely apply calculations on chunks thus faster. 

2. If the issue of slow speed is because of a **loop** on many files, or just a loop with many steps on a single file, consider [**mpi4py**](https://mpi4py.readthedocs.io/en/stable/)(4.2) or **parallel**(4.3). 

There are also some **small tips**( 5.4) to speed up your calculations.

## 4.1 Dask to speed up calculation on large files
### 4.1.1 Choose good chunk size (ref: [a blog to start](https://blog.dask.org/2021/11/02/choosing-dask-chunk-sizes), [xarray dask doc](https://docs.xarray.dev/en/stable/user-guide/dask.html#chunking-and-performance),[easygem dask doc](https://easy.gems.dkrz.de/Processing/dask/index.html))
> The paralle computing with xarray integrates with a library called Dask. The right chunksize is important for fast processing. here is a simple [blog](https://blog.dask.org/2021/11/02/choosing-dask-chunk-sizes)  explaining basic idea behind chunk. For optimal chunsize, here are some [advices](http://xarray.pydata.org/en/stable/dask.html?highlight=rechunk#chunking-and-performance).
> 
Array chunks can’t be too big (we’ll run out of memory), or too small (the overhead introduced by Dask becomes overwhelming). Tips to choose and change the chunk size of your dataArray:
1. Chunks should be aligned with array storage on disk 
    When reading data you should align your chunks with your storage format.you should choose a Dask chunk size that aligns with the storage chunk size and that every Dask chunk dimension is a multiple of the storage chunk dimension. e.g.,
    ```python
    ds_auto = xr.open_dataset(
        f"/fastdata/ka1081/nextgems_cycle3/ngc3028/ngc3028_{selection['time']}_{str(selection['zoom'])}.zarr",
        engine="zarr",
        chunks="auto",
    )['pr']
    ```
    You get the storage chunk size by `ds_auto.encoding["chunks"]`,it is (16, 262144).
    while you could have a look at the "auto" generated dask chunk shapes by print the data, it is (32,524288):
    ![](https://pad.gwdg.de/uploads/d31a9f06-1605-46d1-b412-d960a5044718.png)


2. Lowerboud:the number of chunks alingned with number of worker cores.
    To get the advantage of parallelization, you need the number of chunks to at least equal the number of worker cores available (or better, the number of worker cores times 2). Otherwise, some workers will stay idle.

3. Upper bound: Avoid too large task graphs. More than 10,000 or 100,000 chunks may start to perform poorly.

4. Some people have observed that chunk sizes below 1MB are almost always bad. Chunk size between 100MB and 1GB are generally good, going over 1 or 2GB means you have a really big dataset and/or a lot of memory available per core. 

5. A good rule of thumb is to create arrays with a minimum chunksize of at least one million elements (e.g., a 1000x1000 matrix). for example, you data has 180 lats * 360 lons = 64800 elements, then you could choose roughly 20 time steps in one chunk. With large arrays (10+ GB), the cost of queueing up Dask operations can be noticeable, and you may need even larger chunksizes.

-----
### 4.1.2 Dask slurm cluster to use multiple nodes/cores
Parallel computing on chunks of the dask array can be deployed on HPC (where you can use multiple nodes/cores) using [`slurm_cluster`](https://jobqueue.dask.org/en/latest/generated/dask_jobqueue.SLURMCluster.html?highlight=slurm%20cluster). 

> There is a easy example on DKRZ documents [here](https://docs.dkrz.de/blog/2020/dask_jobqueue.html#dask-dashboard) showing how to use multiple nodes with dask (simple and recommended). 

To make things more custermirized, we create a function that you can directly call in your script. 
1. create a new file called e.g slurm_cluster.py with the following codes. To make the function as a package to import, please refer to 4-step2 [make your own code in \src as package](https://pad.gwdg.de/#Step-2-make-your-own-code-in-src-as-package) in this tutorial
> Note that the file is better in the `src` folder, which is a place for your own codes as package. see [4 in this tutorial](https://pad.gwdg.de/#4-An-example-python-project-with-VS-code) for a recommonded code stucture. 

```python
# can be named as slurm_cluster.py
# the relative path should be "/src/slurm_cluster.py"
# function that you can call in your script by importing. 

from pathlib import Path
import xarray as xr
import numpy as np

from dask_jobqueue import SLURMCluster
from dask.distributed import Client
import dask.config

from tempfile import NamedTemporaryFile, TemporaryDirectory # Creating temporary Files/Dirs
from getpass import getuser # Libaray to copy things


def init_dask_slurm_cluster(scale = 2, processes = 16, walltime="00:30:00", memory="64GiB"):
    dask.config.set(
        {
            "distributed.worker.data-directory": "/scratch/m/m300xxx/dask_temp",
            "distributed.worker.memory.target": 0.75,
            "distributed.worker.memory.spill": 0.85,
            "distributed.worker.memory.pause": 0.95,
            "distributed.worker.memory.terminate": 0.98,
        }
    )

    scluster = SLURMCluster(
        queue           = "compute",
        walltime        = walltime,
        memory          = memory,
        cores           = processes,
        processes       = processes,
        account         = "mh00xx",
        name            = "m300xxx-dask-cluster",
        interface       = "ib0",
        asynchronous    = False,
        log_directory   = "/scratch/m/m300xxx/dask_logs/",
        local_directory = "/scratch/m/m300xxx/dask_temp/",
    )
    
    client = Client(scluster)
    scluster.scale(jobs=scale)
    print(scluster.job_script())
    nworkers = scale * processes
    client.wait_for_workers(nworkers)              # waits for all workers to be ready, can be submitted now

    return client, scluster
```
2. Go to your own script (eg.try_dask.py), import the function, start a client, and do the calculations.
> here your own scipt is better in `script` folder. 
```python
# %%
import xarray as xr
import src.slurm_cluster as scluster
# %%
# %%
client, cluster = scluster.init_dask_slurm_cluster()
# %%
def calcuate_spatial_mean(dir):
    ds = xr.open_mfdataset(dir,combine = 'nested',concat_dim = 'ens')
    ds = ds.mean(dim = ('ens','lon','lat'))
    return ds
# %%
def try_ds (dir):
    ds = calcuate_spatial_mean(dir)
    ds = ds.compute()
    return ds

# %%
try_ds('/work/mh0033/m300xxx/Tel_MMLE/data/CanESM2/ts/*.nc')

```
### 4.1.3 Dashboard to visulize parallel computing

visulize parallel computing can help you to determine the chunk size. you could find tutorial videos like [this](https://www.youtube.com/watch?v=N_GqzcuGLCY) explaining dashboard. 

If you are using **Jupyterhub**, after running `client, cluster = scluster.init_dask_slurm_cluster()`, you can run `client` in the nodebook, are would be presented with a link to the dashboard.

If you are a **VS code** user, Go to the bottom panel of the VS code, click the `PORTS`, if no Port there, the default port should be '8787', add port '8787', click the internet icon, you are led to the fancy dashboard in your default browser. 
> If port 8787 is occupied add ```scheduler_options = {'dashboard_address': ':8989'},```  to the SLURMCluster dictonary and connect the port eg. 8989 as mentioned above. 

Now the dashboard (which you always found on Moritz's screen) can be used to visulize the parallel computing. 
![](https://pad.gwdg.de/uploads/c2f6ceeb-610a-40e6-b890-87d9f3b54ad5.gif)




### Open questions:
* Having the above question in mind how to choose the number of processes and number of cores?
* If very large data sets are calculated, how to deal best when dask is spilling data to disk larger then total memory used? If chunks are sufficiently small, why is dask not able to work more linear and avoid "spill to disk"?


## 4.2 Using mpi4py to speed up loops
### 4.2.1 concept of mpi4py
**Just submit you python code to [slurm](https://docs.dkrz.de/doc/levante/running-jobs/slurm-introduction.html) with `sbatch` doesn't really make your code run faster.** Your python code should also be wrote in the supported way.  

[mpi4py](https://mpi4py.readthedocs.io/en/stable/) (MPI for python)provides Python bindings for the Message Passing Interface (MPI) standard, allowing Python scripts parallely run on allocated nodes and processers. 

The example codes below shows an example code, totally with 12 steps (loops), runs on 2 nodes (node:1, node:2), so that each node runs 6 tasks. since there are 4 cores allocated in each node. and there are  6 tasks for each node, the 4 cores run [2,2,1,1] task correspondingly (see plot below). 
![](https://pad.gwdg.de/uploads/527c4970-ef3e-430d-98a6-77a02c381172.png)

### 4.2.2 An example problem
You can easily adapt your own codes based on the following example.
**task**: Imagine that you have to calculate spatial mean of five models (scenarios) outputs. Assume that all outputs are yearly data. Assume we have 5 models, each with 100 years.
**result**: the result would be five .nc files, each for a single model (scenario), each contains 100 years of values.
**soulution**: Make the data from different models (scenarios) run on different nodes. Make the calculation of different years of one model run on different cores. 
**code**: A Python code to do the calculation, and a bash code to submit it to compute node. 
* The python code recevies pararmeter *node_id* as an index to determine which model (scenario) is going to run on this node.
* *npro* in the following python code is the number of cores that you use for one node, in this case, 10 cores are allocated, therefore, 10 years are always calculated parallely on one node (* 5 nodes). 
* *rank* is a id for the cores. In this case, the rank is [0,1,2,3,4,5,6,7,8,9], that can be used to determine which periods of years should be run on this core. 

### 4.2.2 An python code to adapat
1. A python script, called **python_mpi4py.py**. which would run on a single node
```python
#%%
import sys
import numpy as np
import mpi4py
import time as pytime
import pandas as pd
import xarray as xr
import logging
#%%
# logging configure
logging.basicConfig(level=logging.INFO)

# get the parameters from bash / terminal
node_id = int(sys.argv[1]) # the index for node

# model (scenario) list, from which the node_id termines which model (scenario) is running on this node. 
models = ['MPI_GE','CanESM2','CESM1_CAM5','GFDL_CM3','MK36']

# the model that is run on this node
model = models[node_id]

logging.info(f"Node {num+1} is working on {model}")

# Then we have the location for this node to read data
data_dir = f"/work/mh00xxx/xxx/{model}/xxx.nc"
data = xr.open_dataset(data_dir)


# === mpi4py ===
try:
  from mpi4py import MPI
  comm = MPI.COMM_WORLD
  rank = comm.Get_rank()  # [0,1,2,3,4,5,6,7,8,9]
  npro = comm.Get_size()  # 10
except:
  print('::: Warning: Proceeding without mpi4py! :::')
  rank = 0
  npro = 1


# the whole year lists 
years = data.dt.year # 100 years
# the years that is going to be run on each core
year_single_core = np.array_split(years, npro)[rank]  # 10 list of sublist, each sublist includes 10 years. 


FLDmean = [] # an empty list for storing outputs of different cores

# loop on the years that one core need to run
for step, year in enumerate (year_single_core):
    logging.info(f"Core {rank+1} of node {node_id +1}: {step+1}/len(year_single_core) current year: {year}")
                 
    data_year = data.sel(time = year)
    fldmean = data_year.mean(dim = ('lon','lat'))
                 
    # here you can alread save the result from each core independently. 
    # of course the results from different cores can be collected.
    FLDmean.append(fldmean)
    
logging.info ("collecting the results from all processes")
# gather the index from all processes
index_allens = comm.gather(FLDmean, root=0)

if rank == 0: # the following concatnate part only need one core. 
    # now you want to make them into a single DataArray again
    FLDmean = [item for sublist in FLDmean for item in sublist]
    FLDmeanx = xr.concat(FLDmean, dim = 'time')

    # now save the final results.
    FLDmean.to_netcdf(...)
else:
    pass

```
### 4.2.3 submit the python code with bash to a single compute node
A bash code called **submit_python_mpi4py.sh** to allocate a single node from levante. change your account. 
```bash
#!/bin/bash
#SBATCH --job-name=parallel
#SBATCH --time=00:01:00
#SBATCH --partition=compute
#SBATCH --nodes=1
#SBATCH --ntasks=10
#SBATCH --account=mh0033

conda activate YOURENVIRONMENT # change the name for the environment
mpirun -np 10 python -u python_mpi4py.py $1 # the id you give for the node. eg., 0
```
To run it, simply:
```bash
sbatch ./submit_python_mpi4py.sh 0 # 0 for example, note that the number here is used in your python code to choose model (scenario) that is going to run on this node. 
```

### 4.3.4 submit the bash code using multiple nodes
You can do this mannually by hand, just run the above command line multiple time with differnt *node-id*.

You can also use a python code called **master_submitter.py** to run the bash code above 5 times (number of node). 
```python
#!/usr/bin/env python
#%%
import os
import numpy as np

#%%
nmodels = 5

#%%
for node_id in range(nmodels): 
  os.system(f"sbatch ./submit_python_mpi4py.sh {node_id}") # 1,0,4, 2,4,8, 3,8,12
# %%

```
run the master_submitter.py either in interactive mode or in the terminal, that's it.


## 4.3 Parallize with GNU parallel
! Strongly recommend to read the [tutorical](https://doi.org/10.5281/zenodo.1146014), at least the first 2 chapters, within 20 mins. 

*GNU parallel: do the same processing (for example, one-line-cdo command, a bash function, or a python code) for different input*


Before allocating more nodes to speed up your cdo tasks, take a look at how much of the node's ressources you are actually using (ssh to node on which your job is running and then ```top```). Many cdo commands are not memory intensive, so the node's resources are not fully used.

Here is a simple option to parallize your code by making use of the ```module parallel``` in combination with ```find``` to create a list of files. 
The beauty of a one-liner (see line 20):

```bash
#! /bin/bash
#SBATCH --job-name=cdo_slev
#SBATCH --time=02:00:00
#SBATCH --nodes=1
#SBATCH --output=log.o-%j.out
#SBATCH --error=log.o-%j.out
#SBATCH --account=mh0033
#SBATCH --partition=compute
### SBATCH --mem=256G
#SBATCH --exclusive 

exp=9
variable=v
path_to_input_files=/work/bm1102/m300602/proj_smtwv/icon-oes-zstar4/experiments/smtwv000${exp}/outdata_${variable}/
infilename=smtwv000${exp}_oce_3d_${variable}_PT1H_*.nc
path_to_output_files=/work/bm1239/m300878/smtwv_levs/2/exp${exp}/outdata_${variable}/

echo "Processing in parallel..."
module load parallel
find ${path_to_input_files} -name ${infilename} | parallel --jobs 100 --eta cdo -sellevidx,17,19,26,28,42,43 -selvar,${variable} {} ${path_to_output_files}{/}
echo "read/write finished..."

echo "Renaming files..."
find ${path_to_output_files} -name ${infilename} | xargs rename 3d lev
echo "Job finished..."

infilename=smtwv000${exp}_oce_lev_${variable}_PT1H_*.nc
echo "Drop Coordinate Variables..."
module load nco
find ${path_to_output_files} -name ${infilename} | parallel --jobs 50 --eta ncks -C -O -x -v clat_bnds,clon_bnds,clat,clon {} ${path_to_output_files}{/}
echo "Drop Coordinate Variables finished..."
```


Here is an example showing how to parallelly apply values from lists to a bash function (emergetime). The function has two parameters  (`level` and `member`). `parallel` made it possible to run the function parallelly, with the values coming from two lists (the two lists after `:::`) giving to the two parmeters.
```bash
#!/bin/bash
module load cdo

# Function to merge time levels
mergetime() {
    level=$1
    member=$2
    echo "Merging time for level ${level} member ${member}"
    sleep 1

    # some cdo command for example here
    # cdo -mergetime ${file_list[@]} /scratch/m/m300883/20CR_all_plev/HGT${level}/HGT${level}_1850-2015_mem${member}.nc
    }
    
export -f mergetime

parallel --jobs 80 mergetime ::: 925 850 700 500 400 300 200 ::: {001..080}
```
----

## 4.4 Tips to be faster (copied from [here](https://docs.xarray.dev/en/stable/user-guide/dask.html#optimization-tips))

1. Do your spatial and temporal indexing (e.g. .sel() or .isel()) early in the pipeline, especially before calling resample() or groupby(). Grouping and resampling triggers some computation on all the blocks, which in theory should commute with indexing, but this optimization hasn’t been implemented in Dask yet. (See Dask issue #746).

2. More generally, groupby() is a costly operation and will perform a lot better if the flox package is installed. See the flox documentation for more. By default Xarray will use flox if installed.
3. Save intermediate results to disk as a netCDF files (using to_netcdf()) and then load them again with open_dataset() for further computations. For example, if subtracting temporal mean from a dataset, save the temporal mean to disk before subtracting. Again, in theory, Dask should be able to do the computation in a streaming fashion, but in practice this is a fail case for the Dask scheduler, because it tries to keep every chunk of an array that it computes in memory. (See Dask issue #874)
4. Specify smaller chunks across space when using open_mfdataset() (e.g., chunks={'latitude': 10, 'longitude': 10}). This makes spatial subsetting easier, because there’s no risk you will load subsets of data which span multiple chunks. On individual files, prefer to subset before chunking (suggestion 1).
5. Chunk as early as possible, and avoid rechunking as much as possible. Always pass the chunks={} argument to open_mfdataset() to avoid redundant file reads.
6. Using the h5netcdf package by passing engine='h5netcdf' to open_mfdataset() can be quicker than the default engine='netcdf4' that uses the netCDF4 package.


# 5 Tips

## 5.1 Split the monitor

In any file, press <kbd>Ctrl</kbd> + <kbd>K</kbd>, then <kbd>O</kbd> (without <kbd>Ctrl</kbd>). You now should have two separated windows of vscode that you can distribute on two different screens. You could have a window with `script` code in the horizontal screen, and `src` codes in a vertical screen. 

![](https://pad.gwdg.de/uploads/180a4af2-7ddf-4f88-9616-2b8beb014bd9.png)


## 5.2 Login levante problem: VScode keeps asking for the password
If you are stuck in a password-asking loop when you try to establish a connection to *levante*: try to remove the folder `~/.vscode_server` on levante. Access *levante* via `ssh` from your terminal and remove it via: 
```bash
rm -rf ~/.vscode_server
```
This may remove all the extentions of VS code you already installed. You may need to reinstall them. 

## 5.3 login problem and install vs code extention problem
If you do everything correctly, but still have the problom of login, can not install vs code extenstion, or even worse, can not activate python kernel, there is high probability that **there are no space left in your \home directory**
you can check the space that you used in your home directory at 
website https://luv.dkrz.de/, User panel.

## 5.4 Create a new kernel in the terminal
Kernels are programming language specific processes that run independently and interact with the Jupyter application and their user interfaces. It is recommended to use different virtual environment for each project using VSCode. 

https://docs.dkrz.de/doc/software&services/jupyterhub/kernels.html

Please, follow these steps (Terminal): 

1. Create conda virtual environment
```bash
conda create -n [name of virtual environment] python = [preferred python version i.e. 3.9]
```
Here, it is preferable to create conda/virtualenv environments in /work (in the corresponding project) considering the limitation of the disk usage. For doing this,
```bash
conda create --prefix /work/project_id/$USER/[name of virtual environment] python = [preferred python version i.e. 3.9]
```

2. Add virtual kernel to the jupyter
```bash
conda activate [name of virtual environment]
pip install jupyter
pip install ipykernel
python -m ipykernel install --user --name [name of virtual environment] --display-name "[name of the kernel which can be presented on jupyter]"
```
3. Check the list of virtual kernel
```bash
conda env list : check the list of the virtual environment folders
```
![](https://pad.gwdg.de/uploads/6f420762-1437-4538-8a66-06b351246974.png)
Also, you can check your Kernel specifications in ~/.local/share/jupyter/kernels/[name of virtual environment]/kernel.json


## 5.5 Working with Markdown files in vscode

### Themes
If you work with dark themes in vscode your figures might not be shown properly (eg. if you create them for a white background). A useful extension to overcome this is "Markdown Preview Github Styling". In extension seetings change "auto match" to single theme: light



### Figures
If you write your own website (see 4.5 and 4.6 above) and you want to include your markdown files with your figures without uploading them first (generating a link to a website like (https://pad.gwdg.de/uploads/XXX.png) 

You can include figures with a relative path (easy to update/modify!):
```
! [](../notebooks/images/figure.png)
```
* disadvantage: no options to scale figure with markdown syntax

More options for configurations of your figure with img tag:
eg:
```
<p><img src="../notebooks/images/figure.png"  width="60%" ><BR CLEAR=LEFT>
```
Gives a scaled figure aligned to the left.

> Problem: depending on your Makefile your figure might not be displayed on ...

# 6. Useful websites/reference/documents
1. [**Getting started with Levante**](https://indico.dkrz.de/event/40/attachments/32/50/20220428_LevanteIntro.pdf) is a slide used in a Levante introduction seminar.
2. [**easy.gems**](https://easy.gems.dkrz.de/index.html) enormous tips of processing high resolution data from DKRZ nerds. 
3. [**Intake-ESM**](https://intake-esm.readthedocs.io/en/stable/index.html) a more cataloging way of loading data to do analysis.
4. [**cdo**](https://code.mpimet.mpg.de/projects/cdo/embedded/index.html#x1-4830002.8.11) documents that can be easily searched by <kbd>Ctrl</kbd> + <kbd>f</kbd>. 
5.  [Good Scientific Code Workshop](https://github.com/JuliaDynamics/GoodScientificCodeWorkshop)