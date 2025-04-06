## Git Tutorial Part Additinoal: Tagging, Cherry-Picking, Stashing, Reflog & Rebase

This concluding part looks at marking important points in history, applying specific commits, managing work in progress, recovering lost states, and rewriting commit history with rebase.

---

### Part 1: `git tag` - Marking Specific Points

* **Purpose:** Creates named pointers (tags) to specific commits, typically used to mark release versions (e.g., `v1.0`, `v2.1.3`) or other significant points in history. Tags, unlike branches, generally don't move once created.
* **Types of Tags:**
    1.  **Lightweight Tags:** Simple pointers to a commit hash. No extra info stored.
        * `git tag <tagname>`: Creates a lightweight tag pointing to the commit currently checked out (HEAD).
        * `git tag <tagname> <commitId>`: Creates a lightweight tag pointing to the specified commit ID.
    2.  **Annotated Tags (Recommended for Releases):** Full Git objects that store extra metadata: the tagger's name, email, date, a tagging message, and a GPG signature if configured.
        * `git tag -a <tagname> -m "Your tagging message" [<commitId>]`
            * `-a`: Signifies an annotated tag.
            * `-m "message"`: Provides the annotation message (e.g., "Version 1.0 release").
            * `[<commitId>]`: Optionally specify the commit to tag; defaults to HEAD if omitted.
* **Listing Tags:** `git tag` lists all existing tags.
* **Pushing Tags:** Tags created locally are **not** automatically sent to the remote repository with `git push`. You must push them explicitly:
    * `git push origin <tagname>`: Pushes a single, specific tag to the remote named `origin`.
    * `git push origin --tags`: Pushes all your local tags to `origin` that are not already there.

---

### Part 2: `git cherry-pick` - Applying Specific Commits (Revisited)

* **`git cherry-pick <commitId>`**
    * **Purpose:** Selects the changes introduced by a single, specific commit (identified by `<commitId>`, often from another branch) and applies those changes as a **new commit** on top of your current branch (HEAD).
    * **Outcome:** Creates a new commit on your current branch. This new commit will have the same code changes (patch) and usually the same commit message as the original `<commitId>`, but it will have a different SHA-1 hash because its parent commit and timestamp are different.
    * **Use Case:** Useful for bringing specific bug fixes or small features from one branch to another without merging the entire source branch. Can sometimes lead to conflicts if the surrounding code has changed significantly on the target branch.

---

### Part 3: `git stash` - Temporarily Shelving Changes

* **Purpose:** Allows you to quickly save your uncommitted local changes (both staged and unstaged modifications in your working directory) to a temporary storage area (the "stash stack"), cleaning your working directory back to the state of the last commit (HEAD). This lets you easily switch branches or pull updates without needing to commit incomplete work.
* **Common Commands:**
    * `git stash` or `git stash push [-m "Optional message"]`: Saves your current working directory changes (staged and unstaged) onto the stash stack and cleans your working directory. `-m` adds a description.
    * `git stash list`: Shows all the entries currently saved in your stash stack (e.g., `stash@{0}`, `stash@{1}`). It's a Last-In, First-Out (LIFO) stack.
    * `git stash pop [--index] [<stash>]`: Applies the changes from a specific stash (defaults to the latest, `stash@{0}`) back to your working directory *and removes* that stash entry from the list.
        * `--index`: Attempts to also restore the *staged* status of changes as they were when you stashed them. This might fail if applying the stash creates conflicts.
    * `git stash apply [--index] [<stash>]`: Similar to `pop`, but it applies the changes *without removing* the stash entry from the list. Useful if you want to apply the same stashed changes to multiple branches.
    * `git stash drop [<stash>]`: Deletes a specific stash (defaults to `stash@{0}`) from the list permanently.
* **Use Case:** You're working on `feature-A`, have uncommitted changes, but need to quickly switch to `main` to fix an urgent bug. You run `git stash`, switch to `main`, fix the bug, commit, switch back to `feature-A`, and run `git stash pop` to get your previous work back.

---

### Part 4: `git reflog` - Your Safety Net

* **`git reflog`**
    * **Purpose:** Displays a log of where `HEAD` and your local branch tips have pointed throughout the history of your *local* repository. It records actions like commits, checkouts, resets, merges, rebases, cherry-picks, and amends.
    * **Technical Detail:** Git keeps this reference log (usually in `.git/logs/`) even for operations that rewrite or seem to destroy history (like `reset --hard` or deleting branches). The actual commit objects often remain in the Git database for a while, just becoming "unreachable". The reflog provides pointers (commit hashes) to these states.
    * **Use Case:** **Essential for recovery!** If you accidentally perform a destructive operation (like a bad `reset --hard`, deleting the wrong branch before merging, or messing up a rebase), `git reflog` is often the first place to look. Find the SHA-1 hash of the state you want to return to in the reflog output, and you can often recover using `git checkout <hash>` or `git branch <new-branch-name> <hash>`. Reflog entries expire eventually (defaults are typically 90 days for reachable refs, 30 days for unreachable ones).

---

### Part 5: `git rebase` - Rewriting History (Use with Caution!)

* **Purpose:** Reapplies commits from the current branch onto a different base commit, creating a *linear history*. It rewrites commit history by creating new commits.
* **Basic Rebase (`git rebase <base-branch>`):**
    * Imagine your `feature` branch was created from `main` some time ago. Since then, `main` has received new commits. You are on `feature` and run `git rebase main`.
    * **Process:**
        1.  Git finds the common ancestor of `feature` and `main`.
        2.  It saves the commits unique to `feature` (those made after the common ancestor) temporarily.
        3.  It resets `feature` to point to the latest commit on `main`.
        4.  It then reapplies each saved commit, one by one, onto the new end of `feature`.
    * **Outcome:** The `feature` branch now appears to have started from the latest commit of `main`. The *original* commits on `feature` are discarded, and **new commits (with different SHA-1 hashes but the same changes)** are created. This results in a cleaner, straight-line history compared to a merge commit.
* **`rebase` vs. `merge`:**
    * `merge` joins two histories with a merge commit, preserving the exact historical paths (non-linear graph).
    * `rebase` replays commits onto a new base, creating a linear history but changing commit identities.
* **Interactive Rebase (`git rebase -i <base>`):**
    * A powerful mode that opens an editor listing the commits about to be rebased. It allows you to manipulate these commits *before* they are reapplied: `reorder`, `reword` (change message), `edit` (stop to amend), `squash` (combine into previous commit), `fixup` (squash and discard message), `drop` (delete commit).
    * Excellent for cleaning up your local commit history *before* sharing it.
* **CRITICAL WARNING:** Because `rebase` **rewrites history** (creates new commit SHA-1s for existing changes), you should **NEVER rebase commits that have already been pushed to a remote repository and might be used by others.** Doing so forces history to diverge and creates major problems for collaboration.
    * **Rule of Thumb:** Rebase *your own local, unshared commits* to clean them up. Use `merge` to integrate changes from shared branches (like pulling updates from `main` or merging a shared feature branch).