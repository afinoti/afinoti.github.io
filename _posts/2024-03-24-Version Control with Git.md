---
title: Version Control with Git
author: afinoti
date: 2024-03-24 09:38:00 +0800
categories: [Blogging, References]
tags: [software, versioning, git]
pin: true

---

### Resources

- [**Julia Evans**](https://jvns.ca/blog/2023/11/06/rebasing-what-can-go-wrong-/?utm_source=tldrwebdev) - :us: "git rebase: what can go wrong?" explain how to solve most common problems when rebasing.

- [**LearnThatStack**](https://www.youtube.com/watch?v=Ala6PHlYjmw) - :us: "Git Will Finally Make Sense After This" explain basics of git commands.

---

### 1. The Mental Model: Git is a Graph

To truly understand Git, you must stop thinking of it as a file system and start thinking of it as a **Directed Acyclic Graph (DAG)**. The commands `add`, `commit`, and `checkout` are simply tools to manipulate this graph.

#### Core Concepts
1.  **Nodes are Commits:** Every commit is a node in the graph. It contains a full snapshot of the project (not just diffs), metadata (author, date), and a pointer to the "parent" commit.
2.  **Branches are Pointers:** A branch (like `main` or `feature`) is not a container for files. It is a lightweight, movable sticky note that points to a specific commit hash.
3.  **HEAD is "You Are Here":** HEAD is a special pointer that indicates what commit you are currently looking at. Usually, HEAD points to a Branch, which points to a Commit.

#### The Three Zones
Data moves through three specific areas before becoming permanent:
1.  **Working Directory:** Where you edit files (untracked).
2.  **Staging Area (Index):** Where you prepare the "package" (`git add`).
3.  **Repository:** The database of committed history (`git commit`).

---

### 2. Visualizing `git reset`

Many developers fear `git reset`, but it follows the simple logic of the graph: **it just moves the branch pointer backward.** The only difference between the "flavors" of reset is what happens to your work after the pointer moves.

> **Scenario:** You are on branch `main` at commit `c3`. You run `git reset HEAD~2` to go back to `c1`.

* **`--soft` (The Safe Option)**
    * **Visual:** Moves the branch pointer back to `c1`.
    * **Files:** Changes from `c2` and `c3` are placed in the **Staging Area**.
    * **Use case:** You want to squash the last few commits into one, or you forgot to add a file to the last commit.

* **`--mixed` (The Default)**
    * **Visual:** Moves the branch pointer back to `c1`.
    * **Files:** Changes from `c2` and `c3` are placed in the **Working Directory** (unstaged).
    * **Use case:** You want to keep your work but rethink how you group your commits.

* **`--hard` (The Destructive Option)**
    * **Visual:** Moves the branch pointer back to `c1`.
    * **Files:** **Deleted.** The Working Directory is overwritten to match `c1` exactly.
    * **Use case:** You want to scrap everything you did recently and start over.

---

### 3. The Golden Rule of Rebase

While `git merge` preserves history, `git rebase` rewrites it to create a clean, straight line. Because it changes commit Hashes, it comes with a critical warning:

> **WARNING:** Never rebase commits you have already pushed.

**Why?**
If you rebase commits that exist on a remote server (like GitHub), you change the history that other developers have already downloaded. When they try to sync their work, their history will conflict with your "new" history, causing massive merge conflicts for everyone.

**Rule of Thumb:** Only rebase local, unpushed commits to clean up your own messy history before sharing it.

---

### 4. What Each Command Really Does in the Graph

#### 1. `git checkout` → Moves HEAD

Checkout doesn't change history.

It simply says: "I want to look at this other commit."

HEAD moves. The branch continues pointing to the same place.

**Use when:**
You want to navigate history, test something, or create a branch from an old commit.

**Mental model:**
📍 You move the "where I'm looking" marker — nothing in the graph changes.

#### 2. `git reset` → Moves the Branch

Reset rewrites the branch pointer.

It says: "This branch should no longer point here; now it should point there."

The effects on files depend on whether it was `--soft`, `--mixed`, or `--hard`.

**Use when:**
You want to rewrite your own local history before pushing.

**Mental model:**
🔁 You take the branch sticky note and paste it on another commit.

#### 3. `git revert` → Creates a New Commit

Unlike reset, revert doesn't erase history.

It creates a commit that undoes the effect of another commit.

It doesn't move HEAD backward; it adds a new commit.

**Use when:**
You've already pushed to the team and need to undo something without breaking shared history.

**Mental model:**
➕ You create a new node in the graph that "cancels out" another node.

#### Quick Summary

| Command | What it changes? | Safety | Ideal use |
|---------|------------------|--------|----------|
| `checkout` | Moves HEAD | Safe | Navigate history, switch branches |
| `reset` | Moves the branch | Dangerous (if already pushed) | Rewrite local commits |
| `revert` | Adds new commit | Safe | Undo already shared commits |