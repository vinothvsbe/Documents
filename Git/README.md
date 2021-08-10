# Git
All git commands and important concepts explained

###Configuring Git
To configure the name/email that git will associate with your work, run this command

``` bash
git config --global user.name "Vinoth Suresh" # To set user name
git config --global user.email "vinothvsbe@gmail.com"  # To set email
```
To check do we already have a name configured then we can use

``` bash
git config user.name # To get user name
git config user.email # To get user email
```

To get complete log of the current git repo
``` bash
git log
```
Get current status
``` bash
git status
```
To Create new repository
``` bash
git init
``` 
>If you want to remove local repository then we need to remove <mark>.git</mark> folder

To add files to staging 
``` bash
git add <filename1> <filename2>  # Provide space between files and folders
git add . # '.' will stage all the files together to staged
```
To commit changes from Staging to local repository

```bash
git commit -m "<Message>"
```

> Keep your commits atomic

