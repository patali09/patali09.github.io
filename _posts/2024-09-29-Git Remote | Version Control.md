---
title: Git Remote Branching with Learn Git Branching
date: 2024-09-09 14:10:00 +0800
categories: [Git]
tags: [Git, Version Control]
---

# Git Remote with Learn Git Branching

In this section I am going to revise my git skill with https://learngitbranching.js.org/

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image.png)

# **Push & Pull -- Git Remotes!**

## Task *1: Clone Intro*

### Note

To download the remote repository from github or other alternative.

```bash
git clone http://url.of.repo
```

### Question

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%201.png)

### Solution

1. We just need to do clone
    
    ```bash
    git clone
    ```
    

## Task *2: Remote Branches*

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%202.png)

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%203.png)

### Solution:

1. Make a commit being on `o/main`
2. Checkout to `o/main`  again
3. Make a commit

```bash
git commit
git checkout o/main
git commit
```

## Task *3: Git Fetchin'*

### Note:

To fetch the changes from remote, we use `git fetch`

```bash
git fetch
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%204.png)

### Solution:

Simply do fetch.

```bash
git fetch
```

## Task *4: Git Pullin'*

### Note:

Git pull is the combine of git fetch and git merge. So, we can simply do both with single command.

```bash
git pull
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%205.png)

### Solution:

Simply pull repo.

```bash
git pull
```

## Task *5: Faking Teamwork*

### Note:

This level is just for simulating the updates in remote repo. But here I have solved it in more practical way. 

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%206.png)

### Solution:

1. Clone,  do couple of commit and push
    
    ```bash
    git clone
    git commit
    git commit
    git push
    ```
    
2. Make a new commit at `c1` merge `c1` at `c3`
    
    ```bash
    git checkout c1
    git commit
    git checkout c3
    git merge c4
    ```
    

## Task *6: Git Pushin'*

### Note:

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%207.png)

### Solution:

Simply make two commit and push the changes to remote

```bash
git commit
git commit
git push
```

## Task 7*:* Diverged Work

### Note:

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%208.png)

### Solution:

1. Clone the repo
    
    ```bash
    git clone
    ```
    
2. Make a commit in remote i,e fake teamwork with 1 commit
    
    ```bash
    git fakeTeamwork 1
    ```
    
3. Make a commit in locally
    
    ```bash
    git commit
    ```
    
4. Fetch the remote changes
    
    ```bash
    git fetch
    ```
    

6. Rebase local commit to remote latest commit

```bash
git rebase o/main
```

1. Push the changes
    
    ```bash
    git push
    ```
    


## Task 8*:* Remote Rejected!

### Note:

The remote rejected the push of commits directly to main because of the policy on main requiring pull requests to instead be used.

You meant to follow the process creating a branch then pushing that branch and doing a pull request, but you forgot and committed directly to main. Now you are stuck and cannot push your changes.

Create another branch called feature and push that to the remote. Also reset your main back to be in sync with the remote otherwise you may have issues next time you do a pull and someone else's commit conflicts with yours.

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2010.png)

### Solution:

1. Assign the current branch with another name i,e feature. 
    
    ```bash
    git branch feature
    ```
    
2. Push the commit to remote feature branch 
    
    ```bash
    git push origin feature
    ```
    
3. Checkout to `C1` and name it as `main` branch. So, main branch at local and remote both represent to same state.
    
    ```bash
    git checkout c1
    git branch -f main
    ```
    
4. Then, check  out to `main` and checkout to `feature`
    
    ```bash
    git checkout main
    git checkout feature
    ```
    

# **To Origin And Beyond -- Advanced Git Remotes!**

## Task *:1: Push Main!*

### Note:

When we are working on a base and someone remotely change that base. In such case we might get problem to pull or update the changes locally. For that we need to fetch and rebase properly. 

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2011.png)

### Solution:

1. Fetch the changes to o/main then rebase each of them properly. Point the main branch properly so remote and local both main can attain same state on push

