
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

