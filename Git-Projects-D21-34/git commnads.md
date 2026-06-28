
### git log
displays a detailed, chronological history of all the commits made in a Git repository. It serves as an audit trail showing exactly how a project evolved over time.
- displays Commit Author,Message,hash,date
 ```
 >git log

 # for oneline
 > git log --oneline
```
##### View Actual Code Changes
```
 git log -p
```
##### Limit the number of shown commit
`git log -n 3`

##### Filter by Author
` git log --author="John Doe" `

##### View as a Visual Branch Tree
```
 git log --oneline --graph --all
```
#### merge to main branch 
1. To merge a branch into main, you must first stand on the main branch and then pull the changes into it
 > git switch main or git checkout main
2. Ensure your local main branch has the absolute latest changes from your remote server (like GitHub or GitLab) to avoid outdated conflicts
> git pull origin main
3. Merge the Feature Branch
> git merge main


4. Deal with the Output
   Depending on how your code looks, one of two things will happen:
   - Fast-Forward / Automatic Merge: If there are no conflicting changes, Git will combine the files instantly. A text editor might pop up asking for a commit message; you can just save and close it.
   - Merge Conflicts: If the same line of code was changed on both branches, Git will pause and ask you to fix it. Open the affected files, choose the code you want to keep, remove the Git markers (<<<<<<<, =======, >>>>>>>), save the files, and run:
  ```
   git add .
   git commit -m "Fix merge conflicts"
  ```
 5. Push Changes to Remote
  Once the merge is complete locally, upload the updated main branch to your remote server:
  ```
   git push origin main
  ```
##### To Check remote repo
 ```
  git remote
 ``` 
---
#### What HEAD Points to by Default
Normally, HEAD does not point directly to a commit. Instead, it points to your current branch, and that branch points to the latest commit.
> git log
```
[Commit 1] <--- [Commit 2] <--- [Main Branch] <--- [HEAD]

First time : commit c3b2a1... (HEAD -> main, origin/main)
```
This tells you that your working directory matches the main branch, which is currently on commit c3b2a1
#### Moving the HEAD Pointer
Whenever you switch tasks, you are moving the HEAD pointer:
- Switching Branches: Running` git checkout feature-xyz` or `git switch feature-xyz` moves `HEAD` to point at the `feature-xyz` branch.
- Making a New Commit: When you run `git commit`, Git creates a new commit, moves your current branch forward to it, and `HEAD` follows along automatically

#### What is a "Detached HEAD"?
If you checkout a specific `commit hash` instead of a `branch name` (for example:` git checkout c3b2a1`), you enter a state called a Detached HEAD
```
     [HEAD] (Points directly to Commit 2, not a branch)
                    │
                    ▼
 [Commit 1] <--- [Commit 2] <--- [Main Branch]
```
  - What it means: You are looking at a snapshot of the past.
  - The Risk: If you make new commits while in a detached HEAD state, they will not belong to any branch. If you switch away to another branch, those new commits can easily become lost

  - How to check where HEAD is pointing right now
  - `cat .git/HEAD`
     ##### Fix Detached HEAD
    - `git switch main' or
    - ` git checkout main`   
     Or Keep the changes you made while detached
      `git switch -c my-saved-work` This instantly creates a new branch named my-saved-work and attaches your HEAD to it

      To make changes to main
      `git switch main`
      `git merge my-saved-work`

---
#### Git Remote
```
  #git remote add [<options>] <name> <url>
  # check git remote
     > git remote
  # add a git remote
    > git remote add dev_news /opt/xfusioncorp_news.git
```
#### Managing two Remotes
Let's assume you ran `git remote -v` and your two remotes are named origin (e.g., `GitHub`) and backup (e.g.,`GitLab`), and you are currently working on the  `main` branch.

`git push <remote-name> <branch-name>`
```
  1.  Push to your first remote (origin):
        `git push origin main`
  2.  Push to your second remote
     `git push backup main`
