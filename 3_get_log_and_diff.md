## Git Tutorial Part 3: Viewing History, Comparing Changes, and Cherry-Picking

This part focuses on commands that help you inspect your project's history and understand the differences between various states.

---

### Part 1: `git log` - Viewing Commit History

- **Core Purpose:** The `git log` command is used to display the history of commits in your repository. By default, it shows commits reachable from your current position (HEAD), starting with the most recent.

- **Common Options (often used together):**

  1.  **Formatting Commit Output:**

      - `--abbrev-commit`
        - **What it does:** Instead of showing the full 40-character SHA-1 hash for each commit, it displays a shorter, unique abbreviation (usually the first 7 characters, but Git ensures uniqueness).
        - **Why use it:** Makes the log output significantly cleaner and easier to read, while still providing a usable reference to the commit.
      - `--pretty=oneline`
        - **What it does:** This is a specific format option for the `--pretty` flag. It condenses the information for each commit onto a single line, showing the full commit SHA-1 hash followed by the commit message title.
        - **Why use it:** Provides a very compact view of the history, focusing on the commit ID and its purpose.
      - `--oneline`
        - **What it does:** This is a convenient shorthand alias for `--pretty=oneline --abbrev-commit`.
        - **Result:** Each commit appears on a single line with its _abbreviated_ SHA-1 hash and the commit message title. This is arguably the most common and useful compact log format.

  2.  **Controlling Scope and Visualization:**
      - `--all`
        - **What it does:** By default, `git log` only shows commits reachable from the current HEAD (your current branch tip). `--all` tells Git to show commits from _all_ branches and other references (like tags) in the repository.
        - **Why use it:** Essential for getting a complete picture of all development lines and how they relate, not just your current line of work.
      - `--graph`
        - **What it does:** Draws a text-based graph in the left margin of the output, visually representing the branch structure and merge history. Lines connect commits to their parents, clearly showing where branches diverged and merged.
        - **Why use it:** Invaluable for understanding complex histories, especially when multiple branches are involved. Makes merge points and branches explicit.

- **Example Combining Options:** A very common and useful invocation is:
  ```bash
  git log --graph --oneline --all
  ```
  This gives you a compact, single-line view of each commit with an abbreviated hash, includes commits from all branches, and draws a graph to show how they interconnect.

---

### Part 2: `git diff` - Showing Differences

- **Core Purpose:** The `git diff` command is used to show changes between different states tracked by Git, such as commits, branches, the working directory, and the staging area (index).

- **Common Usage Patterns:**

  1.  **`git diff <thing1> <thing2>`** (Comparing two specific points)

      - **What it does:** Shows the differences _between_ two specific references in your Git history. `<thing1>` and `<thing2>` can be commit SHA-1 hashes (full or abbreviated), branch names, tag names, or special references like `HEAD` (current commit) or `HEAD~N` (N commits before HEAD).
      - **Example:** `git diff main feature-branch` shows changes introduced on `feature-branch` since it diverged from `main`. `git diff HEAD~1 HEAD` shows the changes introduced by the very last commit.
      - **Output:** Shows which files differ and the specific lines added or removed between the two points.

  2.  **`git diff <thing>`** (Comparing one specific point to the working directory)

      - **What it does:** Shows the differences between the specified commit/branch/tag (`<thing>`) and the files currently in your _working directory_.
      - **Example:** `git diff HEAD` shows all changes in your working directory (both staged and unstaged) compared to the last commit. `git diff main` shows how your current working directory differs from the tip of the `main` branch.

  3.  **`git diff`** (With no arguments)

      - **What it does:** Shows the differences between the _staging area (index)_ and your _working directory_.
      - **Result:** This specifically highlights changes in tracked files that you have made but _have not yet staged_ using `git add`.

  4.  **`git diff <path>`** (With just a file or directory path)
      - **What it does:** Shows the differences for the specified file(s) or directory between the _staging area (index)_ and your _working directory_. It's like the no-argument `git diff` but filtered to a specific path.

- **Useful Options:**

  1.  **`--cached` or `--staged`** (These are synonyms)
      - **What it does:** Shows the differences between the _last commit (HEAD)_ and the _staging area (index)_.
      - **Why use it:** This command is crucial for seeing exactly what changes you _have staged_ (`git add`-ed) and _will be included_ in the next commit if you run `git commit` right now. It helps review before committing.
  2.  **`--word-diff`**
      - **What it does:** Instead of showing differences line by line, it highlights changes on a _word_ basis within lines. Added words might be shown like `{+added+}` and removed words like `[-removed-]`.
      - **Why use it:** Very useful for identifying small changes within long lines, especially common when editing prose, documentation, or configuration files where only a few words might change in a paragraph.

---

### Part 3: `git cherry-pick` - Applying Specific Commits

- **Just the Point:** `git cherry-pick` is used to take the changes introduced by an _existing commit_ from somewhere else in the repository history (e.g., another branch) and apply them as a _new commit_ on your current branch.

- **Concept:** Imagine you have a bug fix committed on a feature branch, but you need that _same fix_ on your `main` branch _without_ merging the entire feature branch yet. You can identify the commit hash of the bug fix on the feature branch and then `git cherry-pick <commit-hash>` while on your `main` branch. Git will replay the _changes_ from that commit onto `main` and create a new commit reflecting those changes. The new commit will have the same message and author but a different SHA-1 hash and timestamp, as it's a distinct commit in a different context.

- **Use Cases:** Applying specific bug fixes, selectively bringing small features across branches, or reordering commits (though `rebase` is often preferred for the latter). Be mindful that it duplicates changes and can sometimes lead to conflicts if the context has changed significantly.
