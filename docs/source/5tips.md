
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
