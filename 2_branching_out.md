## Git Tutorial Part 2: Branches, Merging, and Workflow Tools

This tutorial builds on the basics of file states and commits, exploring how Git manages different lines of development using branches, how to combine them, and some helpful workflow commands.

---

### Part 1: Understanding and Using Branches

1.  **What is a Branch?**

    - **Concept:** Think of a branch as an independent line of development within your project. It allows you to work on a new feature, bug fix, or experiment without affecting the main, stable version of your code until you're ready.
    - **Technical Detail:** A branch in Git is fundamentally just a lightweight, movable _pointer_ to a specific commit. When you create a branch, Git creates a new pointer (a small file usually located in `.git/refs/heads/`) that points to the _same commit_ you are currently on (HEAD). As you make new commits _on that new branch_, the pointer automatically moves forward to point to the latest commit in that line of development. The original branch pointer remains unaffected. This makes branching very fast and efficient, as Git doesn't initially copy your entire project directory.

2.  **Default Branch Name**

    - **Common Names:** When you initialize a new Git repository (`git init`), Git creates a default branch for you. Historically, this was named `master`. In recent years, `main` has become the increasingly common and recommended default name. This is typically considered the primary, stable line of development.

3.  **Branching Commands**
    - **`git branch <newBranchName>`**
      - **Purpose:** Creates a _new_ branch with the specified name.
      - **Technical Detail:** This command adds a new pointer named `<newBranchName>` that points to the _current commit_ (the commit HEAD is currently pointing to). Importantly, this command _only creates_ the branch; it _does not_ switch your working directory to that new branch. You remain on the branch you were on before running the command.
    - **`git branch`**
      - **Purpose:** Lists all the local branches in your repository.
      - **Technical Detail:** This command reads the branch pointers stored in your repository (typically in `.git/refs/heads/`). It will usually display the list with the currently active branch (the one your HEAD is pointing to) marked with an asterisk (`*`) and potentially highlighted.
    - **`git switch <branchName>`**
      - **Purpose:** Changes your active working environment to the specified _existing_ branch.
      - **Technical Detail:** This command performs two main actions:
        1.  It updates the `HEAD` pointer (a special pointer usually located in `.git/HEAD`) to point to the specified `<branchName>`.
        2.  It modifies the files in your working directory to match the snapshot of the project stored in the commit that `<branchName>` points to. Git will try to preserve uncommitted local modifications if they don't conflict with the files being checked out; otherwise, it might prevent the switch or require you to stash/commit changes first.
    - **`git switch -c <newBranchName>`**
      - **Purpose:** A convenient shortcut command that _creates_ a new branch named `<newBranchName>` _and immediately switches_ your working directory to it.
      - **Technical Detail:** This is equivalent to running `git branch <newBranchName>` followed by `git switch <newBranchName>`. It creates the new branch pointer based on the current commit, then updates `HEAD` and your working directory to this newly created branch context.

---

### Part 2: Merging Branches

1.  **What is Merging?**

    - **Concept:** Merging is the process of taking the independent lines of development created on different branches and combining them back together. Typically, you merge a completed feature branch _into_ your main development branch (like `main`).
    - **Technical Detail:** When you ask Git to merge, it looks at the commits on both branches since they diverged from a common ancestor. It tries to integrate the changes from the source branch into the target branch (the one you are currently on). If the target branch hasn't had any new commits since the source branch was created, Git might perform a "fast-forward" merge (simply moving the target branch's pointer to the same commit as the source branch). More commonly, Git performs a "three-way merge" by comparing the tips of both branches and their common ancestor, creating a new special "merge commit" that has _two_ parent commits – one from each branch being merged.

2.  **Merge Command**
    - **`git merge <someBranch>`**
      - **Purpose:** Integrates the changes and history from `<someBranch>` _into_ the branch you are _currently checked out on_.
      - **Prerequisite:** You must first switch to the branch you want to receive the changes (e.g., `git switch main`).
      - **Technical Detail:** Running this command initiates the merge process described above. Git attempts to automatically combine the work. If successful (and not a fast-forward), it creates a merge commit. If Git encounters conflicting changes (see Part 4), the merge process will pause, requiring manual intervention.

---

### Part 3: Getting Help

1.  **How to Ask Git for Help**
    - **`git <command> --help`**
      - **Purpose:** Provides the most comprehensive help for a specific Git command.
      - **Technical Detail:** This typically opens the full, official Git documentation (man page or HTML page) for the `<command>` in your default viewer (like a web browser or terminal pager). It contains detailed explanations, all options, configuration variables, and examples.
    - **`git <command> -h`**
      - **Purpose:** Shows a concise summary of usage and common options for a command, directly in your terminal.
      - **Technical Detail:** This prints a brief help message to your standard output. It's useful when you just need a quick reminder of a command's syntax or main flags, without leaving the terminal.

---

### Part 4: Setting Up Your Editor

