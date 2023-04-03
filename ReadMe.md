# Change Log

+ 3-APR 2023
	+ Added the instruction for using Anaconda3.
+ 10-MAR 2023
	+ Revised the `job.psb` so that the examples for applying more than 1 gpus are provided.
+ 3-MAR 2023
	+ Revised the `job.psb` so that the container can know the gpu index. This prevent the bug `AssertionError: Invalid device id` when loading a checkpoint. The bug is caused by the unrecognizable gpu index assigned by the PBS-PRO to the variable `CUDA_VISIBLE_DEVICES`, which further caused `torch.cuda.is_available() = True`, yet `torch.cuda.device_count() = 0`. The revised `job.psb` manually assigns the gpu index instead. 
+ 1-MAR 2023
	+ Uploaded the correct container file (for [option 2](#option-2)).

# Table of Content<a name="table-of-content"></a>

+ [Step 1: Preparation (In Local)](#step-1-preparation-in-local)
	+ [Option 0 (best): Load anaconda3 and then do it in the usual way](#option-0)
	+ [Option 1: Create your own Singularity container](#option-1)
	+ [Option 2: Use a built container and pip install any missing packages every time you submit your jobs](#option-2)
	+ [Option 3: Load the module from NSCC directly](#option-3)
+ [Step 2: Run (In NSCC)](#step-2-run-in-nscc)
+ [Endnote: Useful commands in NSCC](#endnote-useful-commands-in-nscc)

## Step 1: Preparation (In Local)<a name="step-1-preparation-in-local"></a>
[Return to Table of Content](#table-of-content)

First, we need to prepare the Python environment for our code. There are three options:

+ Option 0: The best choice is to load the Anaconda3 module directly and then create your virtual environment.
+ Option 1: Create your own Singularity container.
	+ Pros: Fully customizable. You can install any packages you want and built them all in the container. It's an **one-off effort**.
	+ Cons: Struggling for Singularity installation and container building. 
	+ Comment: Highly recommended and fully covered by this tutorial.
+ Option 2: Use a built container and pip install any missing packages every time you submit your jobs.
	+ Pros: Plug-and-play, no messy installation and building.
	+ Cons: Need to install missing package for each submitted job. (Mostly, missing packages are not so many. Therefore, it’s acceptable.)
	+ Comment: Use this if you want a quick try.
+ Option 3: Load the module from NSCC directly.
	+ Pros: Say goodbye to Singularity.
	+ Cons: Missing core packages. For example, the Pytorch module missed scipy, pandas, ....
	+ Comment: Never worked for me. It may work if your code is written purely using Pytorch API.

#### Option 0<a name="option-0"></a>
[Return to Table of Content](#table-of-content)

Login to your NSCC, type

```
module avail
```

so that you will see the list of all the modules. Find the Anaconda3. It should be something like `anaconda3/2022.10`. Load it by typing

```
load anaconda3/2022.10
```

Now you can use the `conda` command. Then the rest is all the same. By **same** I mean you can `condo create` your environment and then `condo install` or `pip install` your packages. This option is available for the new nscc. By using this you don't need to bother with the messy singularity container.

#### Option 1<a name="option-1"></a>
[Return to Table of Content](#table-of-content)

TLDR: in your host system, install Virtualbox with a Ubuntu 22 as the guest system, install Singularity in the guest system, build the container there, then fetch it in your host system. See below for details.

The installation of virtualbox and the Ubuntu 22 is straightforward and there are plenty of tutorials on this. Therefore I only highlight a few things:

+ Virtualbox [download](https://entuedu-my.sharepoint.com/:u:/g/personal/su012_e_ntu_edu_sg/EVzxRsD8Jx5BmBxsbxuFCNAB5_wWMezZjcy0AxwDHVtvlw?e=DYe8yC). (Why? It's light-weight, cross-platform, and open-sourced. It's a neat solution because we can simply remove the guest system to ditch the messy Singularity from your PC. I share it using my personal url since NTU blocked the official link.)
+ Ubuntu22 [download](http://www.releases.ubuntu.com/22.04/ubuntu-22.04.2-desktop-amd64.iso) 


![](./images/vb1.png)
> Install the Ubuntu guest system on Virtualbox. Leaving the username and pwd by default is totally fine (the password is `changeme`). Check the addition iso so that the host/guest sharing folder and copyboard can be available later. Note that the addition iso should be available in the root directory of your VirtualBox once you installed it.

![](./images/vb1_1.png)
> Specify the hardware for the guest system according to your host system. Usually 8G and 1/4 of your CPUs are enough.

![](./images/vb1_2.png)
> Specify the disk space for the guest system. Don't check `Pre-allocate Full Size`.

![](./images/vb1_3.png)
> Once the settings are completed, the guest system will launch. Choose `Try or install Ubuntu` to install the Ubuntu. 10-20 mins later the guest system will be ready.

![](./images/vb1_4.png)
> Login to the guest system using the default username and password. Note that in `Device` you should enable the sharing folder and bidirectional clipboard, so that we can copy/paste files/commands between the host and guest systems conveniently.

![](./images/vb2.png)
> Check auto-mount and make permanent for the sharing folder.

![](./images/vb3.png)
> You should see the shared folder in your ubuntu guest system after that. In your host system, put the tutorial folder there so that the guest system can access the shell script for Singularity installation. (Note, if you cannot see the shared folder, try restarting the guest system.)

![](./images/vb1_5.png)
> Don't be hasty! Let's take a snapshot. A Snapshot is a checkpoint of the guest system. Later, if you screwed up your guest system, you can simply go back to the fresh stage and no need to reinstall Ubuntu again! To do so, click `Take` to take a snapshot for the fresh Ubuntu. This one is where "all things start". When you want to go back in time, choose the "Fresh Ubuntu" and click `Restore`.


![](./images/vb5.png)
> In the tutorial directory of your guest system, right click --> Open in Terminal. First off, type `su` followed by the password `changeme` to switch to the root user. Then, type `usermod -aG sudo vboxuser` to add the user to sudoer group, and type `su vboxuser` to switch back to the user. Now, type `chmod +x install_singularity.sh` to change the shell script to executable. Finally type `./install_singularity.sh` to install the messy Singularity! 

![](./images/vb6.png)
> The installation can take 10-20 mins. After which, type `singularity` in the terminal to confirm that the command is recognized.

![](./images/vb7.png)
> In your tutorial directory of your HOST SYSTEM, edit `my.def`. In Line 18 follow the example to add any Python packages you want. Just like how you normally install them in your conda environment. This definition file tells Singularity to build these Python packages into the container. Note that the Nvidia A100 demands cudnn8 and higher, which corresponds to a newer version of pytorch. 

![](./images/vb8.png)
> Finally, in the terminal, type `sudo singularity build  pt113.sif my.def` to build the container. After 10-20mins, a container named `pt113.sif` will be generated in the current directory (shared with the host system), and you should be able to access it from your host system. It is now ready to be uploaded to NSCC.



#### Option 2<a name="option-2"></a>
[Return to Table of Content](#table-of-content)

Simply download the built container from [this link](https://entuedu-my.sharepoint.com/:f:/g/personal/su012_e_ntu_edu_sg/EmfmArJu9LtJhgGbPHyGMqgB0t33SoyM2Y_pVXgj94EBdg?e=gOZvT6), and proceed to the next step.

The downloaded file is built following the steps and settings in Option 1, with `scipy scikit-learn matplotlib pandas` being installed. They may meet your common usage already. To further install more packages on-the-go, check `job.psb`.

#### Option 3<a name="option-3"></a>
[Return to Table of Content](#table-of-content)

Login to your NSCC console, type `module avail` to see the available modules, and `module load pytorch` to load any modules you want.

Note that the module needs to be loaded in the runtime of your job, therefore, the `module load pytorch` commands shall be included in the job definition. You may study this option on your own but I never make it.


## Step 2: Run (In NSCC)<a name="step-2-run-in-nscc"></a>
[Return to Table of Content](#table-of-content)

First, edit your job definition. See `jpb.psb` in detail! The examples and comments there covered everything!

Then, upload your container (the `.sif` file, if you use option 0 then ignore this one), dataset, code, and `job.psb` to NSCC. I always put `job.psb` and `main.py` in the same directory for convenience. Moreover, following NSCC's instruction, large files like dataset and container should be stored in `~/scratch` directory.

Finally, in the NSCC terminal, cd to the path storing `main.py`, and type
```
qsub job.pbs
```

to submit your job. If your `main.py` needs arguments, and you have already edited your `job.psb` accordingly (see `job.psb` for example), simply feed them with `-v` flag and comma separator as 
```
qsub -v bs=32,e=100 job.pbs
```

## Endnote: Useful commands in NSCC<a name="endnote-useful-commands-in-nscc"></a>
[Return to Table of Content](#table-of-content)

+ `qstat`: see the job numbers and status of your submitted jobs, but you don't know what variables you fed to the job.
+ `qstat -x -f`: see the summary of your recently submitted jobs, you can see the variables fed to the job if any.
+ `qdel <jobid>`: kill a job.
+ `qdel -W force <jobid>`: force kill a job, use this when a normal kill cannot work.

![](./images/vb9.png)
> :)