**Source of Information:**

It specifies two sources:

1.  **Head First Git:** "Head First" is a popular series of learning books known for their visual and engaging approach to complex topics.

2.  **https://i-love-git.com/:** This website, "i-love-git.com", is another source that have provided the information.this website is related to the "Head First Git" book.

In summary, the most subject comes from the book "Head First Git".

## Git File States: A Deep Dive

Okay, let's break down the fundamental Git file states and the core commands (`git add`, `git commit`) that manage them.

---

## Git Tutorial: Understanding File States and Basic Workflow

Git is a powerful version control system. At its heart, it manages the history of your project by tracking changes to files. To do this effectively, Git needs to know the _state_ of each file within your project directory (your "working directory"). There are main categories files can fall into, which dictate how Git interacts with them.

---

### Part 1: Git File States Explained

Git primarily sees files in two main categories: **Untracked** and **Tracked**. Tracked files have further sub-states.

1.  **Untracked Files**

    - **What it means:** These are files that exist in your working directory but are completely unknown to Git. Git isn't watching them for changes, and they have never been added to Git's tracking system (neither staged nor committed). Typically, these are newly created files you haven't told Git about yet.
    - **Technical Detail:** An untracked file exists only in your filesystem within the project's directory structure. There is no reference to it in Git's internal database (the object store) or in the "staging area" (also known as the index). It will _not_ be included in your next commit unless you explicitly tell Git to start tracking it.
    - **How to see them:** Running the command `git status` will list these files under the "Untracked files" heading.
    - **Analogy:** Imagine a brand new document you just placed on your desk. Your filing system (Git) doesn't even know it exists yet.

2.  **Tracked Files**
    - **What it means:** These are files that Git _knows_ about. They were present in the last snapshot (commit) of your project, or they have been explicitly added to Git's tracking (usually via the staging area). Git actively monitors these files for changes.
    - **Technical Detail:** A tracked file has a corresponding entry in Git's index (staging area) and/or exists as part of a previous commit object in Git's database. Git compares the version in your working directory against its known versions to determine the file's specific sub-state.
    - **Sub-States of Tracked Files:**
      - **a) Unmodified:**
        - **What it means:** The file exists in your working directory, Git is tracking it, and it has _not_ been changed since the last commit. The version on your disk is identical to the version in Git's last recorded snapshot (the HEAD commit).
        - **Technical Detail:** Git compares the checksum (SHA-1 hash) of the file in your working directory with the checksum stored in the index (staging area) _and_ the checksum associated with that file in the HEAD commit. If they all match, the file is unmodified. These files won't show up prominently in `git status` output (unless you use specific flags like `git status -uall`), because there's nothing "to do" with them currently.
        - **Analogy:** A document that's properly filed away and hasn't been touched since it was last filed.
      - **b) Modified:**
        - **What it means:** The file is tracked by Git, but you have made changes to it in your working directory _since_ the last commit. However, you have _not_ yet marked those changes to be included in the _next_ commit (i.e., you haven't staged them).
        - **Technical Detail:** Git calculates the checksum of the current file in your working directory and finds it _differs_ from the checksum recorded for that file in the HEAD commit. Because these changes haven't been added to the index yet, Git flags it as modified.
        - **How to see them:** `git status` will list these files under the "Changes not staged for commit" heading.
        - **Analogy:** You took a filed document, made edits to it, and it's now sitting on your desk, different from the filed version, but not yet in the "ready-to-be-refiled" tray.
      - **c) Staged:**
        - **What it means:** You have taken a modified file (or an untracked file) and marked its _current_ version to be included in the very next commit. These files are residing in a special holding area called the "Staging Area" or "Index".
        - **Technical Detail:** When you stage a file (using `git add`), Git takes the _current_ content of that file from your working directory, calculates its checksum, and places that information (the checksum and file path, essentially a pointer to the content) into the index file (`.git/index`). The index acts as a draft or manifest for what the _next_ commit will look like. Importantly, staging a file takes a snapshot of it _at that moment_. If you modify the file _again_ after staging it, those newer modifications will exist in the working directory but _not_ in the staging area (unless you stage it again).
        - **How to see them:** `git status` will list these files under the "Changes to be committed" heading.
        - **Analogy:** You've finished editing your document, and you've placed it in the special "to be filed" tray, ready for the next time you officially file things away.