1.  **Configuring the Default Text Editor**
    - **`git config --global core.editor "<editor command>"`**
      - **Purpose:** Tells Git which text editor program to use when it needs you to enter text, such as writing commit messages (if you don't use `-m`), editing interactive rebase instructions, or resolving merge conflicts manually.
      - **Technical Detail:** This command modifies Git's configuration settings.
        - `--global`: This flag applies the setting to your user account system-wide, affecting all your Git repositories on that machine. The setting is typically stored in a file named `.gitconfig` in your home directory. Omit `--global` to set it only for the current repository (stored in `.git/config`).
        - `core.editor`: This is the specific configuration key for the editor.
        - `"<editor command>"`: Replace this with the actual command needed to launch your preferred editor from the terminal. Crucially, for graphical editors (like VS Code, Sublime Text, Atom), you often need specific flags that make the editor wait until you close the file, so Git knows you're finished. Examples:
          - `nano` (simple terminal editor): `git config --global core.editor "nano"`
          - `vim` (powerful terminal editor): `git config --global core.editor "vim"`
          - VS Code: `git config --global core.editor "code --wait"`
          - Sublime Text: `git config --global core.editor "subl -n -w"`
        - The quotes around the command are important, especially if the command includes spaces or flags.

---

### Part 5: Understanding and Fixing Merge Conflicts

1.  **What is a Conflict?**

    - **Concept:** A merge conflict occurs during a `git merge` (or similar operations like `git rebase` or `git cherry-pick`) when Git encounters changes in _both_ branches being combined that affect the _exact same lines_ within the _same file_. Git cannot automatically decide which version is correct, so it stops and asks you to resolve the ambiguity.
    - **Technical Detail:** Git detects that the content in a specific section of a file differs between the target branch's version and the source branch's version, relative to their last common ancestor commit. Since both branches modified the same part, automatic merging fails.

2.  **How to Fix a Conflict**
    - **Process:**
      1.  **Identify:** Git will clearly state that the merge failed due to conflicts and list the specific files that contain conflicts. `git status` will also show these files under an "Unmerged paths" section.
      2.  **Inspect:** Open the conflicted file(s) in your configured text editor. Git automatically modifies the file to show you the conflicting sections, marked with special markers:
          ```diff
          <<<<<<< HEAD
          // Code from the current branch you are merging INTO (e.g., main)
          =======
          // Code from the branch you are merging FROM (e.g., someBranch)
          >>>>>>> someBranch
          ```
          - `<<<<<<< HEAD`: Marks the beginning of the conflicting block from your current branch (HEAD).
          - `=======`: Separates the conflicting blocks.
          - `>>>>>>> <branchName>`: Marks the end of the conflicting block from the other branch (`<branchName>`).
      3.  **Resolve:** Manually edit the file to fix the conflict. You need to decide what the final code should look like. This might involve keeping one version, keeping the other, combining parts of both, or writing completely new code. **Crucially, you must also delete the conflict marker lines (`<<<<<<<`, `=======`, `>>>>>>>`).**
      4.  **Stage:** After you have edited the file, saved it, and are satisfied with the resolution, you need to tell Git that the conflict is resolved. Do this using `git add <resolved-file-name>`. This stages the corrected version of the file.
      5.  **Repeat:** If there were multiple conflicted files, repeat steps 2-4 for each one.
      6.  **Commit:** Once _all_ conflicts have been resolved and staged (`git status` should show "All conflicts fixed but you are still merging"), you can finalize the merge by running `git commit`. Git usually provides a default merge commit message; you can use it or edit it (if your editor opens). This command completes the merge process by creating the merge commit.

---

### Part 6: Deleting and Recovering Branches

1.  **Deleting a Branch**

    - **`git branch -d <branchName>`**
      - **Purpose:** Deletes the specified local branch.
      - **Technical Detail:** This removes the branch pointer file (e.g., `.git/refs/heads/<branchName>`). The `-d` (lowercase 'd', for --delete) flag includes a safety check: Git will _refuse_ to delete the branch if it contains commits that haven't been merged into your current branch (or another branch that is reachable from HEAD). This prevents accidental loss of work.
      - **Output Note:** When successful, Git often prints a confirmation message like `Deleted branch <branchName> (was <commit-id>).`. This `<commit-id>` (the SHA-1 hash of the commit the branch pointed to) is useful information if you need to recover the branch.
    - **Force Deletion:** If you are certain you want to delete a branch even if it's unmerged, use the uppercase `-D` flag: `git branch -D <branchName>`. Use with caution!

2.  **Recovering a Deleted Branch (Handling Mistakes)**
    - **Concept:** Even if you delete a branch pointer, the commits themselves usually remain in Git's database for some time before being garbage collected. Git maintains a log of where its references (like HEAD and branch tips) have pointed over recent history, called the `reflog`. You can use this log to find the commit the deleted branch pointed to and recreate the branch.
    - **Recovery Steps:**
      1.  **Find the Commit ID:** Use the command `git reflog`. Look through the output for the last known position of the deleted branch. You might see entries related to checking out that branch or committing to it. You're looking for the SHA-1 commit hash associated with the tip of the branch just before deletion. The message from `git branch -d` also provides this ID directly.
      2.  **Recreate the Branch:** Once you have the correct commit ID (let's call it `<base-commit-id>`), run the command:
          `git branch <branchName> <base-commit-id>`
      - **Technical Detail:** This command creates a _new_ branch pointer named `<branchName>` and makes it point directly to the commit specified by `<base-commit-id>`. This effectively restores the deleted branch, making its history accessible again.
