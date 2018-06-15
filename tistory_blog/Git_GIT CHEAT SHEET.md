### Git is the open source distributed version control system that facilitates GitHub activities on your laptop or desktop.  
### This cheat sheet summarizes commonly used Git command line instructions for quick reference.

## INSTALL GIT
GitHub provides desktop clients that include a graphical user interface for the most common repository actions and an automatically updating command line edition of Git for advanced scenarios.

### GitHub for Windows
https://windows.github.com

### GitHub for Mac
https://mac.github.com

Git distributions for Linux and POSIX systems are available on the official Git SCM web site.

### Git for All Platforms
http://git-scm.com


## CONFIGURE TOOLING
Configure user information for all local repositories

- Sets the name you want attached to your commit transactions
    ```bash
    $ git config --global user.name "[name]"
    ```
- Sets the email you want attached to your commit transactions
    ```bash
    $ git config --global user.email "[email address]"
    ```
- Enables helpful colorization of command line output
    ```bash
    $ git config --global color.ui auto
    ```

## CREATE REPOSITORIES
Start a new repository or obtain one from an existing URL
- Create a new local repository with the specified name
    ```bash
    $ git init [project-name]
    ```
- Downloads a project and its entire version history
    ```bash
    $ git clone [url]
    ```

## MAKE CHANGES
Review edits and craft a commit transaction

- Lists all new or modified files to be committed
    ```bash
    $ git status
    ```
- Shows file differences not yet staged
    ```bash
    $ git diff    
    ```
- Snapshots the file in preparation for versioning
    ```bash
    $ git add [file]
    ```
- Shows file differences between staging and the last file version
    ```bash
    $ git diff --staged
    ```
- Unstages the file, but preserve its contents
    ```bash
    $ git reset [file]
    ```
- Records file snapshots permanently in version history
    ```bash
    $ git commit -m "[descriptive message]"    
    ```

## GROUP CHANGES
Name a series of commits and combine completedefforts

- Lists all local branches in the current repository
    ```bash
    $ git branch
    ```
- Creates a new branch
    ```bash
    $ git branch [branch-name]
    ```
- Switches to the specified branch and updates the working directory
    ```bash
    $ git checkout [branch-name]
    ```
- Combines the specified branch's history into the current branch
    ```bash
    $ git merge [branch]
    ```
- Deletes the specified branch
    ```bash
    $ git branch -d [branch-name]
    ```

## REFACTOR FILENAMES
Relocate and remove versioned files

- Deletes the file from the working directory and stages the deletion
    ```bash
    $ git rm [file]
    ```
- Removes the file from version control but preserves the file locally
    ```bash
    $ git rm --cached [file]
    ```
- Changes the file name and prepares it for commit
    ```bash
    $ git mv [file-original] [file-remaned]
    ```

## SUPPRESS TRACKING
Exclude temporary files and paths

- A text file named `.gitignore` suppresses accidental versioning of files and paths matching the specified patterns
    ```bash
    *.log
    build/
    temp-*
    ```
- Lists all ignored files in this project
    ```bash
    $ git ls-files --other --ignored --exclude-standard
    ```

## SAVE FRAGMENTS
Shelve and restore incomplete changes

- Temporarily stores all modified tracked files
    ```bash
    $ git stash
    ```
- Restores the most recently stashed files
    ```bash
    $ git stash pop
    ```
- Lists all stashed changesets
    ```bash
    $ git stash list
    ```
- Discards the most recently stashed changeset
    ```bash
    $ git stash drop
    ```

## REVIES HISTORY
Browse and inspect the evolution of project files

- Lists version history for the current branch
    ```bash
    $ git log
    ```
- Lists version history for a file, including renames
    ```bash
    $ git log --follow [file]
    ```
- Shows content differences between two branches
    ```bash
    $ git diff [first-branch]...[second-branch]
    ```
- Outputs metadata and content changes of the specified commit
    ```bash
    $ git show [commit]
    ```

## REDO COMMITS
Erase mistakes and craft replacement history
- Undoes all commits after `[commit]`, preserving changes locally
    ```bash
    $ git reset [commit]
    ```
- Discards all history and changes back to the specified commit
    ```bash
    $ git reset --hard [commit]
    ```

## SYNCHRONIZE CHANGES
Register a repository bookmark and exchange version history
- Downloads all history from the repository bookmark
    ```bash
    $ git fetch [bookmark]
    ```
- Combines bookmark's branch into current local branch
    ```bash
    $ git merge [bookmark]/[branch]
    ```
- Uploads all local branch commits to GitHub
    ```bash
    $ git push [aliah] [branch]
    ```
- Downloads bookmark history and incorporates changes
    ```bash
    $ git pull
    ```