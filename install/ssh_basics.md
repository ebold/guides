# Password input is prompted on SSH access (despite of public key authentication)?

## Relevant on creation and usage of SSH keys for other users

Assume you (named __asterix__) want to log in to a remote host (named __forest__) as another user (named __obelix__).
In order to secure your access you have generated SSH key pairs for the user __obelix__ and
copied his public key to the remote host __forest__.

```
asterix@home$ ssh-keygen -f ~/.ssh/obelix                     # with passphrase 'gallier'
asterix@home$ ssh-copy-id -i ~/.ssh/obelix.pub obelix@forest  # you'll be authenticated by password at this time
```

Afterwards, you add his private key to ssh-agent

```
asterix@home$ ssh-add ~/.ssh/obelix   # you'll asked to enter the passphrase
asterix@home$ ssh-add -l              # check if the private key is added
```

Now you can log in to __forest__ as __obelix__. But SSH still prompts password for __obelix__!!!

```
asterix@home$ ssh obelix@forest
obelix@forest's password:              # bang!!! what is gone wrong?
```

## Solutions

Typical causes are file and folder permissions on a remote host for an user.

Please check the permission of relevant file and folders and change them, so that they are writeable **only** by the user:

- **/home/obelix** - typical permission is 700 or 755
- **/home/obelix/.ssh** - typical permission is 700 or 755
- **/home/obelix/.ssh/authorized_keys** - typical permission is 600

```
obelix@forest$ chmod 700 $HOME      # assume HOME=/home/obelix
obelix@forest$ chmod 700 $HOME/.ssh
```

## Source

Credit to this [post](https://unix.stackexchange.com/questions/407394/ssh-copy-id-succeeded-but-still-prompt-password-input).
