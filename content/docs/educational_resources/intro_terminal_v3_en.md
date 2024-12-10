# Terminal introduction
## Intro
- To open a terminal in most Linux distributions you can use `Ctrl+Atl+T` (if it doesn't work, you can try with `Atl+Enter`, `Alt+T`, `Ctrl+T` or something like that or just look for it in the app launcher)
- What is between `<>` has to be substituted and the "<>" are removed.
    - Ex: `cd <directory>` ---> `cd Documents`
- Some commands can be given arguments that modify their behaviour. These can also be called parameters, flags, options...
- Arguments are usually one letter long (but can be whole words) and are written with a dash in front.
- If you want to give a command more than one argument, you can put them together.
    - E.g.: `ls -a -l -h` ---> `ls -alh` (as a general rule, the order of the parameters is not important `-alh` = `-hla`)
- Commands, arguments, file/directory names... are all case sensitive.
    - E.g.: "Documents" is not the same directory as "documents"
- When a command doesn't work **read the error message**. Most of the time it will tell you exactly what went wrong and how to fix it.
- If the terminal completely freezes at some point it's possible that you pressed Ctrl+S by mistake. To unfreeze simply press Ctrl+Q.

## File hierarchy
In Linux the file system has a tree structure and all files and directories are under the root directory, represented as `/` (the root of the tree). For example, the user's home directory is in the path `/home`, which means inside the directory "home" which is inside the directory "/" (root).

There are two types of paths, absolute and relative.

- Absolute paths always start at root and indicate how to reach a file or directory from there. They work independently of where you are in the file system at the moment. When a path starts with `/` it's **always** absolute and the system will start at the root of the file tree.
- Relative paths depend on the directory in which you are located in that moment, so they can't be used from any place. The system follows the "directions" of the path starting at the directory you are currently on.
  
Example:

- `/home/my_usr/Desktop/dir1/file.txt`
- `dir1/file.txt`

Both paths go to the same file, but for the second example you have to be located in the Desktop directory for it to work and it doesn't begin with `/` because its relative, and only absolute paths (like the first one) begin with `/`. It's advisable to use absolute paths for scripts so that they can be executed from any directory without giving error.

There's a series of "special directories" that can be used in paths:

- `..` : it indicates the previous/parent directory. If you move to `..` from `Desktop/dir`, you will end up in `Desktop`.
- `.` : it indicates your current directory. It's useful when you want to copy a file to where you are (copy file to `.`)
- `~` : it indicates your home directory (`/home/my_usr`). It's the directory in which you start when you open a terminal. The paths that use it are a bit different from relative and absolute paths until now. It is relative to the user (if you are logged in into another account it won't work) but inside the user directory, it could be said to be absolute, as it doesn't matter in which directory you are (`~/Desktop/dir1` works even if you are inside `/home/my_usr/Documents/dir2`)

For more information about the structure of the Linux file system you can read
<https://linuxhandbook.com/linux-directory-structure/>

## ls [LiSt]
Lists the files (and subdirectories) in a directory (same as looking at them in the graphical file manager).

The contents of a directory are printed in different colours, which normally are:

