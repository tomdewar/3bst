# Basic SSH notes

## Create and secure the SSH keys folder (Linux or WSL2)
```bash
# if folder does not already exist, create it
mkdir -p ~/.ssh 

# restrict persmissions to .ssh folder
chmod 700 ~/.ssh
```

## Creating an SSH key pair (Linux or WSL2)
```bash
# generate a new SSH key pair
ssh-keygen -t ed25519 -C "comment helpful for identification" -f $HOME/.ssh/filename
```

The `-C` is for a comment. This gets included with the generated keys and should
be something which helps you identify this key pair as specific to:
1. you, 
2. your machine, and 
3. what it's for (e.g. Github or server name). 

If you Google the topic, most suggestions are to use your email address as the
comment. I prefer to use a short desciption which covers 1-3 above.

If you don't specify `-f` you will be prompted to enter a filename subsequently.
I recommend not accepting the default and instead pick a name which gives some
indication of what it's for (2-3 above). For example, if your machine's name is
"ABC" and the key is to connect to Github, I might set the filename to
`$HOME/.ssh/github_ABC`.

You will be prompted to create a passphrase. This is optional but recommended.
(Save your SSH passphrases into a Password Manager app.)

After this is complete, you will have two new files saved into your `~/.ssh`
directory. They will be named the same, after the filename you specified, but 
one will end `.pub`.

For example, `github_ABC` and `github_ABC.pub`.

The one ending .pub contains the public key. This is the one you will copy the
contents of to Github or copy the file to the server you will be connecting to.

The other is your private key. This should remain on your machine only and never
shared.

> [!Note]
> There is nothing special about the SSH key pair filenames. And the `.pub`
> extension is just a convention to make it clear which is the *pub*lic key.

Finally, you should restrict the permissions on these individual files:  
```bash
# private key should be read-write for you (owner), with no access for others
chmod 600 $HOME/.ssh/private_key_filename

# public key should be read-write for you, read-only for others
chmod 644 $HOME/.ssh/public_key_filename
```

## Creating an SSH key pair (Windows)
...is a pain. 

Use Windows Subsystem for Linux v2 (WSL2), if that's an option, and following
the instructions above.

If setting up VS Code on Windows to connect to Github, I suggest connecting and
cloning from VS Code itself. This will default to use HTTPS instead of SSH under 
the hood.

## Configuring SSH to use the right keys (Linux or WSL2)

If it doesn't already exist, create the SSH config file.
```bash
# create file
touch ~/.ssh/config

# restrict access to config file: make read and writable only by user
chmod 600 ~/.ssh/config
```

For connecting to Github, edit the file to include the following.
```
Host github
    User git
    HostName github.com
    IdentityFile ~/.ssh/private_key_filename
```

For connecting to a server, edit the file to include
```
Host friendly-server-name
    User [user name used to SSH into server]
    HostName [URL of server]
    IdentityFile ~/.ssh/private_key_filename
```

If a server has SSH accessible on a non-standard port, you can specify the port
in the config file too. Syntax `Port 2222`.

See [this blog post](https://linuxize.com/post/using-the-ssh-config-file/) for
more on using the SSH config file.

## Adding SSH private key to the ssh-agent (Linux or WSL2)
After you've created your SSH key-pair, streamline its use by using `ssh-agent`.

First, make sure ssh-agent is active by running:  
```bash
eval `ssh-agent`
```

Then add your private key by running:  
```bash
# this adds your private key with a timeout of four weeks;
# after the timeout expires you will need to re-add it;
# setting a timeout is optional but good security practice
ssh-add -t 4w ~/.ssh/private_key_filename
```
When prompted, enter your passphrase.

Once your private key has been added, you won't be prompted to re-enter your
pass-phrase each time, so long as the ssh-agent is running.

Because it's caching your crendentials, good security hygene is to:
- Set a timeout so it is not cached indefinitely
- Only have the ssh-agent running when you need it

Related useful commands:  
```bash
# see a list of all SSH keys which have been added to the ssh-agent
ssh-agent -l

# remove a specific SSH key from the ssh-agent
ssh-agent -d $HOME/.ssh/private_key_filename

# remove all SSH keys from the ssh-agent
ssh-agent -D

# stop the ssh-agent program
eval `ssh-agent -k`
```

## Putting your SSH public keys where they need to be

### To connect to GitHub
For use with GitHub you will need to copy the contents of your public key and
paste them into their web form as directed.

Follow [GitHub's instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).

### To connect to remote servers (Linux or WSL2)
To use an SSH key with a server, the server needs to have a copy of your public
key. You can use the helper program `ssh-copy-id` to do this.

```bash
# user = username on remote server
# server = URL of remote server
ssh-copy-id -i $HOME/.ssh/public_key_filename user@server
```

It will be saved into the `~/.ssh/authorized_keys` file on the remote server.