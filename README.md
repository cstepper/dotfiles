
# dotfiles

-----

## Overview

This is a collection of the dotfiles I built up over time. 

It should help to track the changes and to simplify the setup on different (windows) machines.

## Setup

It was quite a bumpy road to get this working, but here's my workflow for setup.

The most important steps are:

1. Create a [bare Git repository](https://mijingo.com/blog/what-is-a-bare-git-repository)
   in the directory where your dotfiles (*.abc*) live. 
   I put mine in here: *~/Documents/.dotfiles*.
2. Define an alias `dot` which you can use instead of the regular `git` when
   interacting with the configuration repository (i.e. the *.dotfiles* repo).
3. Set a flag to the repository to hide files you are not explicitly tracking yet. 
   When you type `dot status` and other commands later, 
   files you are not interested in tracking will not show up as `untracked`.
4. Add the alias definition to your `.bashrc`<sup>[1](#bashrc)</sup> 
   (file will be created if not existing yet).


```bash
git init --bare $HOME/Documents/.dotfiles
alias dot='git --git-dir=$HOME/Documents/.dotfiles/ --work-tree=$HOME/Documents'
dot config status.showUntrackedFiles no
echo "alias dot='git --git-dir=$HOME/Documents/.dotfiles/ --work-tree=$HOME/Documents'" >> $HOME/Documents/.bashrc
```

5. After you've completed this setup, all files and folders within your defined 
   top directory of the working tree (`--work-tree`) can be versioned with normal commands, 
   replacing `git` with your newly created `dot` alias, like:


```bash
dot status
# bash startup
dot add .bashrc
# git configuration
dot add .gitconfig
dot commit -m "initial commit: add .bashrc and .gitconfig to dotfiles repo"
# R startup
dot add .Rprofile
dot commit -m "add .Rprofile"
# RStudio configs
dot add .R/rstudio/keybindings # addins.json, editor_bindings.json, rstudiobindings.json
dot add .R/rstudio/themes # cs_Monokai.rstheme, cs_TomorrowNightBright.rstheme
# NB: create your own theme here: https://tmtheme-editor.herokuapp.com/#!/editor/theme/Monokai
dot add .R/snippets/r.snippets # Rstudio snippets
dot commit -m "add RStudio personal configs"
# add more dotfiles if you like
```

6. Finally, for putting the dotfiles to *GitHub*, go to your github account and 
   create a new (private) repository. Then push your repo:

```bash
dot remote add origin https://github.com/cstepper/dotfiles.git
dot push -u origin master
```


## Usage

### git configs

#### ssl

If for any reason you need to adjust the SSL settings, 
you can run something like this in your console:

``` 
git config --global http.sslBackend schannel
```

This adds the following to your config file.
```
[http]
	sslBackend = schannel
```

See [stackoverflow](https://stackoverflow.com/questions/16668508/how-do-i-configure-git-to-trust-certificates-from-the-windows-certificate-store) for details. 

#### proxy settings

You can set the proxy settings in your `.gitconfig` file like:

```
[http]
	proxy = http://<username>:<password>@<proxy-server-url>:<port>
```

However, I don't advice to put your plain language credential password into 
your config file. Instead, make use of the *Windows Creditial Manager*:

```bash
git config --global http.proxy http://<user>@<proxy-server-url>:<port>
git config --global https.proxy https://<user>@<proxy-server-url>:<port>
git config --global credential.helper wincred
```
Just to be clear, replace `<username>` with your current windows username, 
and `<proxy-server-url>:<port>` with the your proxy settings.

See [stackoverflow.com](https://stackoverflow.com/questions/22799825/using-git-on-windows-behind-an-http-proxy-without-storing-proxy-password-on-di) and [git-scm.com](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy) for more information.

#### Update

According to the recommendations at [Happy Git with R](https://happygitwithr.com/credential-caching.html#turn-on-the-credential-helper), 
the preferred setting for ```credential.helper``` is now ```manager```, which 
configures Git to use Git Credential Manager for Windows, 
which ships with Git for Windows. 
It may already be configured, but this command sets it explicitly:

```bash
git config --global credential.helper manager
```

### use symlinks

Due to weird Windows settings for the `HOME` environment variable, you might 
end up having __two__ `.gitconfig` (and `.bashrc`) files, one in 
```
C:/Users/user/Documents
```
and another one in 
```
C:/Users/user
```

You can manually sync the `.gitconfig` files in both directories. 
One better option would be to symlink the `.gitconfig` to both directories.

Therefore, run `CMD` as *Administrator* and set the symbolic link:

```cmd
mklink %USERPROFILE%\.gitconfig %USERPROFILE%\Documents\.gitconfig
mklink %USERPROFILE%\.basrc %USERPROFILE%\Documents\.bashrc
```

See [support.rstudio.com](https://support.rstudio.com/hc/en-us/community/posts/203180856-Where-does-RStudio-look-for-the-global-gitconfig-file-)
for details.

If you use SSH keys for push/pull actions, it's advisable to do the same.
For other credentials stored in any of your dotfiles directories (e.g. .aws for Amazon S3), 
I'd suggest to add them to your symlinks in order to ensure a smooth workflow.

```cmd
mklink /J %USERPROFILE%\.ssh %USERPROFILE%\Documents\.ssh
mklink /J %USERPROFILE%\.aws %USERPROFILE%\Documents\.aws
```

See https://happygitwithr.com/ssh-keys.html for details.


## Resources

* https://github.com/randy3k/dotfiles
* https://www.atlassian.com/git/tutorials/dotfiles



## Footnotes

<a name="bashrc">1</a>: `.bashrc` is a shell script that
   Bash runs whenever it is started interactively. 
   It initializes an interactive shell session. 
   You can put any command in that file that you could type at the command prompt. 
   Usually, you put commands here to set up the shell for use in your particular environment,
   or to customize things to your preferences. 
   A common thing to put in `.bashrc` are aliases that you want to always be available.
   Source: [stackexchange.com](https://unix.stackexchange.com/questions/129143/what-is-the-purpose-of-bashrc-and-how-does-it-work)





