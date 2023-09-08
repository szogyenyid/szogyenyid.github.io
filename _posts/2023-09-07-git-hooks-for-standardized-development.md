---
layout: post
title: "Using git hooks for standardized development and efficient collaboration"
date: 2023-09-07 21:30:00 +0200
categories:
    - development
    - meta
tags:
    - development
    - git
    - git-hooks
    - scripting
    - automation
    - productivity
description: "Unearth the capabilities of Git hooks, the often-overlooked automation feature in Git. This article delves into the world of Git hooks, explaining how they can be harnessed to enforce coding standards, maintain commit message consistency, and optimize your development process. Explore practical examples of Git hooks' applications and elevate your collaborative coding experience."
last_modified_at: 2023-09-08 10:38:00 +0200
author: Daniel Szogyenyi
readtime: 15
---

## Introduction

Git, the popular version control system, empowers developers to collaborate seamlessly on software projects. While its fundamental features are well-known, Git offers a hidden gem that can significantly enhance your development workflow: Git hooks. Git hooks are scripts that can be triggered at specific points in Git's workflow, allowing you to automate and enforce various aspects of your development process.

In this article, we'll explore the world of Git hooks and how they can help you maintain code quality, ensure consistent commit messages, enforce branch naming conventions, and many other things. We'll delve into  four important Git hooks: pre-commit, prepare-commit-msg, commit-msg, and pre-push, demonstrating how each of them can be harnessed to streamline your development process.

## Why should I read this article?

At the end of this guide, you will

- understand what git hooks are,
- how to create or modify them,
- have git hooks to check your branch names, check your PHP coding style, automatically add Jira issue ID to commits, and check if your commit message is a Conventional Commit.

## What are Git hooks?

Before we dive into the individual Git hooks, let's establish a clear understanding of what Git hooks are and why they matter.

