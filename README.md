# Scripts
This is an archive of Bash scripts for personal use. If interested in using for your own purposes, please take note of the following:

#### Adding the scripts' directory to your PATH
If you want to be able to execute the scripts from any directory on your machine without specifying the exact filepath, you'll need to add the directory to your machine's PATH variable.

This can be done in your `.bashrc` or `.zshrc` file (depending on which shell your system uses by default- on MacOS, it will likely be ZSH) with the following line:

`export PATH=/path/to/scripts:$PATH`

This will append the path to your scripts to the beginning of your PATH, meaning that your operating system will check that directory first when searching for the command you entered into the terminal. Likewise, if you want this directory to be checked last, you can adjust that line like so:

`export PATH=$PATH:/path/to/scripts`

## clean
This is a script for removing files on my machine that I find bothersome. Take note that the `$CODE` variable will need to be defined in your shell configuration file (`~/.bashrc` or `~/.zshrc`). For myself, I have this variable defined as the path to the location of my programming work.

## push
This is a script for pushing a backup of my code to the University of Arkansas's Linux server for student use, Turing. As such, only those with access to that server will have the ability to use this script. A considerable amount of configuration will need to be done on your part if you're interested in using yourself.

#### Generation of RSA key pair
Because this script is intended to run automatically without user interaction, a way to log in to Turing without the use of password entry was needed. The solution to this was generating an RSA key pair, keeping the private key on the machine running the script and a copy of the public key in an `authorized_keys` file on Turing.

To generate a key pairing, execute the following commands while in the home directory in the terminal. Take note that a .ssh directory may already exist if you have logged on to Turing before, containing a `known_hosts` file:

```
mkdir .ssh
cd .ssh
ssh-keygen -t rsa
```

After completing these commands, you should now have 2 more files in the `.ssh` directory: `id_rsa` and `id_rsa.pub`. The file with the `.pub` extension is your public key- this is an important distinction as <b>you should never share the contents of your private key</b>.

You now need to send the public key to Turing. This can be done using the `scp` command:

`scp id_rsa.pub your_username@turing.csce.uark.edu:~/`

This sends a copy of the public key to your home directory on Turing. This isn't quite sufficient enough; you'll need to add the contents of this key to an `authorized_keys` file in a `.ssh` directory on Turing:

```
ssh your_username@turing.csce.uark.edu
mkdir .ssh
mv id_rsa.pub .ssh
cd .ssh
touch authorized_keys
cat id_rsa.pub >> authorized_keys
rm id_rsa.pub
```

This chain of commands will log you on to Turing, make a corresponding `.ssh` directory, move the copy of the public key to that directory, and append the key contents to a file used for authorized key pairings.

For security reasons, I recommend making your `.ssh` directory on Turing and your local machines only readable, writeable, and executable by the user by running `chmod` in the home directory:

`chmod 700 .ssh`

This should be sufficient if you replace all instances of `turing` in the `push` script with `your_username@turing.csce.uark.edu`. If not, you'll need to create a `config` file in the `.ssh` directory on your local machine and add the following:

```
Host turing
  HostName turing.csce.uark.edu
  IdentitiesOnly yes
  IdentityFile /your/home/directory/.ssh/id_rsa
  User your_username
```

#### Using `cron`
I have this script automated to run every day at 00:00 with `cron`. You can edit your current `cron` tasks by running the following command:

`crontab -e`

A new file should open in the `vim` editor. Press `i` to enter insert mode and add the following:

```
PATH=/path/to/scripts:$PATH
0 0 * * * push
```

Then press `esc` to exit insert mode and run `:wq` to save the file and exit `vim`. This script should now run every day at midnight.

If you wish to change how often the script runs, follow [this link](https://crontab.guru) to learn cron's scheduling format.