```bash
git checkout side1
git rebase o/main
git checkout side2
git rebase side1
git checkout side3
git rebase side2
git branch -f main
git branch -f o/main
```

1. Then finally push the changes
    
    ```bash
    git push
    ```
    

## Task *2: Merging with remotes*

### Note:

We need to solve the previous task with merge than rebase.

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2012.png)

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2013.png)

### Solution:

Merge the commits in a way that they form the commit history like in goal. Make push being checkout to main.

```bash
git fetch
git checkout side1
git merge o/main
git merge side2
git merge side3
git branch -f main
git checkout c2
git branch -f side1
git checkout side1
git checkout main
git push
```

## Task *3: Remote Tracking*

### Note:

1. 

We will checkout a new branch named `foo` and set it to track `main` on the remote.

```bash
git checkout -b foo o/main; git pull
```

 we used the implied merge target of `o/main` to update the `foo` branch. Note how main doesn't get updated!!

1. 

Another way to set remote tracking on a branch is to simply use the `git branch -u` option. Running

`git branch -u o/main foo`

will set the `foo` branch to track `o/main`. If `foo` is currently checked out you can even leave it off:

`git branch -u o/main`

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2014.png)

### Solution:

1. Make a branch and set it to track `o/main` and make a commit
    
    ```bash
    git checkout -b side o/main
    git commit
    ```
    
2. Simply, pull the changes at origin, then  rebase and push
    
    ```bash
    git pull --rebase
    git push
    ```
    

## Task *4: Git push arguments*

### Note:

When we have multiple branches we need to push the branches by specifying its name if we are not in that specific branches.

```bash
git push origin branchname
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2015.png)

### Solution:

All the need to do is make push specifying the branch name as checkout is disable,

1. Push `main`
    
    ```bash
    git push origin main
    ```
    
2. Push `foo` 
    
    ```bash
    git push orign foo
    ```
    

## Task *5: Git push arguments*

### Note:

if we wanted the source and destination to be different? What if you wanted to push commits from the `foo` branch locally onto the `bar` branch on remote? 

In order to specify both the source and the destination of `<place>`, simply join the two together with a colon:

`git push origin <source>:<destination>`

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2016.png)

### Solution:

1. Push `main^` as `foo` and `foo` as `main`

```bash
git push origin main^:foo
git push origin foo:main
```

## Task *6: Fetch arguments*

### Note:

If `git fetch` receives no arguments, it just downloads all the commits from the remote onto all the remote branches.

We want to fetch the remote branch or commit and want plop locally in local branch.

```bash
git fetch origin <source>:<destination>
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2017.png)

### Solution:

1. Fetch `c3` as `foo` 
    
    ```bash
    git fetch origin c3:foo
    ```
    
2. Fetch `c6` as `main`
    
    ```bash
    git fetch origin c6:main
    ```
    
3. Checkout to `foo` and merge `main`
    
    ```bash
    git checkout foo
    git merge main
    ```
    

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2018.png)

## Task *7: Source of nothing/ deleting remote branch*

### Note:

We can delete remote branch by specifying nothing as source to a branch during push.

```bash
git push origin :foo
```

But,  fetching "nothing" to a place locally actually makes a new branch.

```bash
git fetch origin :bar
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2019.png)

### Solution:

1. Make a bar branch
    
    ```bash
    git fetch origin :bar
    ```
    
2. Delete foo branch
    
    ```bash
    git push origin :foo
    ```
    

## Task *8: Pull arguments*

### Note:

We can use pull to fetch and merge by assigning a branch name to a pulled commit.

```bash
git pull origin c3:branchname
```

### Question:

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2020.png)

![image.png](../images/Version%20Control/Git-Remote%2010f4d591802a80358a1ce5181e0a9be8/image%2021.png)

### Solution:

Being on main pull `c3` as `foo` branch and `c2` as `side` branch

```bash
git pull origin c3:foo
git pull origin c2:side
```