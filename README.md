# Setting up multiple git users on one machine

Let's say you have 3 github accounts (usera, userb, and userc) and you want to use them on a single machine. Here is how I achieve this.


## Step 1: Create Folder for each user

I typically create a "Workspace" folder to house all my repos, and then I create a separate folder for each user. For example:

```
~/Workspace
├── usera_repos
├── userb_repos
└── userc_repos
```

## Step 2: Setting up `~/.gitconfig`

Use `includeIf` and `gitdir` in `~/.gitconfig` to specify the user for each user folder. Here is an example:

```
[user]
    name = usera
    email = usera@itworksonmymachine.com
    signingkey = ABCDEF0123456789ABCDEF0123456789ABCDEF01

; ignore some configs that are not associated with multiple users setup

[includeIf "gitdir:userb_repos/"]
    path = ~/.gitconfig.d/userb.inc

[includeIf "gitdir:userc_repos/"]
    path = ~/.gitconfig.d/userc.inc
```

Assume that 'usera' is the default user, while 'userb' and 'userc' are the additional users. The `includeIf` block specifies the path to each user's folder and the corresponding to the user's config file.

Now create the user-specific config files, I put them in `~/.gitconfig.d/` folder. For example, the configuration for 'userb' in `userb.inc` might look like this:

```
[user]
    name = userb
    email = userb@itworksonmymachine.com
    signingkey = 1234567890ABCDEF1234567890ABCDEF12345678
[url "git@userb-github.com:userb/"]
    insteadOf = git@github.com:userb/
```

You can set up the `user.name`, `user.email`, and `user.signingkey` just as you would for the default user. The `url` block is used to rewrite the remote URL to use the correct SSH key. For instance, I use `userb-github.com` for 'userb' and place the original `github.com` URL in the `insteadOf` field.

Similarly to 'userb', the configuration for 'userc' in `userc.inc` might look like this:

```
[user]
    name = userc
    email = userc@itworksonmymachine.com
    signingkey = 9876543210FEDCBA9876543210FEDCBA98765432
[url "git@userc-github.com:userc/"]
    insteadOf = git@github.com:userc/
```

This setup allows 'userc' to also have a custom configuration, ensuring that the user.name, user.email, and user.signingkey are correctly set, and don't forget to use a different rewritten remote URL in the url block.

## Step 3: Setting up `~/.ssh/config`

I will skip the details of setting up SSH keys for each user, assuming that you've already done so. Since 'usera' is the defualt user, you can use the default SSH key(e.g., `~/.ssh/id_ed25519`) for it. For 'userb' and 'userc', let's say you've already created `id_ed25519_userb` and `id_ed25519_userc` for them, you now need to specify the correct key in `~/.ssh/config`. Here is an example:

```
# for usera, the default user
Host github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519

# for userb
Host userb-github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519_userb

# for userc
Host userc-github.com
    HostName github.com
    User git
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519_userc
```

Ensure the `Host` for 'userb' and 'userc' matches the rewritten remote URLs in the `~/.gitconfig.d/userb.inc` and `~/.gitconfig.d/userc.inc` files. In the `IdentityFile` field, specify the correct SSH key for each user.

And that's it! You can now use multiple git users on a single machine. Each user has their own folder, git config, and SSH key.

## Step 4: Using the users

When you clone a repo from 'usera', there's no need for anything special. Simply do:

```bash
cd ~/Workspace/usera_repos
git clone git@github.com:usera/usera-repo.git
```

However, when cloning a repo from 'userb' or 'userc', you must specify the rewritten remote URL and ensure that you are in the correct folder. For example:

```bash
cd ~/Workspace/userb_repos
git clone git@userb-github.com:userb/userb-repo.git
```
