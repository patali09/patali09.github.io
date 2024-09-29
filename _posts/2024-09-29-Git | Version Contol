---
title: Git Branching with Learn Git Branching
date: 2024-09-09 14:10:00 +0800
categories: [Git]
tags: [Git, Version Control]
---

# Git with Learn Git Branching

In this section I am going to revise my git skill with https://learngitbranching.js.org/

![image](https://github.com/user-attachments/assets/51202671-1227-4b2d-ad6a-8abadbf685dd)

# 1: Introduction to Git commit

## Task 1

### Note

We can commit the changes in git with following command.

```jsx
git commit -m "commit remark"
```

### Challenge

In the given challenge we need to make two commit.

![Goal of task](https://github.com/user-attachments/assets/9992b8d8-3796-48c3-9a1f-6eb8cce25fc9)

Goal of task

### Solution

With commit command for twice we solved the challenge.

```jsx
git commit 
git commit
```

![Successfully solved Task 1](https://github.com/user-attachments/assets/e1542c91-abdc-4ef4-ae7f-29102efeac29)

Successfully solved Task 1

## Task 2

### Note

We can create the new branch with following command

```jsx
git branch "Branch Name"
```

We can change our branch with following command

```jsx
git checkout "Branch Name"
```

### Challenge

![Challenge ](https://github.com/user-attachments/assets/18319afe-059d-4d3c-90b4-91b3f4ca405d)

Challenge 

### Solution

Step 1: 

We made new branch with name `bugFix` .

```jsx
git branch bugFix
```

Step 2:

We move into our newly created branch `bugFix`.

```jsx
git checkout bugFix
```

![Task 2 solved](https://github.com/user-attachments/assets/b6f680c4-976c-4ae3-a56d-b824c2fdde11)

Task 2 solved

## Task 3

### Note

We can merge two branches in git with `git merge` . Say, we have two branches `main` and `branchtobemerged` . Now, before we merge `branchtobemerged` into `main`. We should make proper commit on both branch. Then we can merge simply with

```jsx
git merge branchtobemerged
```

### Challenge

![image.png](https://github.com/user-attachments/assets/61152cad-7735-4f90-b53b-29418504e070)

### Solution

Steps 1:

Made a new branch `bugFix` and get into that branch. Then we made a commit in that branch. 

```jsx
git branch bugFix
git checkout main
git commit
```

Step 2:

We come back to main and made a commit

```jsx
git checkout main
git commit
```

Step 3:

We merged `bugFix` into `main`.

```jsx
git merge bugFix
```

![Solved 3rd challenge of Introduction to Git commit](https://github.com/user-attachments/assets/3dd567a9-04a0-4d89-a014-9b8ba1be74f5)

Solved 3rd challenge of Introduction to Git commit

## Task 4

### Note

Rebase is the technique to align the two different branch into same commit line. For more clarity visit [https://www.youtube.com/watch?v=0chZFIZLR_0](https://www.youtube.com/watch?v=0chZFIZLR_0) .

If we have to rebase a `targetbranch` to `main` branch, we can first checkout or move to main and  run following command.

```jsx
git rebase targetbranch
```

### Challenge

![image.png](https://github.com/user-attachments/assets/671f86d7-0b84-45fb-9cd7-ffd503aa9d55)

### Solution

Step 1: Create a branch  `bugFix` ,  checkout to it and make a commit 

```jsx
git branch bugFix
git checkout bugFix
git commit
```

Step 2: Checkout `main` branch and make a commit

```jsx
git checkout main
git commit
```

Step 3: Checkout `bugFix` and rebase onto main

```jsx
git checkout bugFix
git rebase main
```

![image.png](https://github.com/user-attachments/assets/f408bd95-885d-42ab-b054-4a15eed460ad)

# 2: **Ramping Up**

## Task 1: *Detach yo' HEAD*

### Note:

https://www.youtube.com/watch?v=HvDjbAa9ZsY

To detach head from branch to commit, we can checkout to commit

```jsx
git checkout commithash
```

### Question

![image.png](https://github.com/user-attachments/assets/f02473da-2eb7-4387-ac72-45dfc33279cd)

### Solution

Since, we our head is at `bugFix`  and its commit hash is `C4` . So,  to solve it we need to checkout on its commit hash. We can do so with following command.

```jsx
git checkout C4
```

![image.png](https://github.com/user-attachments/assets/f45ccc85-4a52-4446-bd25-3854301d5edf)

## Task 2: *Relative Refs (^)*

### Note:

In Git, relative references, often denoted by the caret symbol (`^`), are used to specify commits relative to other commits. `HEAD^` refers to the commit before the current `HEAD` (i.e., the parent commit of the current commit). Instead of head we can also go with main or other branches. E.g. 

```jsx
git checkout main^
```

### Question

![image.png](https://github.com/user-attachments/assets/45d741c6-c0b7-44fb-80ab-f0c230002891)

### Solution

Here, we need to detach head to `C3` . For that, we can

```jsx
git checkout C3
```

![image.png](https://github.com/user-attachments/assets/45d741c6-c0b7-44fb-80ab-f0c230002891)

### Task 3:  *Relative Refs #2 (~)*

### Note

It might be tedious to type `^` several times, so Git also has the tilde (`~`) operator.

We can undo the changes in git even by erasing the history with 

```jsx
git branch -f branchname commit_hash
```

### Question

![image.png](https://github.com/user-attachments/assets/e2abdf43-af12-42f1-a642-d04a3326f46b)

### Solution

Step 1: point main brach to C6 commit

```jsx
git brach -f main C6
```

Step 2: Point bugFix branch to `C0`

```jsx
git branch -f bugFix C0
```

Step 3: Detach head on C1 

```jsx
git checkout C1
```

![image.png](https://github.com/user-attachments/assets/98972cb2-b571-405f-9d22-9b414aa2491a)

## Task 4: *Reversing Changes in Git*

### Note

There are two primary ways to undo changes in Git -- one is using `git reset` and the other is using `git revert`. The main difference between them is `git reset` actually remove the history of commit but `git revert` just make another commit. 

### Question

![image.png](https://github.com/user-attachments/assets/a9be052a-7546-4fb0-8c77-54d554157b65)

### Solution

Step 1: Reset c1

```jsx
git reset c1
```

Step 2: Checkout to pushed branch

```jsx
git checkout pushed
```

Step  3: Now, revert to pushed

```jsx
git revert pushed
```

# **3. Moving Work Around**

## *Task 1: Cherry-pick Intro*

### Note

When we have to make a series of commit by picking specific commit we can use git cherry-pick. 

```jsx
git cherry-pick c1 c2 c3
```

### Question

![image.png](https://github.com/user-attachments/assets/8551fa3d-ebaa-4de9-b396-4c7323ded58f)

### Solution

Its more straight forward. 

```jsx
git commit c3 c4 c7
```

![image.png](https://github.com/user-attachments/assets/5e20be33-9d5f-4381-8e92-b3809ac51c52)

## *Task 2: Interactive Rebase Intro*

### Note:

Git has UI based interactive rebase feature that allows us to alot from UI. We can use following command to access interactive rebase

```jsx
git rebase -i  c1
```

### Question

![image.png](https://github.com/user-attachments/assets/afa8ec60-834a-4508-be44-18406133f105)


### Solution

Simply, go  with git rebase with interactive flag i.e `-i` .

```jsx
git rebase -i c1
```

Then, simple omit `c2`, then arrange `c3` ,`c5` and `c4` then confirm. 

![final stage before confirm](https://github.com/user-attachments/assets/117efa2e-6c5e-4b5e-9b56-9c6cc96003af)

final stage before confirm

![image.png](https://github.com/user-attachments/assets/19dd0623-4fa7-4cd2-90e4-dc62be619ffa)

# **4. A Mixed Bag**

## Task *1: Grabbing Just 1 Commit*

### Note

Both  `git cherry-pick` and `git rebase -i`  can be used to play with specific commits.

### Question:

![image.png](https://github.com/user-attachments/assets/e38d8015-18ee-431c-aa02-dff08b31a2b6)


### Solution

Here, we can solve this challenge with `git cherry-pick` and `git rebase -i`  . Since, we are already clearly visualize the structure I preferred wot go with  `git cherry-pick`  .

Step 1: Checkout to main 

```jsx
git checkout main
```

Step 2: Cherry-pick to c4

```jsx
git cherry-pick c4
```

## Taks 2: *Juggling Commits*

### Note:

1. We can rebase the branches in more flexible way with interactive mode

```jsx
git rebase -i commithash
```

1. We can point a branch to a commit.

```jsx
git branch -f targetbranch destinationcommit
```

### Question:

![image.png](https://github.com/user-attachments/assets/2d412d51-65bd-4372-aa21-d9940abe34fc)
### Solution:

Step 1. Rebase the branch in interactive 

```jsx
git rebase -i c1 
```

![arrangement  of braches for rebase at step 1](https://github.com/user-attachments/assets/a07575bd-0805-46e3-ae80-1f98b245f6a8)

arrangement  of braches for rebase at step 1

Step 2: Rebase C2’ only

```jsx
git rebase c3'
```

![arrangement  of braches for rebase at step 2](https://github.com/user-attachments/assets/39b4b7b0-3855-46d6-920f-b4e28d25449a)

arrangement  of braches for rebase at step 2

Step 3: Rebase at c1

```jsx
git rebase -i c1
```

![arrangement  of braches for rebase at step 3](https://github.com/user-attachments/assets/bf1903c1-3d73-4e88-a0f6-55447e6fd335)

arrangement  of braches for rebase at step 3

Step 4: To achieve goal we need to point main to caption.

```jsx
git branch -f main caption
```

![finally solved](https://github.com/user-attachments/assets/c5ae8cd8-f397-4f44-808b-69a01d84c82f)

finally solved

## *Task 3: Juggling Commits #2*

### Note

1. We can point a branch to a commit.
    
    ```jsx
    git branch -f targetbranch destinationcommit
    ```
    
2. Cherry-pick is git flexible way to rebase
    
    ```jsx
    git cherry-pick targetcommit1 targetcommit2
    ```
    

### Question

![image.png](https://github.com/user-attachments/assets/ca3f9d33-2fe2-4cde-8251-01300c8f2f69)

### Solution

1. Checkout to main and cherry-pick to `c2`
    
    ```jsx
    git checkout main
    git cherry-pick c2
    ```
    
2. Again checkout to `c1` and cherry-pick to `c2’` and `c3` .
    
    ```jsx
    git checkout c1
    git cherry-pick c2' c3
    ```
    
3. Finally, point main to `c3’`
    
    ```jsx
    git branch -f main
    ```
    

![image.png](https://github.com/user-attachments/assets/ed8c0afc-a689-4e76-9dec-2f70c6964f3a)

### Alternative solution

```jsx
git checkout main
git cherry-pick C2
git commit --amend
git cherry-pick C3
```

## *Task 4: Git Tags*

### Note:

We can tag a special commit.

```jsx
git commit version1 targetcommithash
```

### Question:

![image.png](https://github.com/user-attachments/assets/87e87b90-0879-48cc-bd7e-d644cefa4b7e)

### Solution:

1. Tag `c1` with `v0` and `c2` with `v1`
    
    ```jsx
    git tag v0 c1
    git tag v1 c2
    ```
    
2. Checkout to `c2`
    
    ```jsx
    git checkout c2
    ```
    

![image.png](https://github.com/user-attachments/assets/5aea8c9c-a298-4a0a-ab95-c60decdcd6f4)

## *Task 5: Git Describe*

### Note:

`git describe` is a command in Git that provides a human-readable name for a commit. It uses the most recent tag in your repository to create a description that includes the tag name, the number of commits since that tag, and the abbreviated commit hash. Its output is in `<tag>-<number_of_commits>-g<commit_hash>` format

```jsx
git describe commithash
v1.0-5-gabc1234
```

### Question:

![image.png](https://github.com/user-attachments/assets/d9fa39db-bb3f-4f61-8d06-aff23a4db90f)
### Solution:

1. Try git describe with any commit
    
    ```jsx
    git describe c1
    ```
    
2. Checkout to `bugFix` and make a commit
    
    ```jsx
    git checkout bugFix
    git commit
    ```
    

![image.png](https://github.com/user-attachments/assets/c215bc0b-7699-4efa-99e2-9d27d891c256)

# **5. Advanced Topics**

## Task *1: Rebasing over 9000 times*

### Note:

### Question:

![image.png](https://github.com/user-attachments/assets/59e29674-f1cc-4e33-b9dd-83fe4484037a)

### Solution:

1. Rebase `bugFix` branch to main then side to `bugFix` , then `another` to `side`. Then, point main to latest commit. 

```jsx
git rebase main bugFix
git rebase bugFix side
git rebase side another
git branch -f main another
```

![image.png](https://github.com/user-attachments/assets/36f2e09f-1056-4ecb-bf20-4be406deab88)

## Task *2: Multiple parents*

### Note:

We can combine the modifiers like `HEAD~^2~`

```bash
git checkout HEAD~^2~
```

### Question:

![image.png](https://github.com/user-attachments/assets/ac038351-5caf-4df3-bae9-48dae966aacb)

### Solution:

1. Checkout to `c2` 
    
    ```bash
    git checkout HEAD~^2~
    ```
    
2. Point bugWork branch to c2 commit
    
    ```bash
    git branch -f bugWork
    ```
    
3. Checkout to main to accomplish the level
    
    ```bash
    git checkout main
    ```
    

## Task 3*: Branch Spaghetti*

### Note:

### Question:

![image.png](https://github.com/user-attachments/assets/03bc9a87-80e2-4ab7-8aee-53da424fbba5)

### Solution:

1. Checkout to `c1` then cherry-pick `c4`,`c3` and `c2`
2. Again checkout to `c2`  then cherry-pick `c5`  `c4’` `c3’` `c2’`
    
    ```bash
    git checkout c1
    git cherry-pick c5 c4' c3' c2'
    ```
    
3. Then point respective commit with branch name.
    
    ```bash
    git branch -f two
    git checkout c2
    git branch -f three
    git checkout c2'
    git branch -f one
    ```
    

![image.png](https://github.com/user-attachments/assets/078149f9-ed1b-4773-92a2-8da9fd793142)

Finally, completed the challenges related to local branches.