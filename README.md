# Guide-Tracking-Changes-By-An-Author-In-A-Git-Branch

To streamline the process of identifying all the files modified by a specific author in a branch compared to its base branch (e.g., develop-master-demo) and generating their difference files along with their current versions, follow these steps within the project directory

1. Ensure the branch you want to work with is synced with its base branch.

2. Switch to the desired branch using the command:

   ```bash
   git checkout <branch_name>
   ```

   For example:

   ```bash
   git checkout feature/SFD-Team/zd-InterFax
   ```

3. Generate a list of files modified by a specific author using Git log, for example, author "Kamyar mofakhami":
   ```bash
    git log --author="<author>" --pretty=format: --name-only > temp.txt
    ```
   ```bash
    git log --author="Kamyar mofakhami" --pretty=format: --name-only > temp.txt
    ```
   4.Sort and distinct the files by

### Windows (CMD):

```cmd
type temp.txt | sort /uniq > uniquefiles.txt
```

### Windows (PowerShell):

```powershell
Get-Content temp.txt | Sort-Object -Unique > uniquefiles.txt
```

### Linux/Mac:

```bash
sort -u temp.txt > uniquefiles.txt
```

5. Remove empty lines from `uniquefiles.txt`, and exclude specific file types like "RolloutScripts.sql". Append these excluded files to the end of your migration files list.

6. Create and save the following script in the project directory:

 ### Windows (PowerShell):

 ```powershell
   # Read the file names from "uniquefiles.txt"
   Get-Content "uniquefiles.txt" | ForEach-Object {
     # Replace slashes with underscores in the file name
     $safe_filename = $_ -replace '/', '_'

     # Capture the output of the git log command in a variable
     $output = git log --color=never -p -- $_

     # Check if the output is empty
     if (![string]::IsNullOrEmpty($output)) {
       # If not empty, write it to a file
       $output | Out-File "${safe_filename}_changes.diff"
     }

     # Copy the current version of the file to a new location
     Copy-Item $_ "${safe_filename}_current.cs"
   }
  ```

### Windows (CMD):

```cmd
@echo off
setlocal enabledelayedexpansion

rem Read the file names from "uniquefiles.txt"
for /f "delims=" %%a in (uniquefiles.txt) do (
  rem Replace slashes with underscores in the file name
  set "file=%%a"
  set "safe_filename=!file:/=_!"

  rem Capture the output of the git log command in a variable
  for /f "delims=" %%b in ('git log --color=never -p -- "!file!"') do (
    set "output=!output!%%b"
  )

  rem Check if the output is not empty
  if defined output (
    rem If not empty, write it to a file
    echo !output! > "!safe_filename!_changes.diff"
  )

  rem Copy the current version of the file to a new location
  copy "!file!" "!safe_filename!_current.cs"
)
```

### Linux/Mac:

```bash
#!/bin/bash

# Read the file names from "uniquefiles.txt"
while IFS= read -r file; do
  # Replace slashes with underscores in the file name
  safe_filename=$(echo "$file" | tr '/' '_')

  # Capture the output of the git log command in a variable
  output=$(git log --color=never -p -- "$file")

  # Check if the output is not empty
  if [ -n "$output" ]; then
    # If not empty, write it to a file
    echo "$output" > "${safe_filename}_changes.diff"
  fi

  # Copy the current version of the file to a new location
  cp "$file" "${safe_filename}_current.cs"
done < "uniquefiles.txt"
```

7. Run the script using:

### Windows (PowerShell):

```powershell
   powershell ./uniquefiles.ps1
 ```

### Windows (CMD):

 ```
   To execute PowerShell scripts, it's best to use the PowerShell environment (powershell.exe) and run the script using the appropriate PowerShell execution commands
  ```

 ### Linux/Mac:

 ```bash
   chmod +x uniquefiles.sh
   ./uniquefiles.sh
  ```

  Make sure to navigate to the project directory using `cd` before running the script.

8. After execution, you'll find all modified files along with their diff files and current versions in the project directory. These files will be named according to their paths and can be opened in VS Code for reviewing changes, including author information in the diff files.

These generated files can be helpful for tracking changes while migrating files to the another branch, ensuring you keep track of all modifications made by specific authors during the migration process. Adjust the `<branch_name>` and `<author_name>` placeholders as needed for your specific use case.
