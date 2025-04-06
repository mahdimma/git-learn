## Git Tutorial Part 4: Undoing Changes, Managing Files, and More

This tutorial covers essential commands for correcting mistakes, reorganizing files tracked by Git, and renaming branches. It includes `git restore`, `git rm`, `git mv`, `git branch -m`, `git reset`, and `git revert`.

---

### Part 1: `git restore` - Undoing Working Directory & Staged Changes

`git restore` is a modern (introduced more recently compared to `reset` or `checkout`) command specifically designed for undoing changes in your working directory or staging area.

1.  **`git restore <file>`**

    - **Purpose:** Discards changes made to `<file>` _in your working directory_.
    - **Technical Detail:** This command takes the version of `<file>` that is currently in the _staging area (index)_ and uses it to overwrite the version in your working directory. If the file hasn't been staged (`git add`-ed) since the last commit, it effectively restores the file to match the version in the last commit (HEAD).
    - **Use Case:** Use this when you've made modifications to a file locally that you decide you don't want to keep. **Warning:** This operation discards your working directory changes for that file without easy recovery.

2.  **`git restore --staged <file>`** (Opposite of `git add`)
    - **Purpose:** Removes the changes for `<file>` _from the staging area_ (unstages them).
    - **Technical Detail:** This command resets the entry for `<file>` in the staging area (index) to match the version in the last commit (HEAD). Importantly, it _does not_ modify the file in your working directory. Any changes you made remain in the working directory file, but they are no longer marked ("staged") to be included in the next commit.
    - **Use Case:** Use this when you've accidentally staged a file or changes within a file using `git add` and want to remove them from the staging area before committing, without losing the actual modifications in your working file.

---

### Part 2: Managing Files with `git rm` and `git mv`

These commands help manage files both in your working directory and within Git's tracking system.

1.  **`git rm <file>`**

    - **Purpose:** Removes a file from Git's tracking _and_ deletes it from the working directory.
    - **Technical Detail:** This performs two actions:
      1.  Removes the file from the staging area (index). This removal is staged for commit.
      2.  Deletes the file from your filesystem (working directory).
    - **Use Case:** When you want to completely delete a file from your project and stop tracking it. The deletion will become part of the project history once committed.
    - **Variant:** `git rm --cached <file>` removes the file _only_ from Git tracking (the index) but leaves the file untouched in your working directory. Useful if you previously tracked a file but now want to ignore it (e.g., after adding it to `.gitignore`).

2.  **`git mv <old-filename> <new-filename>`**
    - **Purpose:** Moves or renames a file that is tracked by Git.
    - **Technical Detail:** This command is essentially a convenience wrapper that performs the necessary steps for Git to understand a file rename or move:
      1.  Renames the file on the filesystem (`mv old new`).
      2.  Removes the old filename from the index (`git rm old`).
      3.  Adds the new filename to the index (`git add new`).
    - **Use Case:** The standard way to rename or move files within a Git repository, ensuring Git tracks the history correctly as a rename/move rather than a deletion and addition of unrelated files. The change is staged for the next commit.

---

### Part 3: `git branch -m` - Renaming a Branch

1.  **`git branch -m [<old-branch-name>] <new-branch-name>`**
    - **Purpose:** Renames a local branch.
    - **How to Use:**
      - **To rename the branch you are currently on:** Switch to that branch first (if needed) using `git switch <branch-to-rename>`, then run:
        ```bash
        git branch -m <new-branch-name>
        ```
      - **To rename a branch you are _not_ currently on:** You can specify the old name:
        ```bash
        git branch -m <old-branch-name> <new-branch-name>
        ```
    - **Technical Detail:** This updates the reference file for the branch (in `.git/refs/heads/`). If this branch tracks a remote branch and/or you've pushed it, you will need additional steps to update the remote repository (typically involving deleting the old branch name on the remote and pushing the new one).

---