- **White:** files
- **Bold blue:** directories (folders)
- **Bold green:** executable files (if it's not green it doesn't necessarily mean that file is not meant to be executed, it just means you don't have the permissions to do it)
(If it's not coloured by default you can try to use the parameter `--color=auto`)

You can use paths to make it list the contents of a specific directory (`ls ..` | `ls Desktop/dir`). By default it uses your current directory (so it's the same as specifying `ls .`)

In Linux, files and directories with a name starting by `.` are considered hidden. This means they won't be shown by default when you execute `ls` (or in the graphical file manager). It's used, for example, for configuration files, to avoid having them clutter the home directory and make it easier to navigate the files that are actually interesting for the user.

- `ls -a`: shows [a]ll, including hidden files.
- `ls -l`: [l]ists information about the files (like size, last modified date, permissions...)
- `ls -lh`: shows the file size in [h]uman readable mode (transforms to KB, GB... instead of writing it in bytes).

## cd [Change Directory]
Allows to move between directories using absolute or relative paths (equivalent to clicking on a folder in the graphical file manager).

- `cd <directory>`
- You can concatenate more than one directory : `cd dir1/dir2/dir3` (where `dir2` is a subdirectory of `dir1` and `dir3` of `dir2`)
- To go to the previous/parent directory : `cd ..`
- `cd` on its own takes you to your home directory (same as `cd ~`)
- `cd -` : goes to the directory in which you previously were.
    - e.g.:  if you're in `~/Desktop/dir1` and make `cd ~/Documents/dir2`, executing `cd -` will take you back to dir1. If you execute it again, it will take you to `dir2`, and you can toggle between those two until you move to a third directory.
    - e.g.: if you were in `~/this/is/a/directory/with/a/very/long/path` and you accidentally execute `cd` and go back to `~` (home), then instead of writing everything again you can just do `cd -`

## mkdir [MaKe DIRectory]
Creates a directory.

- `mkdir <directory>`: creates "directory" as a subdirectory in your current path

- You can write the path in which you want the directory to be created, but all previous directories have to exist.
    - E.g.: If you create a directory inside Documents (being the absolute path `/home/my_usr/Documents`), it only works if `home`, `my_usr` and `Documents` currently exist. You can use relative or absolute paths:
        - `mkdir /home/my_usr/Documents/new_dir`
        - `mkdir Documents/new_dir`

- You can create more than one directory at once:
    - `mkdir dir1 dir2`: `dir1` and `dir2` are created in your current directory
    - `mkdir Documents/dir1 Desktop/dir2`: `dir1` is created inside `Documents` and `dir2` is created inside `Desktop`
- If you want to create `dir2` in `Desktop/dir1/dir2` but `dir1` doesn't exist, you can use the `-p` parameter to tell it to automatically create every directory needed:
    - `mkdir -p Desktop/dir1/dir2`: creates `dir1` and then creates `dir2` inside

## cat
Prints the contents of a text file to the terminal.

- `cat <file>`
- `cat -n <file>`: prints the file with numbered lines

<br><br>

---

**WARNING: THE NEXT THREE COMMANDS ARE USED TO MOVE, RENAME, COPY AND ERASE FILES. IT'S IMPORTANT TO TAKE INTO ACCOUNT THAT THE TERMINAL DOESN'T HAVE AN UNDO BUTTON AND THERE IS NO BIN FOLDER WHERE DELETED FILES ARE SENT. IF SOMETHING GETS ERASED, OVERWRITTEN OR LOST, THERE IS NO WAY OF GETTING IT BACK. IT'S NOT A MATTER OF NEVER USING THESE COMMANDS, BUT IT'S IMPORTANT TO KNOW WHAT YOU ARE DOING, SPECIALLY IF YOU ARE USING WILDCARDS (EXPLAINED LATER).**

---

## mv [MoVe]
Moves/renames files and directories.

#### Moving
You specify the file (or files) you want to move and the destination (which has to be common for all files).

- `mv <file/dir> <destination>`
- `mv test.txt ib`: moves the file `test.txt` to the directory `ib`
- `mv dir ib`: moves the directory `dir` to the directory `ib`
- `mv test.txt dir ib`: moves file `test.txt` and directory `dir` to `ib`

You can use the path modifiers:

- `mv file ..`: moves the file to the parent directory

**Warning: If the destination doesn't exists and there is only one file/directory to move, it will rename it (to the destination name). If there is more than one file/directory to move, it will just give an error.**

#### Renaming
You specify the file you want to rename and the new name.

- `mv <file/dir> <new_name>`
- `mv fle.txt file.txt`: renames file `fle.txt` to `file.txt`
- `mv dir dir1`: renames directory `dir` to `dir1`
- `mv dir/file dir/file1`: renames file to `file1` inside the `dir` directory

**Warning: if the file already exists, it will be overwritten. If it exists but it's a directory, it will instead move it to the directory.**

#### Combining both
You can combine the move and rename operations:

- `mv dir/file file1`: moves file to the current directory (`./file1`) and renames it to `file1`
- `mv file dir/file1`: moves `file` to the directory `dir` and renames it as `file1`
- `mv dir/dir1 dir2`: moves `dir1` to the current directory and renames it as `dir2`
- `mv dir1 dir/dir2`: moves `dir1` to the directory `dir` and renames it as `dir2`

If a file hasn't been moved to where you expected and you can't find it anymore, try taking a careful look at the command and see if you could have renamed the file by accident.

## cp [CoPy]
Copies files/directories (can automatically rename the copy)

- `cp <file> <destination>`
    - `cp test.txt ib`: copies `test.txt` to the `ib` directory
    - `cp test.txt ib/test_copy.txt`: copies `test.txt` to the `ib` directory and renames it as `test_copy.txt`
- If you want to copy a directory you need to use the `-r` (recursive) flag so that it recursively copies the contents of the directory as well (if you forget, it simply gives an error and reminds you to do it).

For this command as well you can use the special path specifiers.

- `cp dir/test.txt .`: copies `test.txt` from the `dir` to the current directory
- `cp file ..`: copies the file to the parent directory

## rm [ReMove]
Deletes files/directories.

There shouldn't be any problem with using it if you are careful with what you do (aka, don't just paste the first command you find on the Internet without knowing what it does) but, if you feel more comfortable, you can always create a command that moves the files to a Bin directory instead, and then you can use rm inside it (see aliases in [Other interesting concepts to learn about](#other-interesting-concepts-to-learn-about)).

- `rm <file>`: removes the file
- `rmdir <directory>`: removes a directory but **only** if it's empty (useful if you want to make sure all directories you are erasing have nothing left inside).
- `rm -r <directory>`: removes recursively (the directory and its contents). Just like with `cp`, if you forget to put the `-r` the terminal will give an error (directory isn't empty). If it is empty, it will tell you to use `rmdir` but you can also use `rm -r`.

## nano
Simple terminal text editor. Because it works on the terminal, mouse support isn't activated by default (but there is an option to use it though). This means you need to use the arrow keys to move the cursor.

- `nano <file>`: opens a file to edit it. If it doesn't exists it creates it.
- **Ctrl+O**: saves the changes (capital O, not zero) and asks you what name you want to use. If you don't want to save them to a new file, just hit enter.
- **Ctrl+X**: exists the editor (if you haven't saved it will prompt you to do it).
There's a cheat sheet in the base of the editor with other useful keybindings.

## VS Code
If you use VS Code as you editor, you can launch it form the terminal (which is useful when you are already in the directory you want to work on).

- `code`: opens VS Code
- `code <file>`: opens the file in VS Code
- `code <dir>`: opens the directory in VS Code (if you are currently in it you can use `code .`)

## file
In Linux, file extensions are only useful as a reference for the user, the system ignores them and they don't have to really correspond to the real type of the file or be there at all (they can be omitted). To check the type of a file you can use the file command.

- `file <file>`: indicates the type of the file

## sudo
In Linux there exists a special user called root. This superuser has the permissions to do almost anything in the computer and it would be the equivalent of an administrator role. A user can have the permissions to execute commands as root and there are two ways of doing it. The first one is to log in as the superuser, which is dangerous because the computer will execute whatever you ask for without thinking twice about it. The second option (recommended) is writing `sudo` in front of the command you need to execute as root. The rest of your commands will be executed with normal permissions.

- `apt install` ---> `sudo apt install`
- `rm file` ---> `sudo rm file`

## sudoedit
When you are going to edit a system file, instead of using `sudo` directory it's recommendable to use `sudoedit`.

- `sudo <text editor> <system file>` ----> `sudoedit <system file>`

The command opens the file using your default editor (you can change it by adding the line `export VISUAL=<editor>` at the end of your `.bashrc` file in your home)

The first reason to use `sudoedit` is that it's safer. The command makes a copy of the system file which the user can edit and, when you are finished making changes to it, it writes them to the original file. Using `sudo`, the file is opened by the superuser, who has permissions to execute whatever command or delete that file or any other (in nano you can't, but if you use other editors like vim, you can open terminals or execute commands from inside it and those would be executed with root permissions). With `sudoedit`, everything will be done as your user, so there are less probabilities that something goes terribly wrong.

The second and more practical reason is, as it's executed as your user and not as root, the command uses your editor configuration. Again, maybe if you use nano it doesn't make much of a difference, but if you have a more configurable or extensible editor you can use your configuration when editing the system files instead of having to stick with the root config (which will probably be the default one).

## Other commands

- `clear`: clears the terminal (erases everything written and brings the cursor back to the top)
- `exit`: exists the terminal
- `find <dir> -name <name>`: finds every file/directory inside `dir` that has the name `name`. It has to be the exact name (unless you use wildcards)
- `history`: shows the history of commands
- `pwd` [Print Working Directory]: prints to the terminal the absolute path to the directory you are currently in
- `less <file>`: same as `cat`, it prints the contents of a text file, but less allows you to scroll through them
- zoxide: alternative to `cd` that stores a list with your most visited directories and jumps directly to the first one that matches the string you give it (not installed by default)
    - `z Doc`: could jump to `~/Documents`
    - `z screen `: could jump to `~/Desktop/pictures/screenshots`
    - `z shots `: could also jump to `~/Desktop/pictures/screenshots`
- `grep`: allows to search inside of files or in the output of a command (for example, if ls has a big output and you want to see if there is some pdf, you can use `grep` to search the ".pdf" string inside the output of `ls`). It's recommended to learn about redirections beforehand, since they will be specially useful here (in [Other interesting concepts to learn about](#other-interesting-concepts-to-learn-about))
    - `grep <string> <file>`: prints all lines `file` that contain `string`
    - `grep -i <string> <file>`: the search is case insensitive
- `xkill`: if a window freezes and stops responding, you can execute `xkill` inside a terminal and click with the mouse on the offending window to kill it.

## Other concepts

- To copy and paste, highlight the text to copy and press the mouse wheel to paste it. You can also use Ctrl+Shift+C and Ctrl+Shift+V.
- To interrupt a command : Ctrl+C
- Up arrow to see previous commands (goes through the command history). It allows you to execute them again or edit them.
- Tab autocompletes commands and file names. If there is more than one option, pressing it twice shows all possibilities (pressing it once seems to do nothing, that's when you know there are various options or none at all). Its use is recommended, not only to type quicker but to make sure the command/file exists and is correctly written (take into account that there isn't autocompletion of parameters for all commands).
    - `cd Do` [tab tab]
        > Documents/ \
          Downloads/ \
          Doc.txt
- Many commands have the option `-v` to make the output more [v]erbose (outputs more information about what the command is doing)
- Instead of using the command `clear`, it's possible to clean the screen using Ctrl+L. This has the added advantage that if you already have a command written you don't need to erase it in order to clear the screen.
- Instead of using `exit` you can exit the terminal by pressing Ctrl+D. For this you *do* need to have no input in the command line.

## The wildcard
In Linux, `*` represents the wildcard. It tells the terminal that it can substitute the asterisk by anything, including nothing. If you tell it to move to a directory every file with the name `*ing`, it will move the files "ing", "programming", "editing"... because they all end in "ing", whatever goes before doesn't matter (you have indicated it can be anything). However, the file "programming.txt" will not be moved. To include it you would have to indicate that it should also not care about what comes after "ing" (`*ing*`).

Another example would be if you have the files "ing.pdf", "ing.txt", "programming.txt" and "editing" and you just want to move the first two. For this you would use `ing*`, indicating that the file has to start with "ing" and whatever goes after doesn't matter.

This is very useful when you want to move al pdf files to a directory, for example:

- `mv *.pdf pdf-dir/` : moves every file that ends in ".pdf"
(We have to remember that in Linux you don't need to have the appropriate extension in every file, so this would move all files with a name ending in pdf, not necessarily all pdf files. But if the conventions are followed, which normally are, it should be the same in the end)

Correspondingly, if you have a bunch of files starting in the same way and want to erase them all, you can write the start string and add `*`.

- `rm erasable*`: will erase "erasable_edited.pdf", "erasable_original.pdf", "erasable.txt"...

**BE VERY CAREFUL WHEN USING WILDCARDS, SPECIALLY WITH RM.** If written on its own (`rm *`) it will erase everything in the directory (you are telling the system to erase everything with any name). This can be very useful but also very dangerous.

## Help

- `apropos <string>`: searches for `string` in the list of installed commands (has the command name and a brief description).
    - E.g.: `apropos editor`
        > ... \
          nano (1)  - Nano's ANOther text editor, inspired by Pico \
          ...
- `man <command>`: prints the command's [man]ual. Most commands (specially the most used ones) have a manual page. However, sometimes it can be difficult to understand, specially when you are starting. You press `Q` to exit the command page and yes, there is a manual for the manual (`man man`).
- `--help`: many commands accept this parameter and print a shorter version of the manual.
- `tldr` [Too Long Didn't Read] : instead of an extensive file about what the command does and every single possible option it has, `tldr` just gives a couple of examples of its most typical uses. The tldr pages are downloaded as you access them, so it's possible you need access to the Internet and they can take a bit to load if you haven't used it before. To avoid this you can do `tldr --update`(tldr is not installed by default).
- `cheat`: allows the use of cheat sheets. You can install the ones premade by the community (which is useful) but the most interesting thing about cheat is that it allows you to create your own entries. This way you can write your own cheat sheets as you go on learning new commands and parameters, to have everything handy and not have to look it up from scratch again (this command is not installed by default)
    - `cheat <command>`: prints the cheat sheet for `command`
    - `cheat -e <command>`: edits the cheat sheet for `command`. If it doesn't exist it is created.

My recommendation is: when you need to find a new command use `apropos` or the Internet ("terminal how to edit text file", "terminal merge pdfs"). When you know the command but need the parameters, use `tldr` or `cheat`. These tell you what to write to get the desired result, but they don't explain you the command. You can use `man` to search for the explanation for those parameters (you search by pressing `/` [ex: `/-l` + Enter] and move between the results with `n` and `N`). This way, you start to get comfortable with the manual without having to depend completely on it. If all fails, the Internet is your friend. I'd write down everything you learn on a text file, for future reference, or if it's a specific example for a command inside a cheat cheat sheet. Sometimes it takes a while to get the exact command you need and this saves you time the next time you have to use it.

## Useless comands everyone should know
They aren't installed by default and usually have a bunch of parameters and options you can learn about in the help pages (`man`, `tldr`...).

- `cowsay <text>`: prints a cow with a speech bubble that has `text` inside. With the parameter `-f` you can change the ascii drawing (my favourite is the stegosaurus with a hat)
- `sl`: animation of a train crossing your terminal. You can't stop it with Ctrl+C (they used "sl" to nag everyone that mistypes `ls` so not being able to stop it is on purpose). It has a couple of options, like people screaming for help or making the train fly.
- `cmatrix`: characters falling through your terminal screen
- `espeak`: text-to-speech
- `fortune`: prints a random fortune (like the ones from the cookies). You can create your on fortune files and use them instead of or with the default ones (it is also useful when you want a quick way of getting a random sentence from a set of them)
- `oneko`: renders a cat that follows your cursor through the screen.
- `xeyes`: a classic. Eyes that follow your cursor (if you use Wayland they won't work correctly)
- `toilet`/`figlet`: print asciiart text
- `rig`: creates a random identity (name-surname and address)
- `xpenguins` and `xsnow`: might not work everywhere (even inside X11) but should make a bunch of penguins appear and start moving through your screen (for the first one) and snow fall from the top of your screen and on top of your windows (for the second one).
- `lolcat`: prints multicoloured text. It's usually used with command redirection (e.g.: `ls | lolcat`).
- `neofetch`/`fastfetch` (and others): print an ascii drawing of your distro's logo (this can be changed) and your device specs (distro, desktop environment, RAM, CPU...). Neofetch is he most typical one but it has been discontinued and fastfetch is faster. Both can be personalized but neofetch has many pre-designed themes.
    - NeoCat (neofetch themes): <https://github.com/m3tozz/NeoCat>
    - Neofetch-themes: <https://github.com/Chick2D/neofetch-themes/tree/main>


## Other interesting concepts to learn about

- Package manager: the most practical way of installing packages in Linux. Depending on the distro you will use one manager or another but the important thing is to learn the commands to update/upgrade the system and install, remove and search packages.
- Redirection: allows the concatenation of commands, redirecting the output of one to the input of the next one or to a file.
  <https://www.freecodecamp.org/news/linux-terminal-piping-and-redirection-guide/>
- Alias: you can create an alias for a command to give it a different name (so you can write `potato` instead of `sudo`) or to simplify certain combinations of parameters that you use a lot (`ll` instead of `ls -lh`).
  <https://www.freecodecamp.org/news/how-to-add-aliases-to-terminal-commands/>
  (in most distros you don't need to touch the `.bashrc`. If you create a `.bash_aliases` file in your home directory it detects it automatically. This allows you to have an alias-specific file, cleaner and more concise).
- Automation: terminal scripts allow to automate many things and cron is also very useful for this, allowing to execute commands at certain hours/days automatically.
    - Bash Scripting Tutorial:
      <https://ryanstutorials.net/bash-scripting-tutorial/>
    - Understanding Crontab in Linux with Examples:
      <https://linuxhandbook.com/crontab/>
- zsh: bash is the default shell for many distros but there are others. One of the better known is zsh, due to its ample functionality and configurability, like syntax highlighting in the command line. It's usually installed alongside ohmyzsh, which allows to easily configure it and install extensions.
    - ohmyzsh web page: <https://ohmyz.sh/>
    - ohmyzsh GitHub: <https://github.com/ohmyzsh/ohmyzsh>
- `hollywood`: some Hollywood-hacking magic (just keep pressing Ctrl+C to exit it)

## Other resources
Some useful resources to learn how to use the terminal and Linux in general. I haven't looked at them all and they aren't in any particular order.

#### Tools/documentation:

- Explain Shell: you write a command and it explains what each part does (not perfect but it can help)
  <https://explainshell.com/>

#### Courses

- Linux Journey: web page to learn the Linux basics
  <https://linuxjourney.com/>
- Beginner's Guide to the Bash Terminal (video):
  <https://www.youtube.com/watch?v=oxuRxtrO2Ag>
- The Linux Command Line (book): https://www.linuxcommand.org/
- Linux Terminal - Getting Started! (video series):
  <https://www.youtube.com/watch?v=b5NmtmNwMgU&list=PLW5y1tjAOzI2ZYTlMdGzCV8AJuoqW5lKB>
- Text adventure game using the terminal commands (video game):
  <https://web.mit.edu/mprat/Public/web/Terminus/Web/main.html>
- Learning Terminal in Linux (video series):
  <https://www.youtube.com/playlist?list=PLc7fktTRMBozYfi4zlDeH0IdLdGImeOnO>
- Linux Tutorial:
  <https://ryanstutorials.net/linuxtutorial/>
- The Unix Shell:
  <https://swcarpentry.github.io/shell-novice/>

#### Articles

- Command Line for Beginners - How to Use the Terminal Like a Pro:
  <https://www.freecodecamp.org/news/command-line-for-beginners/>
- How to Type Less and Work Faster in the Linux Terminal:
  <https://www.howtogeek.com/815219/how-to-work-faster-in-linux-terminal/>

#### Interesting blogs

- ItsFOSS: News and tutorials about the free software world in general:
  <https://itsfoss.com/>
- Linux Handbook: Linux courses and tutorials:
  <https://linuxhandbook.com/>
- FreeCodeCamp: articles and courses about computers science in general:
  <https://www.freecodecamp.org/>
- LinuxOPsys: linux tutorials and tips:
  <https://linuxopsys.com/>

#### YouTube channels

- Learn Linux TV:
  <https://www.youtube.com/@LearnLinuxTV/videos>
- Veronica Explains:
  <https://www.youtube.com/@VeronicaExplains/videos>
- The Linux Experiment (Linux news):
  <https://www.youtube.com/@TheLinuxEXP/videos>

