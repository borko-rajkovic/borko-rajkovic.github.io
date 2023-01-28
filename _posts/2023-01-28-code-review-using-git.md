---
layout: post
title: Code review using Git
author: borko
date: 2023-01-28 17:48 +0100
description: Working on the Code Review can be challenging for the complex PRs. Sometimes it requires you to change the code in place while doing the Code Review. In this post, I am going to show you how I do the Code Review using Git.
categories:
- git
tags:
- git
- code-review
image: "/assets/img/posts/2023-01-28-code-review-using-git/thumbnail.png"
---
A common practice in modern software development companies is to have a **Code Review** process in their development teams, which I believe is a **must** for a clean code base.

In this post, I am going to show you how I do the **Code Review** using Git.

## Simple vs complex PRs

**PR**s can be divided into **simple** or **complex**. With practice, over time you can get a feeling of what makes a PR simple and what makes it complex, and it can differ from person to person. Anyway, each one of us considers some **PR** as simple or complex based on their criteria, or for the lack of it, just a gut feeling 😉.

### Dealing with simple PRs

For simple PRs, I like to use the help of the platform where the code is hosted, be it **GitLab**, **GitHub**, **BitBucket**, **Azure Repos**, you name it. All of them are pretty good at offering basic functionality, such as:

- viewing the changed files
- making comments
- marking something as already reviewed 
- offer suggestions the other developer can merge directly in their code (not all platforms, though)

### Dealing with the complex PRs

Now, when it comes to complex PRs, what I am missing when using the above-mentioned platforms is to actually change the code itself, run the tests for my changes and commit these changes into a separate commit.

Let's formalize what is it exactly we want to achieve here:

- have the changes from the **PR** in Git's `Working Directory`
- after reviewing the piece of code, commit original changes alongside changes from **Code Review** (if any)
- in the end, we'll have just one additional commit with **Code Review** changes

## Pre-requisites

- Git
- VS Code (optional)
- [Git Lens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) extension for VS Code (optional)

> All of the steps I will show can be made in any other IDE or even using the command line.
{:.prompt-info}

## My recipe for Code Review using Git

1. revert all the commits from the PR starting from the latest
2. reset to the last commit from the feature branch
3. commit all changes from the `Working directory` to the "Code review" commit
4. revert the "Code review" commit
5. reset to commit "Code review"
6. do the actual review and `commit amend` original changes with any modifications from the **Code review**

Don't worry if this looks too abstract, we'll cover it in the **Example** section.

<!-- ## Steps breakdown

Let's go step-by-step and explain what is happening.

### Step 1 - revert all the commits from the PR starting from the latest

If you have more than one commit in PR, revert the commits starting from the last one and go all the way to the commit from which the branch was created (and do not revert that commit).

The meaning of this step is to bring the code to the same state as before any changes were made. This will be our base for committing the PR changes.

### Step 2 - reset to the last commit from the feature branch

After step 1 we'll have as many "revert" commits as there were original commits on the feature branch after branching. So, if a PR had 3 commits, we'll have 3 "revert" commits.

Once we reset to the last commit from the feature branch, we'll get the changes that are reverting all the commits made on a feature branch (3 in our example) in Git's `Working directory`.

### Step 3 - commit all changes from the `Working directory` to the "Code review" commit

Now we just commit all those "revert" changes to a commit named "Code review". At this point, it looks like this commit is just reverting everything a PR wanted to change.

In the next 2 steps, we are going to fix this.

### Step 4 - revert the "Code review" commit

Again reverting?! Yes! So now we have all the original changes from the PR back again.

The point of this step is to prepare the original changes for the actual Code Review.

### Step 5 - reset to commit "Code review"

And the final preparation step is to reset to the "Code review" commit.

Hope it's clear now that if you commit amend all the changes to a "Code review" commit it will actually be empty!

But if you did any modifications along the way, the "Code review" commit will contain only those modifications you did in a Code Review process.

And that is exactly what we wanted in the first place!

### Step 6 - do the actual Code Review

Now you can go through the code and `commit amend` changes if you approve them, or modify if needed and then `commit amend` original changes with your modifications.

In the end, you will be left with just one additional commit on the PR that contains all the changes you made during the Code Review process. -->

## Example

Let's see how to use this flow on a sample project.

