# How to: git tricks

## Cherry pick a commit from another repository

To cherry pick a single commit from other existing repository:

```
$ git remote add other <URL or folder>  # add another repo as 'other'
$ git remote -v                         # check it
$ git fetch other                       # fetch commits of the 'other' repo
$ git cherry-pick <commit>              # cherry pick a desired commit from 'other' repo
$ git remote remove other               # remove 'other' repo if cherry-pick is complete
```

More commits can be picked up with:

```
$ git cherry-pick a^..b                 # cherry pick commits from 'a' to 'b' including 'a'
```

## Clone a private Github repository locally

If you try to clone a private Github repo without user name, then it will fail:

```
$ git clone https://github.com/<my_private_repo> <my_clone>
Cloning into '<my_clone>'...
remote: Repository not found.
fatal: repository 'https://github.com/<my_private_repo>/' not found
```

One way to solve this problem is specify <user_name> and provide public access token (PAT) as password (if you provide password, then it fail again):

```
$ git clone https://<user_name>@github.com/<my_private_repo>
Cloning into '<my_clone>'...
Password for 'https://<user_name>@github.com':
remote: Enumerating objects: 141, done.
remote: Counting objects: 100% (141/141), done.
remote: Compressing objects: 100% (85/85), done.
remote: Total 141 (delta 84), reused 102 (delta 52), pack-reused 0
Receiving objects: 100% (141/141), 32.35 KiB | 1.08 MiB/s, done.
Resolving deltas: 100% (84/84), done.
```

Once you have cloned the private repo, you can set up local credential, if you work on **multipe repos with different credentials**:

```
$ git config user.name "<my_name>"
$ git config user.email "<my_email>"
$ git config credential.helper 'store --file ~/.git-credentials.<my_clone>'
```

Links:
- [Clone a Github private repo](https://stackoverflow.com/questions/2505096/clone-a-private-repository-github)

## Use Github personal access token (PAT)

Store Github personal access token (PAT) locally to reduce the number of times you must type your username or password:

```
$ git config credential.helper store
$ git push
Username: <type your username>
Password: <insert your personal access token>   # your credential is stored in ~/.git-credentials
```

Next time when you push your commits, you won't be prompted for password :).
