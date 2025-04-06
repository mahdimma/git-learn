## Understanding Git Objects

At its core, Git is a **content-addressable filesystem**. This means that instead of storing data based on file names or locations, Git stores data based on the *content* itself. The fundamental unit of this storage is called an **object**.

**Purpose:**
Git objects are used to store all the information necessary to represent your project's history, including file contents, directory structures, and historical snapshots (commits). Every piece of data in your Git repository is stored as one of these object types.

**Technical Details:**
* **Identification:** Each object is identified by a unique **SHA-1 hash** (a 40-character hexadecimal string). This hash is calculated based on the object's content plus a small header indicating its type.
* **Immutability:** Once an object is created in Git, it is **immutable**. Its content cannot be changed. If you change a file, Git doesn't modify the existing blob object; it creates a *new* blob object with a new hash representing the new content. This immutability is key to Git's data integrity and history tracking.
* **Storage:** Objects are typically stored in the `.git/objects/` directory within your repository. Initially, they might be stored as loose files (named after their SHA-1 hash, potentially split into a subdirectory based on the first two characters of the hash). Later, Git may pack these loose objects into more efficient "packfiles" (`.git/objects/pack/`) to save space and improve performance.
* **Header:** Before hashing, Git prepends a small header to the content. The header consists of the object type (`blob`, `tree`, `commit`, or `tag`), a space, the size of the content in bytes, and a null byte (`\0`). For example: `blob 12\0Hello World!`.

There are three main types of objects we'll discuss now: **blob**, **tree**, and **commit**. (There's also a fourth, `tag`, used for annotated tags, which we can cover later).

---

### 1. Blob Object

**Purpose:**
A **blob** (Binary Large Object) stores the raw content of a file. It holds *only* the file's data, without any metadata like the filename, path, or permissions.

**Why it's useful:**
By storing only the content, Git can efficiently deduplicate data. If multiple files in your repository (even with different names or in different directories) have the exact same content, Git stores only *one* blob object for that content and references it multiple times.

**Technical Details:**
* **Content:** The raw byte sequence of a file.
* **Hashing:** The SHA-1 hash is calculated based on the header (`blob <size>\0`) followed by the file's content.
* **Metadata:** Blobs do *not* contain filenames or directory information. This information is stored in `tree` objects.

**Use Cases & Examples:**
Imagine you have a file named `README.md` with the content "Hello Git!".

1.  **Creating a Blob (Low-level):** You can manually add content to Git's object database using `git hash-object`.
    ```bash
    # Create a file
    echo "Hello Git!" > temp_file.txt

    # Calculate the hash and store the blob object
    # The -w option writes the object to the database
    git hash-object -w temp_file.txt
    # Output: <sha1_hash_of_the_blob> e.g., d1b9e0b9..

    rm temp_file.txt # Clean up
    ```