> You can clone this repo and try it out yourself [here](https://github.com/borko-rajkovic/git-code-review)
{:.prompt-info}

### Project setup

To keep things simple, I will use a project with two branches:

- **main** - the branch to which we are going to merge PR
- **fix-prime** - the branch on which changes are made and are ready for a review

On the **main** branch, we have two commits - the **Initial commit** with README.md and **Prime** where we added the `prime.js` file.

![main branch](/assets/img/posts/2023-01-28-code-review-using-git/main_branch.png){: .shadow style="max-width: 90%" }
_main branch_

On the other hand, the **fix-prime** branch has 3 commits in front of the **main** branch that are used to make fixes to the `prime.js`

![fix-prime branch](/assets/img/posts/2023-01-28-code-review-using-git/fix_prime_branch.gif){: .shadow style="max-width: 90%" }
_fix-prime branch_

### Step 1 - revert all the commits from the PR

We need to revert all the commits starting from the last commit made in the branch until we get to a commit from the **main** branch.

![step 1 - revert all the commits from the PR](/assets/img/posts/2023-01-28-code-review-using-git/step_1_revert_all_commits.gif){: .shadow style="max-width: 90%" }
_step 1 - revert all the commits from the PR_

Using git from the terminal you can do the same by running the following command for every commit:

```sh
git revert --no-edit <commit-SHA>
```

### Step 2 - reset to the last commit

Once we have all the commits reverted, we'll reset to the last commit from the **main** branch.

![step 2 - reset to the last commit from the fix-prime branch](/assets/img/posts/2023-01-28-code-review-using-git/step_2_reset_to_last_commit.gif){: .shadow style="max-width: 90%" }
_step 2 - reset to the last commit from the fix-prime branch_

```sh
git reset <commit-SHA>
```

### Step 3 - create a new commit called "Code review"

Now we'll create a new commit with all changes in the `Working directory` and we'll name it the **Code review**.

![step 3 - create the "Code review" commit](/assets/img/posts/2023-01-28-code-review-using-git/step_3_create_code_review_commit.gif){: .shadow style="max-width: 90%" }
_step 3 - create the "Code review" commit_

```sh
git add .
git commit -m "Code review"
```

### Step 4 - revert that new commit

After we have all the reverted changes from commits in the **feature** branch merged into one commit (**Code review** branch), we need to revert that branch as well, and you'll see in a moment why.

![step 4 - revert the "Code review" commit](/assets/img/posts/2023-01-28-code-review-using-git/step_4_revert_code_review_commit.gif){: .shadow style="max-width: 90%" }
_step 4 - revert the "Code review" commit_

```sh
git revert --no-edit <commit-SHA>
```

### Step 5 - reset to commit "Code review"

Now reset to the **Code review** branch, essentially leaving changes of the commits from the **feature** branch to Git's `Working directory`.

![step 5 - reset to the "Code review" commit](/assets/img/posts/2023-01-28-code-review-using-git/step_5_reset_to_code_review_commit.gif){: .shadow style="max-width: 90%" }
_step 5 - reset to the "Code review" commit_

```sh
git reset <commit-SHA>
```

Now that we prepared our code base for a code review, we can start the code review process.

### Step 6a - commit original changes as is

Now we can go with the code review process. In this example, we'll show how to commit original changes without any modifications to them.

![step 6a - commit original changes to the "Code review" commit](/assets/img/posts/2023-01-28-code-review-using-git/step_6_commit_changes_as_is.gif){: .shadow style="max-width: 90%" }
_step 6a - commit original changes to the "Code review" commit_

```sh
git commit --amend --no-edit
```

### Step 6b - commit original changes with some modifications from the code review

You'll see here how easy it is to make modifications to the original changes and commit them.

We spotted that the condition is wrong, it should be `<=` instead of `<`. Let's see how to do this modification.

![step 6b - commit modifications to the PR in the "Code review" commit](/assets/img/posts/2023-01-28-code-review-using-git/step_7_commit_modifications_to_the_pr_in_code_review_commit.gif){: .shadow style="max-width: 90%" }
_step 6b - commit modifications to the PR in the "Code review" commit_

```sh
git commit --amend --no-edit
```

### The result

And this is what we get at the end - one commit with all modifications we made in the code review process.

![The result of the Code Review](/assets/img/posts/2023-01-28-code-review-using-git/step_8_the_result_of_code_review.gif){: .shadow style="max-width: 90%" }
_The result of the Code Review_

> You may wonder what happens if you did not make any modifications? In such case, Git will warn you that doing `commit amend` will make your commit empty. It's up to you do you prefer to have an empty **Code review** commit, or you prefer to delete it.
> If you choose to do the former, then you should commit with: `git commit amend --no-edit --allow-empty`
{:.prompt-info}