> Hooks are programs you can place in a hooks directory to trigger actions at certain points in git’s execution. Hooks that don’t have the executable bit set are ignored. [[1]](https://git-scm.com/docs/githooks)

### Client-side vs. Server-side hooks

Git hooks can be categorized into two main types: client-side and server-side hooks. Client-side hooks run on the local repository of each developer, while server-side hooks execute on the remote Git server. In this article, I'll focus exclusively on client-side hooks, which empower individual developers to enforce code quality, commit message consistency, and other development process aspects within their own local repositories.

![The typical flow of git usage, and the trigger points of hooks.](https://szogyenyid.github.io/assets/git-hook-order.jpg)
*The typical flow of git usage, and the trigger points of hooks. [[2]](https://devopedia.org/git-hooks)*

In this article, I will show some practical examples of four of the five local hooks: pre-commit, prepare-commit-msg, commit-msg, and pre-push.

### Locating and editing Git hooks
Git hooks are stored as executable scripts within the `.git/hooks` directory of your Git repository. When you initialize a new Git repository, a set of sample hook files are automatically created in this directory. To customize a hook, simply locate the corresponding script in the `.git/hooks` directory, remove the `.sample` extension from its filename, and edit it using your preferred text editor. These scripts can be written in various scripting/programming languages (e.g., shell, Python, Ruby, even PHP), allowing you to tailor them to your specific needs. Once edited, Git will automatically execute the custom hook at the specified trigger point in your Git workflow.

For example, if you want to customize the pre-commit hook, you would find it in `.git/hooks` as `pre-commit.sample`, rename it to `pre-commit`, and then edit the script to include your desired logic.

Remember that Git hooks are specific to each Git repository, so you can customize them differently for different projects based on your requirements.

Note: the commands of the following guide are tested on MacOS and Linux. They should work on Windows using Git Bash, but I cannot guarantee that you do not have to modify anything.
{: .info-blue }

### A note on regular expressions
To fully grasp how the script works, it's helpful to have some familiarity with regular expressions (regex). Regex is a powerful pattern-matching language used in the script to define the branch name pattern. However, if you're not well-versed in regex, don't worry—there are comments within the script to guide you through its usage. Additionally, you can find valuable resources and tutorials online to learn more about regular expressions. A great starting point is the documentation and tutorials available on websites like https://regex101.com/ or https://regexr.com/, and regex visualizers like https://regexper.com/. With a bit of practice, you'll be able to customize the script to match any specific branch naming requirements effectively.

## 1) Branch name verification
Maintaining a consistent and well-defined branch naming convention is crucial for organized and efficient version control. In this section, we'll explore how to implement branch name verification using a pre-push Git hook. This hook will ensure that all branch names adhere to a specific regex pattern, helping to enforce naming conventions across your Git repository.

### Why use pre-push for branch name verification

Utilizing the pre-push hook for branch name verification offers key advantages. It proactively enforces branch naming conventions before pushing, maintaining a tidy repository by preventing poorly named branches being pushed to the remote. This preemptive approach saves developers from dealing with rejected pushes after their work is complete, streamlining the development process. In essence, the pre-push hook acts as an efficient gatekeeper, allowing only well-named branches into the shared codebase.

### Setting up the pre-push hook

To enforce branch name verification with a pre-push hook, follow these steps:

1. Navigate to the .git/hooks directory: inside your Git repository, navigate to the `.git/hooks` directory.
2. Create the pre-push hook script: create a new file named `pre-push` (without any file extension) within the .git/hooks directory.
3. Edit the pre-push hook script: open the `pre-push` script in a text editor and add the following code:

```shell
#!/bin/sh

# Verify if branch name is standard:
#   - (optional) type/
#   - PRJT-number
#   - (optional) _dash-separated-description

local_branch="$(git rev-parse --abbrev-ref HEAD)"
valid_branch_regex="^(feature/|bugfix/|release/|hotfix/)?(PRJT-[0-9]+)(_[a-z-]+)?$"
if [[ ! $local_branch =~ $valid_branch_regex ]]
then
    echo "Something is wrong with your branch name."
    echo "It should match the regex: $valid_branch_regex"
    echo "But it's value is: $local_branch"
    exit 1
fi
exit 0
```
4. Make the script executable: ensure the pre-push script is executable. You can do this by running:

```shell
chmod +x .git/hooks/pre-push
```

### How the pre-push hook works

Whenever you attempt to push changes to the remote repository, the pre-push hook will be triggered. It extracts the branch name from the local reference and checks if it matches the specified regex pattern. If the branch name doesn't conform to the pattern, the hook will prevent the push and display an error message.

By using this pre-push hook, you can ensure that all branches created and pushed to your repository adhere to the defined naming conventions, helping maintain a structured and organized Git workflow.

## 2) Pre-commit hooks for code quality

One of the most common use cases for Git hooks is to enforce code quality standards before commits are made. In this section, we'll explore how to set up a pre-commit hook to run PHPCodeSniffer, a powerful tool for ensuring consistent code styling in PHP projects. As I - the author - am a backend developer, it’s evident for me to write about PHP, but I am a 100% sure that these kind of tools exist for JavaScript, or any other programming language as well.

### What is PHPCodeSniffer?

PHPCodeSniffer, often referred to as PHPCS, is a widely-used static analysis tool that checks PHP code against a predefined coding standard. It can identify and report coding style violations, making it an invaluable tool for maintaining clean and consistent code in PHP projects.

### Creating the git hook

Note: As all the scripts run on your computer, you should have all the required software. To run composer scripts, you should have Composer, therefore PHP installed and configured in your PATH in order to be able to run as a command.
{: .info-blue }

1. Create the pre-commit hook script: inside your Git repository, navigate to the `.git/hooks` directory. You'll find a file named `pre-commit.sample`. Duplicate it and rename the copy to `pre-commit` (without the .sample extension), or just use the sample file by removing the extension.
2. Edit the pre-commit hook script: Open the `pre-commit` script in a text editor of your choice and add the following code:

```shell
#!/bin/sh

composer phpcs
exit $?
```
The above snippet supposes that you are working on a Composer project, and have a line in your `composer.json` similar to this:

```yaml
"scripts": {
    "phpcs": "phpcs --standard=PSR12 .",
  },
```

4. Make the script executable: ensure the pre-commit script is executable. You can do this by running:


```shell
chmod +x .git/hooks/pre-commit
```

Now, each time you attempt to make a commit, the pre-commit hook will automatically run PHPCodeSniffer to check your PHP code for adherence to the configured coding standard. If any violations are found, the commit will be blocked until you address them, ensuring that only clean and compliant code makes its way into your repository.

### Running PHPCBF and PHPUnit on git hooks

Running PHP Code Beautifier (PHPCBF) and PHPUnit on the pre-commit hook may seem like a good idea to maintain the high quality of the codebase, but it comes with downsides. These processes can be slow, especially in larger codebases, causing delays in your development workflow.

Also, not every commit requires these checks; minor changes or non-code modifications may not benefit from them. It's more efficient to run PHPCBF and PHPUnit locally and only when it is needed, ensuring code quality without slowing down commits for minor changes. This approach strikes a balance between quality and developer productivity.

## 3) Prepending the Jira ID to the commit message
Many organizations follow a strict convention of including Jira issue IDs in commit messages for effective issue tracking and traceability. In this section, we'll explore how to automate this process using a prepare-commit-msg Git hook. The following hook ensures that the Jira issue ID from the currently checked-out branch is automatically added to the beginning of each commit message, helping maintain consistency and compliance with the company's policies.

### Setting up the prepare-commit-msg hook
To automatically prepend the Jira issue ID to commit messages using a prepare-commit-msg hook, follow these steps:

1. Navigate to the .git/hooks Directory: inside your Git repository, navigate to the `.git/hooks` directory.
2. Create the prepare-commit-msg hook script: create a new file named `prepare-commit-msg` (without any file extension) within the .git/hooks directory.
3. Edit the prepare-commit-msg hook script: open the `prepare-commit-msg` script in a text editor and add the following code:

```shell
#!/bin/sh

# Get the name of the current branch by matching the second group of the regex

valid_branch_regex="^(feature/|bugfix/|release/|hotfix/)?(PRJT-[0-9]+)(_[a-z-]+)?$"
local_branch="$(git rev-parse --abbrev-ref HEAD)"

if [[ $local_branch =~ $valid_branch_regex ]]; then
    issue="${BASH_REMATCH[2]}"
else
    echo "Failed to fetch issue ID from branch name"
    exit 1
fi

# Prepend the [issue key] to the commit message

commit_msg_file="$1"
if [ -f "$commit_msg_file" ] && [ -n "$issue" ]; then
    tmp_file=$(mktemp)
    echo "[$issue] $(cat "$commit_msg_file")" > "$tmp_file"
    mv "$tmp_file" "$commit_msg_file"
else
    echo "Commit message or Jira issue not found"
    exit 1
fi
```

4. Make the script executable: ensure the prepare-commit-msg script is executable by running:

```shell
chmod +x .git/hooks/prepare-commit-msg
```

### How the prepare-commit-msg hook works
When you make a commit, Git triggers the prepare-commit-msg hook. This script extracts the Jira issue ID from the current branch name by searching for patterns like "PRJT-12345" using regex groups. If a Jira issue ID is found, it automatically prepends it to the commit message using a generated commit message file.

This automation eliminates the need for you to manually add Jira issue IDs to each commit message, ensuring that every commit is properly and uniformly tagged (without typos) for issue tracking purposes. It's an effective way to enforce commit message conventions and streamline the development process.

## 4) Validating commit messages

Ensuring consistent and informative commit messages is essential for effective collaboration and version control. In this section, we'll discuss how to use a commit-msg Git hook to validate commit messages, including compliance with the Conventional Commits standard. While this standard may not be mandatory in the repository you are working on, it can play a vital role in defining the risk and impact of pull requests and maintaining clarity in your Git history.

### A note about Conventional Commits

Conventional Commits is a widely adopted convention for structuring commit messages, making them more informative and consistent. While not mandatory, using Conventional Commits can greatly benefit your development workflow, as it is the commit message equivalent of Semantic Versioning (and SemVer can be programatically calculated based on Conventional commit messages). Here are the basic rules:

- Commit message structure:
    - Each commit message should have a structured format: <`type>(optional scope): <description>`. It consists of three parts:
        - \<type>: Describes the purpose of the commit, such as "feat" for a new feature, "fix" for a bug fix, "chore" for routine tasks, and more.
        - (optional scope): Indicates the scope of the commit, which is optional but helps provide context.
        - : \<description>: A concise and clear description of the changes made in the commit.
- Breaking changes: If a commit introduces breaking changes, it should be marked with an exclamation mark ! right after the \<type>. For example, `feat!: change API endpoint`.

Using Conventional Commits fosters better collaboration and clarity in your version control history. To explore this convention further and learn about advanced use cases, consider visiting the official website at [conventionalcommits.org](https://conventionalcommits.org).

### Setting up the commit-msg hook
To enforce commit message validation, including Conventional Commits standard, follow these steps:

1. Navigate to the .git/hooks directory: inside your Git repository navigate to the `.git/hooks` directory.
2. Create the commit-msg hook script: create a new file named `commit-msg` (without any file extension) within the .git/hooks directory.
3. Edit the commit-msg hook script: open the `commit-msg` script in a text editor and add the following code:

```shell
#!/bin/sh

# Check if the commit message matches the required format:
#   - (prepended automatically) [PRJT-number]
#   - conventional category of the commit
#   - (optional) name of the changed component in parentheses
#   - (optional) exclamation mark to indicate breaking change
#   - a colon and a space
#   - a message containing letters (upper- or lowercase), numbers, commas, dots, parentheses
#   OR
#  - a default Merge commit (with auto-prepended [PRJT-number])

conventional_regex="^\[PRJT-[0-9]+\] (build|chore|ci|docs|feat|fix|perf|refactor|revert|style|test)(\([^\)]+\))?(!)?: [A-Za-z0-9 ,.()]+$"
merge_regex="^\[PRJT-[0-9]+\] Merge branch '[^']+' into [A-Za-z0-9_-]+$"

commit_msg_file="$1"
commit_msg=$(cat "$commit_msg_file")

if [[ $commit_msg =~ $conventional_regex ]]; then
    exit 0
fi

if [[ $commit_msg =~ $merge_regex ]]; then
    exit 0
fi
echo "Commit message does not match a required pattern:"
echo "Pattern 1: $conventional_regex"
echo "Pattern 2: $merge_regex"
echo "Commit message: $commit_msg"
exit 1
```

4. Make the script executable: ensure the commit-msg script is executable by running:

```shell
chmod +x .git/hooks/commit-msg
```

### Commit message validation
The commit-msg hook uses two regular expression patterns to validate commit messages. The first pattern enforces Conventional Commits, which categorize commits with prefixes like "feat" (feature), "fix" (bug fix), "chore" (routine tasks), and more. The second pattern allows the merge of branches.

While Conventional Commits may not be mandatory in your project, adhering to them greatly aids in defining the risk and impact of pull requests. Clear and standardized commit messages provide valuable context and traceability in your version control history. By using this commit-msg hook, you encourage best practices in commit message formatting and enhance other developer’s ability to understand the significance of each change.

## Conclusion

In the realm of version control, Git hooks stand out as indispensable tools for automating tasks, maintaining code quality, and ensuring consistency. We've explored pre-commit, prepare-commit-msg, commit-msg, and pre-push hooks, each serving a unique purpose in enhancing your development workflow.

Git hooks are versatile tools, adaptable to any team's specific needs and preferences. As you integrate them into your workflow, you're enhancing code quality and facilitating more efficient collaboration. Embrace the automation, adhere to best practices, and enjoy the benefits of smoother, more productive development.

Git hooks are more than scripts; they're your allies on the path to efficient, organized, and high-quality software development.