```
---

####   Git Revert
 It is used to undo an old commit by creating a brand-new commit that does the exact opposite of the target commit  
 It is the safest way to undo errors on shared or public repositories because it does not rewrite history. It leaves the old mistake in the log but safely cancels out its changes.  
 - To revert a specific commit
   ` git revert <commit-hash>`
 1. Run git log --oneline to find the bad commit hash (e.g., a1b2c3d)
 2. Run git revert a1b2c3d.
 3. Git will automatically open a text editor asking for a commit message
 4. Save and close the editor. Git will create a new commit named` "Revert 'Add broken feature'"`
 #####  Revert Without Instantly Committing
  If you want to reverse the changes but don't want Git to automatically create the new commit yet (allowing you to inspect or modify the files first), use the  
  `-n (no-commit) flag`:  
  `git revert -n <commit-hash>`  # hash of commit you want to remove , not the hash which yo want to reach
 ##### Revert the Absolute Last Commit
  `git revert HEAD`

 
| Command | How it Works | Best Used For |
| :--- | :--- | :--- |
| **`git revert`** | Creates a new commit that reverses changes. Past history is untouched. | Public / Shared branches (so you don't mess up your teammates' histories). |
| **`git reset`** | Erases past commits from the history entirely, as if they never happened. | Private branches where no one else has downloaded your code yet. |
| **`git stash`** | Temporarily shelves uncommitted changes so you can work on a clean directory. | Quickly switching branches without committing half-done work. |


---
**Git Stash**
git stash is a command that temporarily shelves (saves) uncommitted changes in your working directory so you can work on something else, and then come back and reapply them later

It is ideal when you are in the middle of a code change but need to urgently switch branches to fix a bug, without creating a messy, half-finished "work in progress" commit.  

* 1. Check your messy status
    ```
     git status
     # Output: Modified: feature.py
    ```

* 2. Stash your work safely
   Cut and paste your current changes into Git's hidden storage area. Your working directory becomes completely clean.
   ```
     git stash -m "working on new feature"
   ```
* 3. Do your urgent work
     Now you can safely switch branches, fix the bug, commit it, and push it
     ```
      git switch main
      # ... fix bug ...
      git commit -am "fix critical bug"
      git switch feature-branch