2.  **Inspecting a Blob:** You can view the content of any Git object using `git cat-file`.
    ```bash
    # Use the hash obtained above
    git cat-file -p <sha1_hash_of_the_blob>
    # Output:
    # Hello Git!

    # See the type of the object
    git cat-file -t <sha1_hash_of_the_blob>
    # Output:
    # blob
    ```
    When you run `git add file.txt`, Git creates a blob object for the content of `file.txt` (if one doesn't already exist) and adds a reference to this blob in the staging area (index).

---

### 2. Tree Object

**Purpose:**
A **tree** object represents a directory structure within your project. It acts like a directory listing, mapping filenames to the corresponding blob objects (for files) or other tree objects (for subdirectories).

**Why it's useful:**
Trees allow Git to reconstruct the state of your project's working directory for any given snapshot (commit). They provide the structure and context (filenames, paths, permissions) that blobs lack.

**Technical Details:**
* **Content:** A list of entries, where each entry contains:
    * **Mode:** File permissions (e.g., `100644` for a regular file, `100755` for an executable file, `040000` for a subdirectory).
    * **Type:** The type of object being referenced (`blob` or `tree`).
    * **SHA-1 Hash:** The hash of the blob or tree object being referenced.
    * **Filename:** The name of the file or subdirectory within this tree.
* **Hashing:** The SHA-1 hash is calculated based on the header (`tree <size>\0`) followed by the concatenated binary representation of all its entries.
* **Structure:** Trees can point to other trees, creating a hierarchical structure that mirrors your project's directories.

**Use Cases & Examples:**
Imagine a project structure:
```
project/
├── README.md (content: "Project Intro")
└── src/
    └── main.py (content: "print('hello')")
```

1.  **Creating Blobs:** Git first creates blobs for `README.md` and `main.py`. Let's assume their hashes are `blob_readme_hash` and `blob_main_hash`.
2.  **Creating the `src` Tree:** A tree object for the `src` directory is created. It contains one entry:
    * `100644 blob <blob_main_hash> main.py`
    Let's call the hash of this tree `tree_src_hash`.
3.  **Creating the Root Tree:** A tree object for the root directory (`project/`) is created. It contains two entries:
    * `100644 blob <blob_readme_hash> README.md`
    * `040000 tree <tree_src_hash> src`
    Let's call the hash of this tree `tree_root_hash`.
4.  **Inspecting a Tree:**
    ```bash
    # Use the hash of the root tree
    git cat-file -p <tree_root_hash>
    # Output might look like:
    # 100644 blob <blob_readme_hash> README.md
    # 040000 tree <tree_src_hash>    src

    # Inspect the subtree
    git cat-file -p <tree_src_hash>
    # Output might look like:
    # 100644 blob <blob_main_hash> main.py
    ```
    The command `git write-tree` takes the current state of the staging area (index) and creates the necessary tree objects, returning the hash of the root tree. This is used internally by `git commit`.

---

### 3. Commit Object

**Purpose:**
A **commit** object represents a specific snapshot (a version) of your entire project at a particular point in time. It ties together the project's state (via a tree object) with metadata about *who* made the change, *when* it was made, and *why* (the commit message). Commits form the linked history of your project.

**Why it's useful:**
Commits are the milestones in your project's history. They allow you to:
* Track changes over time.
* Revert to previous states.
* Understand the evolution of the codebase.
* Collaborate by sharing sequences of commits.

**Technical Details:**
* **Content:** Contains metadata:
    * **Tree:** The SHA-1 hash of the top-level **tree** object representing the project's complete state at the time of the commit.
    * **Parent(s):** The SHA-1 hash(es) of the preceding commit(s).
        * Most commits have **one** parent.
        * The very first commit (root commit) has **no** parents.
        * A **merge commit** has **two or more** parents (one for each branch merged).
    * **Author:** The person who originally wrote the code changes (name, email, timestamp).
    * **Committer:** The person who actually created the commit (name, email, timestamp). Often the same as the author, but can differ (e.g., if someone applies a patch created by someone else, or during a `git rebase`).
    * **Commit Message:** The description explaining the changes made in this commit.
* **Hashing:** The SHA-1 hash is calculated based on the header (`commit <size>\0`) followed by the commit metadata content.

**Use Cases & Examples:**
Continuing the previous example, after creating the root tree (`tree_root_hash`):

1.  **Creating a Commit:** The `git commit` command creates a commit object.
    ```bash
    # Assume you've staged changes leading to tree_root_hash
    git commit -m "Initial project setup"
    # Output: [main (root-commit) <commit_hash>] Initial project setup...
    ```
2.  **Inspecting a Commit:**
    ```bash
    # Use the hash from the commit command
    git cat-file -p <commit_hash>
    # Output might look like:
    # tree <tree_root_hash>
    # author Your Name <your.email@example.com> 1678886400 +0100
    # committer Your Name <your.email@example.com> 1678886400 +0100
    #
    # Initial project setup

    # If it were a subsequent commit, it would also include:
    # parent <parent_commit_hash>
    ```

---

### How They Fit Together

The relationship between these objects forms the core of Git's repository structure:

* A **Commit** object points to a single top-level **Tree** object, representing the state of the project at that commit. It also points to its parent commit(s), forming the history chain.
* A **Tree** object points to **Blob** objects (representing files within that directory) and other **Tree** objects (representing subdirectories).
* A **Blob** object holds the actual file content.

```
+--------+      points to       +--------+      points to      +------+ (file content)
| Commit | ------------------>  | Tree   | -----------------> | Blob |
+--------+      (Project State) +--------+      (File Entry)   +------+
     |                           |     ^
     |                           |     |
(Parent)                         |     +------ (Subdir Entry)
     |                           |
     v                           v
+--------+                   +--------+      points to      +------+ (file content)
| Commit |                   | Tree   | -----------------> | Blob |
+--------+                   +--------+      (File Entry)   +------+
 (Previous                   (Subdirectory)
  Snapshot)
```

---

### Summary & Key Takeaways

* Git stores everything as **objects**: blobs, trees, and commits (and tags).
* Objects are identified by **SHA-1 hashes** based on their content + header.
* Objects are **immutable**.
* **Blobs** store file content (data only).
* **Trees** store directory structure (filenames, modes, references to blobs/trees).
* **Commits** store snapshots of the project (reference to a root tree) along with metadata (parents, author, committer, message), forming the project history.
* This object model makes Git efficient (deduplication via blobs), robust (immutability ensures integrity), and powerful for tracking history and structure.

Understanding these fundamental objects is crucial for grasping how Git works under the hood and for using its more advanced features effectively. Let me know what Git topic you'd like to explore next!