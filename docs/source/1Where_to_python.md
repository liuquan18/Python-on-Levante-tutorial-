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