* 4. Bring your stashed work back
     Now that you are back on your feature branch, retrieve your saved work from the storage area:
     ```
      git stash pop
     ```
     * Result: feature.py is restored exactly where you left off, and the stash is deleted from storage.
     Command:
     * `git stash list`: Shows all your saved stashes with their IDs (e.g., stash@{0}).
     * `git stash apply`: Reapplies the stashed changes but keeps a copy saved in the stash list (unlike pop, which deletes it).
     * `git stash drop`: Permanently deletes a specific stash from your list.
     * `git stash clear`: Deletes all saved stashes at once.
   #### Example : to bring stash stash1 ,
  To bring back a specific stash by its name, you cannot use the text name directly (like "stash1"). Instead, Git identifies stashes using their index numbers, which look like stash@{0}, stash@{1}, etc
  ```
   git stash list

   stash@{0}: On master: temporary fix
   stash@{1}: On feature: stash1
   stash@{2}: On main: old work
  ```

  Run : `git stash pop stash@{1}`
  or  : `git stash apply stash@{1}'
     ---
     ### Git Merge
     If you stand on the master branch and run git merge feature, Git will perform a Fast-Forward merge.Because master is a direct ancestor of feature (no other commits were made on master since they split), Git does not need to combine any conflicting code. It will simply slide the master pointer forward to match feature.What Your Log Will Look Like After the MergeBoth branches will point to the exact same commit hash:texte56125e (HEAD -> master, origin/feature, feature) Update welcome.txt
a7196e2 Update info.txt
30be729 (origin/master) Add welcome.txt
7a98941 initial commit
Critical Step Before Running the CommandYou must ensure you are standing on the receiving branch (master) first. If you run the command right now while HEAD is on feature, Git will say Already up to date.Run these exact commands to complete the merge correctly:bash# 1. Switch over to the master branch
git switch master

# 2. Pull the feature branch's changes into master
git merge feature

# 3. Update the remote server (optional)
git push origin master
Did you want to keep a clean linear history using this default fast-forward, or do you want to force Git to create a merge commit to show where the branch joined?You said: and what if master has a commit toIf master has its own new commit too, your repository history splits into two separate paths.Because both branches have moved forward independently, Git can no longer do a simple "Fast-Forward" merge. Instead, when you run git merge feature from the master branch, Git will perform a Three-Way Merge.What Git Does Behind the ScenesIt looks at the common ancestor commit (30be729).It looks at the changes made on feature.It looks at the changes made on master.It combines all the changes together into a brand-new Merge Commit.

The Two Possible OutcomesDepending on whether you edited the same lines of code, one of two things will happen:

**Outcome A**: Automatic Merge (No Conflicts)If you edited different files (or different parts of the same file), Git will automatically combine everything.A terminal text editor (like Nano or Vim) will pop up asking for a commit message.You can save and close it. Git will create a new merge commit.Your log will now look like a loop:text*   7b8c9d0 (HEAD -> master) Merge branch 'feature'  <-- New Merge Commit
```text
|\  
| * e56125e (origin/feature, feature) Update welcome.txt
| * a7196e2 Update info.txt
* | f23a4b1 (master) [The new commit you made on master]
|/  
* 30be729 (origin/master) Add welcome.txt
* 7a98941 initial commit
```

Outcome B: Merge ConflictIf you edited the exact same line of code in the same file on both master and feature, 
Git will pause and print a warning:CONFLICT (content): Merge conflict in [filename]
- Git will modify the conflicting files and insert markers (<<<<<<<, =======, >>>>>>>).
- You must open the files, choose which code to keep, delete the markers, and then run:bashgit add .
  ```
   git commit -m "Resolve merge conflict"
  ```
---

## ** Git Cherry Picking**

Cherry-picking in Git means choosing a single specific commit from one branch and applying it to another branch  
Instead of merging an entire branch (which pulls in dozens of changes), cherry-picking lets you pluck out just one isolated fix or feature and copy it exactly where you need it.  

##### Real-World Example
Imagine your repository looks like this:
- `main`: The stable version of your app currently running in production.
- `feature`: A branch where you are building a massive new login system.

 While working on the feature branch, you notice and fix a critical security bug in commit c7a1b23. The rest of the login system isn't finished yet, but you need that specific security fix live on main immediately

 ```text
                    a7196e2 ── c7a1b23 (Security Fix) ── e56125e [feature]
                 /
 initial ── 30be729 [main]
 ```
* Step 1: Find the Commit Hash
  Run `git log --oneline` feature to locate the exact hash of the fix. In this case, it is `c7a1b23`
*  Step 2: Switch to the Target Branch
   ```
    git switch mainStep 
   ```
   
* 3: Run the Cherry-Pick Command
  ```
    git cherry-pick c7a1b23
  ```
  ** Result**
    Git copies the exact changes from that single commit and creates a brand-new commit on main. The unfinished login system stays safely behind on the feature branch.
  ```text
       a7196e2 ── c7a1b23 ── e56125e [feature]
                 /
   initial ── 30be729 ── d4e5f67 (Security Fix - Copied!) [main]
  ```
  
---
## Git Reset 
It allows you to move your repository's timeline backward to an older commit. The "tough" part is just deciding what happens to your current, uncommitted code when you travel back in time.

There are three main modes of 
- `git reset: --soft`,
- `--mixed (the default)`, and
- `--hard`.
Let’s look at them through a clear example

##### Example Setup
Imagine your project folder currently has 3 commits in its history:
- Commit 1: Added index.html
- Commit 2: Added styles.css
- Commit 3 (HEAD): Added app.js (This is where you are standing right now)
  ** You realize Commit 3 was a mistake. You want to use git reset to go back to Commit 2**
* 1. `git reset --soft` (The "Keep Everything" Mode)
     This moves your repository history back to Commit 2, but leaves your files completely untouched
     ```
      git reset --soft <Commit-2-Hash>
     ```
     - **What happens to the code?:` app.js` is not deleted.**
     - Where does it go?: It stays in your `Staging Area `(green status). It is` ready to be committed again` immediately.
     - Best used when: You committed too early, or you want to combine (squash) multiple recent commits into one clean commit
* 2. `git reset --mixed` (The **Default** "Safety Net" Mode)
   If you type git reset without adding a flag, Git automatically uses --mixed. This moves history back to Commit 2 and unstages your files
   `git reset <Commit-2-Hash>`
  - Where does 'app.js' go?: It is moved out of the staging area into your `Working Directory (red status, unstaged)`.
  - Best used when: You added files using git add . by accident and want to unstage them, or you want to rework the code before committing again
* 3. `git reset --hard` (The "Nuke It" Mode)
     This is the dangerous one. It moves history back to Commit 2 and completely wipes out all changes made after that point.
     ```
      git reset --hard <Commit-2-Hash>
     ```
    - Where does it go?: It is gone forever. Your folder looks exactly like it did when you finished Commi
    - Best used when: Your recent work went completely wrong, you want to throw it all away, and start fresh from an older checkpoint.
#### Example 2 , git local commit is at commit 5, i force to commit 2 using hard rest
  > git reset --hard <commit-2>
Now the git reomte is at commit 5 ,so git oush failed ,
> To force commit

> git push --force-with-lease origin master
  
| Mode | Moves History? | Keeps Changes in Files? | Where do changes go? |
| :--- | :--- | :--- | :--- |
| **`--soft`** | **Yes** | **Yes** | Staging Area (Ready to commit) |
| **`--mixed`** (Default) | **Yes** | **Yes** | Working Directory (Unstaged) |
| **`--hard`** | **Yes** | **No** | Destroyed permanently |
---
#### Git Rebase
Git rebase is a command that moves or combines a sequence of commits onto a new base commit. Its primary purpose is to maintain a clean, linear project history by rewriting commit history.
Its primary purpose is to maintain a clean, linear project history by rewriting commit history

Instead of creating a "merge commit" when combining branches, rebasing takes the unique commits from your current branch, temporarily sets them aside, applies the latest updates from the target branch, and then replays your commits one by one on top.
##### Merge vs. Rebase
**Starting Point:**
```
       A---B (main)
           \
            C---D (feature)
