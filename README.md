[![Build Status](https://travis-ci.org/bo0rsh201/borssh.svg?branch=master)](https://travis-ci.org/bo0rsh201/borssh)
# borssh
Is a utility that helps you to transfer your dotfiles 

to any number of remote hosts transparently on ssh connection

Supported file types:
- .bash_profile
- .vimrc
- .inputrc
- any custom files you want

Let's see how it works

![intro](https://raw.githubusercontent.com/bo0rsh201/borssh/assets/record.gif)

# Algorithm

## Compile

Compiles all files from config into ~/.borssh/section_name.compiled

Calculates hash of all files and puts it into ~/.borssh/hash.compiled

Includes all compiled files into local:
- ~/.bash_profile
- ~/.vimrc
- ~/.inputrc

*Doesn't create duplicate include line*

## Connect
Reads local hash, does ssh command that checks remote ~/.borssh/hash.compiled file

If version is ok, opens bash session in this connection

If version is old/absent, runs rsync of total ~/.borssh directory 

and does same includes as on compile stage (but now on remote side)
 
Syncs new ~/.borssh/hash.compiled file to remote host and then connects

__The main thing is that config files are not changing so often and__ 

__in 99% cases borssh doesn't bring any overhead at all__

__(version is ok and we perform check and connect in single ssh command)__ 

# Getting started
You can get binaries for linux/darwin [here](https://github.com/bo0rsh201/borssh/releases/latest)

Or install it manually with
```
go install github.com/bo0rsh201/borssh
```
*you should have Go [installed](https://golang.org/doc/install)*

Bootstrapping dirs and config
```
mkdir -p -m755 ~/.borssh/bash/
echo 'PS1="Hey there \$"' >> ~/.borssh/bash/my_fancy_prompt
echo 'BashProfile = [ "bash/my_fancy_prompt" ]' >> ~/.borssh/config
```

"compile" command will include all files, listed in "BashProfile" config section
into your local ~/.bash_profile
```
borssh compile
``` 

Now you can sync them to remote host using 
```
borssh connect <put_hostname_here>
```

After new changes in config or bash/* files, you need to run compile command again

To make your borssh work on simple "ssh" command call, you need to add something like 
```
ssh () {
        if [ "$#" -eq 1 ]; then
                borssh connect $1
        else
                /usr/bin/ssh $@
        fi
}
```
to your local ~/.bash_profile

# Configuration
*all config paths are relative to $HOME/.borssh/ directory*
```
home dir: $HOME/.borssh
config file: $HOME/.borssh/config
```
Example:
```
BashProfile = [ "bash/my_bash_config_file", "bash/another_bash_config_file" ]
VimRc = [ "vim/my_vim_config" ]
InputRc = [ "vim/my_input_rc" ]
CustomFiles = [ "any_custom_file" ]
InitialSync = [ "hostname_mask" ]
```
## BashProfile,VimRc,InputRc

All files from these directives are compiled into single one 

and are automatically included to local and remote ~/.bash_profile ~/.vimrc ~/.inputrc respectively

so you don't need to do any extra actions on local/remote host

## CustomFiles

You can also transfer any custom files in ~/.borssh dir - just mention them in config section

## InitialSync

You may have some hosts that you visit very often and it can be annoying 

to wait for sync process after each configuration change (even if sync is fast)

so there is an opportunity to run initial sync on "compile" command for certain hosts.

You need to specify a list of [glob](https://en.wikipedia.org/wiki/Glob_(programming)) patterns 

and initial sync will be performed to all hosts from your "~/.ssh/known_hosts" that match

# Commands
```
borssh compile
```
compiles current version of config to:
- $HOME/.borssh/bash_profile.compiled - concat of all dotfiles from config
- $HOME/.borssh/hash.compiled md5 of previous file
and includes it into local .bash_profile

Also performs initial sync (if required)
```
borssh connect <host>
```
checks remote config version, performs sync and install if required and does ssh

# Flags
```
-q Quite mode (suppress all output)
```

# Supported OS
Supported OS
- Darwin (OS X)
- Linux