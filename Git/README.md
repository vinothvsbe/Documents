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
git commit -m "<Message>" # To commit added items
git commit -a -m "<Message>" # -a will add all the un added files and then it will commit with message. 
#Dont have to call of git add seperately
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

### Amend
For some reason you forgot to commit a file or you have written wrong commit message. Anything to do with exisitng commit, then you following commands will help

``` bash
git commit -m "Some Commit"
git add <missedFile> # in case if you have missed
git commit --amend # This open up the default editor to 
#change comment in case if you want to change the comment
```
### .gitignore
If we have to ignore following patterns can be used
<mark>\*.txt</mark> -- To ignore text files
<mark>FolderName/</mark> -- To ignore folders use '/' after
<mark>[Ff]olderName/</mark> -- To ignore folder name starts with F capital or small case.
<mark>**/[Pp]ackages/*</mark> -- Wherever Packages folder is there can be ignored and all items inside that will also be ignored

<mark>!**/[Pp]ackages/build/</mark> -- Wherever Packages fodler is there that can be ignored except the Packages folder which contains build folder should not be (!) ignored

## Branching

#### HEAD
HEAD is nothing to pointer for the current branch

Wherever the current branch is HEAD will move to that branch. If branch needs to be changed then HEAD need to be switchted to that branch

Viewing Branch
``` bash
git branch # Shows current branch
```
Whichever has \* is considered to be active branch

To create a new branch
``` bash
git branch <branch-name> #Make sure that it shouldnt include spaces
```
#### Switch
Switching between different branches will be done with the help of 

``` bash
git switch <branch-name> # Switch to existing branch
git switch -c <branch-name> # Create the branch and switch to it
```
switch  <mark>-c</mark> is nothing but create

#### Checkout
Checkout is just similar to Switch but with lot more additional features

``` bash
git checkout <branch-name> # Switch to existing branch
git checkout -b <branch-name> # Create the branch and switch to it
```

>*Whenever we switch the branches if the file which is present accross has conflict then it will not allow you to switch without committing the changes. Whereas if the file is completely new and that file is not present in any branch then you can take that file across any branches without committing.*

#### Deleting and Renaming Branches
To Delete the branch following command will help

``` bash
git branch -d <branch-name> # This will delete the branch if it is new
git branch -D <branch-name> # This will force delete the branch though it is used. 
#But thing to remember is if you want to delete the brancht then HEAD should not be in that branch
git branch -d -f <branch-name> # Just similar to git branch -D <branch-name> . To Force delete
```
To rename the branch
``` bash
git branch -m -f <branch-name> # -m stands for move/rename. 
#The concept is we should be in that current branch to move/rename it. 
#So first switch to the right branch and then execute this command 
```