```
###### Option A: Git Merge
 If you run git merge main while on your feature branch, Git creates a new "Merge Commit" (E) to tie the two histories together. This creates a fork-and-join look in your logs.
 ```
       A---B------- (main)
           \       \
            C---D---E (feature)
```
###### Option B: Git Rebase
If you run git rebase main while on your feature branch, Git lifts commits C and D, updates your branch base to B, and replays your work. This results in a perfectly straight, linear line.text
```
       A---B (main)
           \
            C'---D' (feature)
```
Note: C' and D' are brand new commits with new cryptographic hashes because their parent commit changed from A to B

**Example**
- 1. You are working on your feature branchYou have made two commits, but your team lead says main has critical updates you need.
   ```
    git checkout feature-login
   ```
- 2. Fetch the latest updates from the remote serverGet the fresh code from GitHub/GitLab without touching your local files yet.
   ```
    git fetch origin
   ```
- 3.  Run the rebaseTell Git to take your login feature commits and place them on top of the updated origin/main branch.
  ```
   git rebase origin/main
  ```
- 4. Handle conflicts (if any)
  If you modified the same lines as someone else, Git will pause. Open the files, fix the conflicts, and then type:
 ```
 git add .
 git rebase --continue
 ```

---
#### More Explained
* 1. The SituationYou started your feature branch from Commit 2. While you were working, a teammate added Commit 3 to the main branch.
     ```
      [Main Branch]          ─── Commit 1 ─── Commit 2 ─── Commit 3
                                       │
      [Feature Branch]                 └─── Commit A ─── Commit B
     ```
* 2. The "Git Merge" Approach
     If you choose to merge, Git leaves your history exactly where it is. It creates a brand-new Merge Commit (M) at the end to tie the two branches together. Your history now has a fork and a loop.
     ```
       [Main Branch] ─── Commit 1 ─── Commit 2 ─── Commit 3 ────┐
                                         │                      ▼
       [Feature Branch]                  └─── Commit A ─── Commit B ─── [Merge Commit M]
     ```
     * The Common Ancestor (Commit 2): The last point where both branches agreed.
     * The changes on your Feature Branch (Commit A and Commit B).
     * The changes on the Main Branch (Commit 3).
     it looks at these three points to intelligently combine the code. If you and your teammate edited different files or different lines, Git stitches them together automatically.
     This will be '[Merge Commit M]'
* 3. The "Git Rebase" ApproachIf you choose to rebase, Git temporarily lifts Commit A and Commit B out of the way. It extends the main branch line to include Commit 3, and then replays your commits right on top of it.
     ```
      [Main Branch] ─── Commit 1 ─── Commit 2 ─── Commit 3 
                                                   │
      [Feature Branch]                             └─── Commit A' ─── Commit B'
     ```
     clubs commit 3 and then all feature commits are append at the end
     Because the base of your feature branch moved from Commit 2 to Commit 3, it results in a perfectly straight, **linear timeline**:
     ```
      ─── Commit 1 ─── Commit 2 ─── Commit 3 ─── Commit A' ─── Commit B'
     ```
---
## git fetch
Think of git fetch as checking the mailbox. It downloads the latest commit history, new branches, and files from the remote server (like GitHub) and stores them in a hidden tracking area inside your .git folder

It updates your remote-tracking branches (like origin/main), but it does not touch your actual local branches (like main) or the code you are currently editing.
#### Fetch vs. Pull
- `git fetch`: Downloads data. Your files stay exactly the same
  ```text
   Remote Repo ────────> Remote-Tracking Branch (origin/main)
                         [ STOP HERE: Safe to inspect ]
  ```
- `git pull`: Downloads data AND immediately merges it into your code
  ```text
   Remote Repo ────────> Remote-Tracking Branch ────────> Your Local Files

  
                         [ Fetching ]                   [ Merging ]
  ```

 **Why use git fetch?**
  Because it is safe, you use it to see what your team has done before you decide to mix their code with yours
  - After running git fetch, you can safely view the remote changes using:
  ```bash
   # See what files changed on the remote server
   git diff main origin/main --name-only

   # See the commit messages your teammates pushed
   git log main..origin/main --oneline
  ```
 ## match the local to remote
  #### `git fetch origin master`
  #### `git diff master origin/master`
  
 ## Merge Conflict Example 
 - I tried to mmerge story-blog file , but the remote repo already has some line which and not match
 - - Moose (local) -- Mouse(remote)
   - And addtional line in local `The Donkey and the Dog`
   ```text
     <<<<<<< HEAD
    1. The Lion and the Mooose
    2. The Frogs and the Ox
    3. The Fox and the Grapes
    4. The Donkey and the Dog
    =======
    1. The Lion and the Mouse
    2. The Frogs and the Ox
    3. The Fox and the Grapes
     >>>>>>> origin/master
   ```
   * HEAD : represent local repo
   * origin/master represent the remote repo
   **Edit the File ** : as below and save and push
     ```
      1. The Lion and the Mouse
      2. The Frogs and the Ox
      3. The Fox and the Grapes
      4. The Donkey and the Dog
     ```
---
## Git Hook
Git hooks are custom scripts that Git runs automatically before or after specific actions occur (like committing, pushing, or merging).

Think of them as automated quality checkpoints. They let you enforce rules, run tests, or format code automatically before anything gets saved or sent to the server. If a script fails, Git cancels the operation.

#### How They Work Internally
Every Git repository has a hidden folder called `.git/hooks/`.

f you look inside that folder on your computer right now, you will see a list of example files like 
`pre-commit.sample`, `pre-push.sample`, and `post-merge.sample`. To activate any hook, you simply remove the .sample extension from the filename and write your script inside it.

##### Real-World Example: The pre-commit Hook
A pre-commit hook runs the exact moment you type git commit, but before Git actually creates the commit. This is the perfect place to block bad code from being committed.

 Let's say you want to prevent anyone (including yourself) from accidentally committing temporary debugging notes like "TODO: FIX THIS" or "FIXME" into your story-blog project.
 * Step 1: Create the Hook File:
   ` touch .git/hooks/pre-commit