### Part 4: `git diff HEAD~1 HEAD` - Reviewing the Last Commit

- **Purpose:** A specific use of `git diff` to show exactly what changed in the _most recent commit_.
- **Technical Detail:**
  - `HEAD` refers to the commit your current branch is pointing to (the latest commit).
  - `HEAD~1` refers to the first parent of `HEAD` (the commit immediately before the latest one).
  - `git diff HEAD~1 HEAD` compares the state of the repository _before_ the last commit (`HEAD~1`) with the state _after_ the last commit (`HEAD`) (also can go more back with 2, 3, ...), thereby showing precisely the changes introduced by that single commit. Useful for quick reviews or understanding the scope of the last piece of work committed.

---

### Part 5: `git reset` - Resetting HEAD, Index, and Working Directory (Use with Care!)

`git reset` is a powerful command used to move the current branch HEAD pointer to a specific commit, potentially rewriting history and affecting the staging area and working directory. **Use caution, especially with `--hard`, as it can discard work.**

1.  **Modes:** The behavior depends heavily on the mode (`--soft`, `--mixed`, `--hard`). The command structure is `git reset [<mode>] <commit>`. `<commit>` can be a SHA-1 hash, branch name, tag, or relative reference like `HEAD~N`.

    - **`git reset --soft <commit>`**

      - **Action:** Moves the HEAD branch pointer to `<commit>`.
      - **Effect:** Only the branch pointer moves. The staging area (index) and the working directory are _unchanged_. All the differences between the original HEAD and `<commit>` will appear as _staged_ changes.
      - **Use Case:** To undo one or more commits locally while keeping all the changes staged, perhaps to recommit them as a single "squashed" commit or with a different message.

    - **`git reset --mixed <commit>`** (This is the **default** mode if no mode is specified)

      - **Action:**
        1. Moves the HEAD branch pointer to `<commit>`.
        2. Resets the staging area (index) to match `<commit>`.
      - **Effect:** The branch pointer moves, and the index is updated. Changes from the commits between `<commit>` and the original HEAD are kept in the _working directory_ but are _unstaged_.
      - **Use Case:** To undo commits locally, review the changes in the working directory, and selectively re-stage and re-commit them differently.

    - **`git reset --hard <commit>`**
      - **Action:**
        1. Moves the HEAD branch pointer to `<commit>`.
        2. Resets the staging area (index) to match `<commit>`.
        3. **Resets the working directory** to match `<commit>`.
      - **Effect:** Everything (HEAD, index, working directory) is forced to match the state of the project at `<commit>`. **Any changes made after `<commit>`, whether committed or uncommitted, in the staging area or working directory, are permanently discarded.**
      - **Use Case:** To completely throw away recent commits and any subsequent work, returning to a known good state. **EXTREME CAUTION ADVISED.** This is often irreversible (beyond relying on `git reflog` for a short time) and should generally _not_ be used on commits that have been shared/pushed.

---

### Part 6: `git revert` - Safely Undoing Committed Changes

`git revert` is the standard, safe way to undo the effects of a previous commit, especially if that commit has been shared.

1.  **`git revert <commit>`**
    - **Purpose:** Creates a _new commit_ that introduces the exact opposite changes of the specified `<commit>`.
    - **Technical Detail:** It analyzes the changes made in `<commit>` and calculates the inverse patch. It then creates a _new commit_ on top of your current history that applies this inverse patch. The original `<commit>` remains part of the project history, but its effects are cancelled out by the new revert commit. Git will usually open your editor to confirm the default commit message (e.g., "Revert 'Commit message of original commit'").
    - **Use Case:** The preferred method for undoing commits that have already been pushed to a shared repository. It doesn't rewrite history, so it avoids confusing collaborators. It clearly documents that a previous change was undone.
    - **vs. `reset`:** `reset` effectively removes or alters past commits (history rewriting), while `revert` adds a _new_ commit to undo a previous one (history preserving).
