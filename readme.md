### Arch Linux *systemd-nspawn* based container management wrapper

## Dependencies

In order to use this script the packages **sudo** and  **arch-install-scripts** must be installed.
Additionally, for tab completion the package **bash-completion** is needed.

## Installation

Copy the wrapper script (*"container"*) in any folder in your PATH.

Then enable and start the machinectl systemd service:

    systemctl enable machines.target
    systemctl start machines.target

And that should cover the bare minimum configuration. However I strongly advice you to read sections "Base Packages", "Bash Tab Completion" and "Home Bootstrapping" for further customization.

## Usage

In general, the script is invoked as

    container (spawn|start|stop|remove|login) NAME [ADDITIONAL_ARGS...]

where NAME is a user-specified label for addressing the container. Following is a description of all commands:

### spawn

In order to create a new container called "foo", we call

    container spawn foo

which creates a new folder *"/var/lib/machines/foo"* and bootstraps a new arch linux base system into this folder using the *pacstrap* script included in **arch-install-scripts**.
If you want to automatically include some other packages you can specify them as additional arguments. For example:

    container spawn foo apache php

### start

Once a container "foo" has been created using the spawn command, the container (or "machine") can be started using

    container start foo

### stop

Stopping machines started using the start command is equally simple:

    container stop foo

Note that calling *poweroff* after logging into a container achieves the same goal.

### login

The whole point of having a container is to be able to log into the container and use the contained system without altering the host system.
This is achieved using the login command and the name of the container:

    container login foo

You can (initially) use the root user without a password. Of course you may change the password or create users after logging in.
The login shell is just like a normal shell - you can use *Ctrl+D* or the logout command to exit it.

### remove

Once a container is no longer required you may use the remove command followed by the container name to remove it:

    container remove foo

## Example Usage Scenario

Consider a scenario where you have developed a small python tool to accomplish great things. Now since you are better than average (even great?) you obviously wrote it in python 3, not
that deprecated version 2 the untalented majority uses for some reason. However the more people can use your software the better the world is, so you decide to make sure it runs with python 2.

Again obviously, issuing **pacman -S python2** is out of the question; your machine is neat and clean and you want it to stay that way. So containers to the rescue:

    container spawn test_python2 python2 python2-some-dependency-foo python2-some-dependency-bar
    container start test_python2

Now you have a fresh arch linux tainted with ugly old python2 packages.
You will need your script in there, so you either copy it manually to */var/lib/machines/test_python2/some/folder* or use

    machinectl copy-to /path/to/your/script/awesome.py /path/in/container

That's it. You forgot some package above, but we will fix that on the way:

    container login test_python2
    pacman -S python2-forgotten-dependency-baz
    cd /path/in/container
    python2 awesome.py # script runs just fine! nice...
    (ctrl-d)

Awesome, your script runs with python2 and you no longer need the container. So:

   container stop test_python2
   container remove test_python2

Done!

## Base Packages

As mentioned above when spawning containers any additional packages may be specified as additional arguments to the spawn command.
However when more packages are needed on every spawn one might want to define a base set of common packages to install. Examples include packages
like *bash_completion*, *sudo* or *vim*/*emacs*.

Due to this, there is a **BASEPKGS** variable in the beginning of the wrapper script which should be customized to ones own needs. Just set it to a space-delimited list of all desired base packages.


## Bash Tab Completion

If you want full tab completion in bash, first make sure that the **bash-completion** package is installed.
Then copy the *container_completion* script to some suitable location and make sure it gets source'd on login.

For my setup I copied the script into the *~/bash_completion.d* folder (I had to create that) and added the following lines to my *~/.bashrc*:

    if [ -d ~/.bash_completion.d ]; then
        for f in ~/.bash_completion.d/*; do
            source "$f"
        done
    fi

Note that for bash completion to work, the user must be able to see the contents of */var/lib/machines* (the script uses *ls /var/lib/machines*).

## Home Bootstrapping

Since I create containers a lot using this script I needed some way to boostrap the home folder of the root user (*/root*) using my common config files (e.g. my *.bashrc*, *.vimrc*, etc...).
This is implemented by having one folder (in my case *~/containers* containing/symlinking all files I want to have in roots home folder).

The folder can be customized by changing the **CONTAINER_CONFIG** variable in the beginning of the *container* script (in order to disable copying files just set it to a non-existing folder).
The wrapper script will copy **all** files in this directy (including hidden ones) non-recursively into */var/lib/machines/foo/root*. Note that symlinked files are copied from source.

## Networking

In order to have network access to/in the container some further steps are necessary. Read https://wiki.archlinux.org/index.php/Systemd-nspawn#Networking for some ways to achieve this (I use the method under "use host networking").