`
* Step 2: Add the Script
  ```
   #!/bin/bash

  # Search staged files for the word "FIXME"
  if git diff --cached | grep -q "FIXME"; then
      echo "❌ ERROR: You left a 'FIXME' comment in your code!"
      echo "Please remove it before committing."
      exit 1 # Returning a non-zero number forces Git to cancel the commit
  fi

  exit 0 # Returning 0 tells Git everything is fine; proceed with commit
```
   
 * Step 3: Make it Executable
 ```
  chmod +x .git/hooks/pre-commit
```
You edit story-index.txt and accidentally add:
The Lazy Dog FIXMEYou run: git add story-index.txtYou run: git commit -m "Add fifth story"

`❌ ERROR: You left a 'FIXME' comment in your code!`
```
##### Excercise 2 :
Create a post-update hook in this git repository so that whenever any changes are pushed to the master branch, it creates a release tag with name `release-2023-06-15`, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20
- Got to:
  ```
  cd /usr/src/kodekloudrepos/apps/.git/hooks
  ```
- Create or use sample file 'vi post-update'
-  Add the Hook Script Content
```
 #!/bin/sh

 # Get the current date in YYYY-MM-DD format
 CURRENT_DATE=$(date +%Y-%m-%d)

 # Create the release tag
 git tag "release-$CURRENT_DATE"
```
- Make the Hook Executable
  `chmod +x post-update`

  Move the hook to the remote repository's hooks folder
  ```
   sudo mv /usr/src/kodekloudrepos/ecommerce/.git/hooks/post-update /opt/ecommerce.git/hooks/
  # since this repo will trigger hook
  # chmod +x post-update
   
- Push the change to Remote
  ```
    git checkout master
    git merge feature

    ## push this will trigger hook
    git push origin master
  ```
  - Check tag
    ```
     git tag
    ```
    