---

### Part 2: The `git add` Command

- **Command:** `git add <file-name>` or `git add .` (to add all changes in the current directory and subdirectories)
- **Purpose:** This command is the bridge between your working directory and the staging area. It tells Git, "Prepare to include the _current_ state of this file (or these files) in the next commit."
- **Technical Function:** `git add` does two primary things depending on the file's current state:
  1.  **From Untracked to Staged (and Tracked):**
      - If you run `git add` on an `Untracked` file, Git performs these steps:
        - It reads the content of the file from your working directory.
        - It calculates the SHA-1 checksum of the content.
        - It stores the file's content in Git's internal object database (if it's not already there).
        - It adds an entry to the index (`.git/index`) that points to this content, marking the file path as ready for the next commit.
      - **Result:** The file transitions from `Untracked` -> `Staged`. Crucially, it also implicitly becomes a `Tracked` file from this point forward.
  2.  **From Modified to Staged:**
      - If you run `git add` on a `Tracked` file that is currently `Modified`:
        - Git reads the _new, modified_ content from your working directory.
        - Calculates its checksum.
        - Stores the new content in the object database (if needed).
        - It _updates_ the existing entry for that file in the index (`.git/index`) to point to this _new_ content.
      - **Result:** The file transitions from `Modified` -> `Staged`. The changes you made are now queued up for the next commit. The previous version (from the last commit) is still safe in Git's history; the staging area now reflects the intended _next_ version.

---

### Part 3: The `git commit` Command

- **Command:** `git commit -m "Your descriptive commit message"`
- **Purpose:** This command takes the snapshot of your project _as represented by the staging area (the index)_ and permanently stores it in your Git repository's history. It creates a new point in time you can always refer back to or revert to.
- **Technical Function:**
  1.  Git looks at the index (`.git/index`), which contains the list of files and their corresponding content checksums that you staged using `git add`.
  2.  It creates a new "tree" object representing the entire project structure based on the index.
  3.  It creates a new "commit" object which includes:
      - A pointer to the root tree object (the snapshot).
      - A pointer to the parent commit(s) (linking it into the history).
      - Author and committer information (name, email, timestamp).
      - The commit message you provided with the `-m` flag.
  4.  It updates the reference (usually a branch, like `main` or `master`) that HEAD is pointing to, making it point to this newly created commit object.
- **State Transition (Staged to "Saved" / Committed / Unmodified):**
  - Files that were in the `Staged` state are now part of this new commit.
  - Immediately after the commit, Git compares the versions of these files in your working directory to the versions just recorded in the new commit (which is now HEAD).
  - If the file in the working directory has _not_ been changed _since_ you staged it (`git add`), its state transitions from `Staged` -> `Unmodified` because the working directory version now matches the latest commit (HEAD).
  - If you _had_ modified the file _after_ staging but _before_ committing, the file will transition from `Staged` (as part of the commit) immediately into the `Modified` state relative to the new commit, because the working directory version is already different from what was just saved.
- **The `-m "message"` Flag:** The `-m` flag allows you to provide a short, descriptive message explaining _what_ changes are included in this commit. Writing clear, concise commit messages is crucial for understanding the project's evolution later. If you omit `-m`, Git will typically open a text editor for you to write a longer message.
- **Key Point:** `git commit` _only_ records the changes that were staged (`git add`-ed). Files that are untracked or modified but not staged are _not_ included in the commit.

This covers the basic lifecycle of files in Git and the two fundamental commands (`git add` and `git commit`) used to move them through these states and build your project history. Remember to use `git status` frequently to see the current state of your files!
