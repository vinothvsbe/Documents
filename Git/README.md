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
git log --oneline #Will provide only summary of the log
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

Whenever we forgot to type git commit message and just we provide git commit then by default it opens up **Vim** editor. But to work on that is little complicated
By default it asks us to enter commit message. But it is not editable. To make it editable press '**i**'
Then write message and then press '**:wq**' to write and quit editor in command prompt. 
But we can chnage this editor to our default editor like ***Visual Studio Code***

To configure **Visual Studio Code** as a default editor, here is the command for that

``` bash
git config --global core.editor "code --wait" # To configure VS Code
git config core.editor notepad # ToConfigure notepad as editor
```
If you are facing any problem with error message such as <mark>code not found</mark>, its possibly because VS Code wouldn't have got added to path. So the right way is to navigate to VS Code => ⌘+⇧+P (To open command pallette) or Windows + Shift + P (In Windows) => Type "Code" and click "Add code to path"







