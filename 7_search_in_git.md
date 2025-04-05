## Git Tutorial Part 7: Code Archeology, Advanced Log Search, Checkout, and Bisect

This tutorial covers tools to understand _who_ changed _what_ and _when_, search through code and history effectively, revisit the versatile `checkout` command, and use `bisect` to efficiently find regressions.

---

### Part 1: Code Archeology (`blame`, `grep`)

These commands help you investigate the history and content of your files.

1.  **`git blame <file>`**

    - **Purpose:** Annotates each line in the specified `<file>`, showing which commit was the last one to modify that line, who the author of that commit was, and the commit timestamp.
    - **Technical Detail:** For the version of the file you specify (defaults to the version in `HEAD`, the current commit), `git blame` works backward through the commit history for each line individually. It stops when it finds the commit responsible for the line as it currently appears. You can blame a version from an older commit using `git blame <commit-hash> <file>`.
    - **Options:**
      - `-s`: Suppresses the author's name and timestamp, showing only the (abbreviated) commit SHA-1 hash for each line. Useful for more compact output or scripting.

2.  **`git grep <pattern>`**
    - **Purpose:** Searches for a specified `<pattern>` (string or regular expression) within the files tracked by Git. By default, it searches files in your current working directory.
    - **Technical Detail:** Similar to the standard command-line `grep`, but operates specifically on files known to Git. You can also make it search files as they existed in specific commits (e.g., `git grep 'myFunction' HEAD~3`).
    - **Common Options:**
      - `-i` or `--ignore-case`: Performs a case-insensitive search.
      - `-n` or `--line-number`: Prepends the line number within the file to each matching line in the output.
      - `-l` or `--files-with-matches`: Instead of printing the matching lines, it prints only the names of the files that contain at least one match. (Note: `-l` is the standard grep flag, sometimes people mistakenly use `--name-only` which belongs to other git commands like `diff`).

---

### Part 2: Advanced `git log` Searching & Output Options

`git log` has powerful options beyond simple listing.

1.  **`git log -S <string>`** (Pickaxe Search)

    - **Purpose:** Finds commits where the _number of occurrences_ of the specified `<string>` changed (i.e., the string was added or removed). It searches the _content diff_ (the patch) of each commit.
    - **Use Case:** Excellent for finding exactly when a specific function call, variable name, specific error message, etc., was introduced or deleted, even if the line containing it was modified in other ways.

2.  **`git log -p` or `git log --patch`**

    - **Purpose:** Shows the full patch (the diff showing lines added/removed) for each commit listed in the log output.
    - **Does `-p` work for all git commands?** **No.** The `-p`/`--patch` option primarily applies to commands that display commit information or differences, such as `git log`, `git show`, `git format-patch`, `git stash show`. It doesn't apply to commands like `git add`, `git status`, `git branch`, etc.
    - **Use Case:** Provides the essential context of _what changed_ in each commit, not just the commit message. Often used with other log filters.

3.  **`git log --word-diff`**

    - **Purpose:** When used in conjunction with `-p` (patch view), this option displays the diffs using a _word-level_ comparison instead of the default line-level comparison.
    - **Use Case:** Helps visualize small changes within lines more clearly, especially useful for prose or documentation changes.

4.  **`git log -G <regex>`** (Regex Diff Search)

    - **Purpose:** Finds commits where the added or removed lines in the _patch text_ contain matches for the specified `<regex>` (regular expression). Unlike `-S`, it doesn't care about the _count_ changing; it simply looks for the pattern within the diff lines.
    - **`vs -S`:** `-S` finds commits where the _occurrence count_ of a fixed _string_ changes in the file overall. `-G` finds commits where the _diff itself_ contains lines matching a _regex_. `-G` is useful for finding any commit that touched lines related to a certain pattern, even complex ones.
    - **Use Case:** Finding commits that affected code matching a certain pattern, for instance, any commit that modified lines containing `useState\(.*\)` in a React project.

5.  **`git log --grep <pattern>`**
    - **Purpose:** Filters the log output to show only those commits whose _commit message_ matches the specified `<pattern>`. The search is typically case-sensitive unless combined with `-i`.
    - **Use Case:** Searching for commits related to a specific issue number (e.g., `--grep="JIRA-123"`), author (`--author="Alice"` combined with `--grep="Refactor"`), or feature description mentioned in commit messages.

---

### Part 3: `git checkout` Revisited

`git checkout` is a versatile but sometimes confusing command because it performs several distinct functions. Modern Git introduced `git switch` and `git restore` for clearer separation, but `checkout` is still prevalent.

- **Key Functions:**
  1.  **Switching Branches:** `git checkout <branch-name>`
      - Updates HEAD to point to `<branch-name>`.
      - Updates your working directory to match the state of that branch.
      - (Modern equivalent: `git switch <branch-name>`)
  2.  **Restoring Working Directory Files:** `git checkout -- <file>`
      - Replaces the specified `<file>` in your working directory with the version from the staging area (index). Discards un-staged changes to that file.
      - (Modern equivalent: `git restore <file>`)
  3.  **Restoring Files from a Specific Commit:** `git checkout <commit> -- <file>`
      - Replaces the specified `<file>` in your working directory _and_ staging area with the version from the specified `<commit>`.
      - (Modern equivalent: `git restore --source=<commit> <file>`)
  4.  **Creating & Switching Branch:** `git checkout -b <new-branch-name>`
      - Creates `<new-branch-name>` based on the current commit.
      - Immediately switches HEAD and the working directory to the new branch.
      - (Modern equivalent: `git switch -c <new-branch-name>`)
  5.  **Detached HEAD State:** `git checkout <commit-hash>` or `git checkout <tag-name>`
      - Moves HEAD to point directly to a specific commit/tag instead of a branch name.
      - Useful for inspecting past states, but be cautious: new commits made here don't belong to any branch unless you create one.

---

### Part 4: `git bisect` - Finding Regressions Efficiently

`git bisect` is a powerful debugging tool that uses a binary search algorithm to quickly find the specific commit that introduced a bug.

- **Workflow:**

  1.  **Start Session:**
      - `git bisect start`
        - Initiates the bisect process.
  2.  **Define Boundaries:**
      - `git bisect bad [<commit>]`
        - Marks a commit where the bug _is known to exist_. Often, you'll just use `git bisect bad` to mark the current `HEAD` as bad.
      - `git bisect good <commit>`
        - Marks an older commit (like a tag or specific hash) where the bug _is known NOT to exist_.
  3.  **Test and Narrow Down:**
      - Git automatically checks out a commit roughly halfway between the known 'good' and 'bad' boundaries.
      - **Your Job:** Test the code at this commit to see if the bug is present.
      - **Report Result to Git:**
        - If the bug **IS present**: `git bisect bad`
        - If the bug **IS NOT present**: `git bisect good`
      - Git uses your feedback to eliminate half the remaining commit range and checks out a new commit in the middle of the narrowed range.
      - **Repeat** the testing and reporting (`bad`/`good`) process.
  4.  **Identify Culprit:**
      - With each step, the range of possible commits containing the bug is halved. Eventually, Git will narrow it down to the single commit that introduced the regression and will report its commit hash and details.
  5.  **End Session:**
      - `git bisect reset`
        - Cleans up the bisect state and returns your HEAD and working directory to where you were before starting the bisect (usually the branch you were on when you ran `git bisect start`).

- **Use Case:** Invaluable for efficiently pinpointing the source of a bug in a project with a long or complex commit history, saving significant manual effort.
