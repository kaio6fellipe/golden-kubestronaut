# Essential Commands

> Reference: [Kodekloud public notes](https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Introduction/Course-Introduction)

## Login and Console Access

### Local Console Login

**Virtual Terminals (TTY):**
```bash
Ctrl + Alt + F1-F6           # Switch to virtual terminals (TTY1-TTY6)
Ctrl + Alt + F7              # Return to graphical interface (if available)
tty                          # Display current terminal
who                          # Show logged-in users
w                            # Show who is logged in and what they're doing
last                         # Show login history
```

**Text Mode vs Graphical Mode:**
- **Text Mode**: Direct console login, shows username/password prompt
- **Graphical Mode**: Display manager (GDM, LightDM, SDDM) with GUI login

### Remote Console Access

**SSH (Secure Shell):**
```bash
ssh user@hostname                    # Basic SSH connection
ssh -p 2222 user@hostname            # Connect to non-standard port
ssh -X user@hostname                 # X11 forwarding for GUI apps
ssh -i ~/.ssh/key user@hostname      # Use specific SSH key
ssh -v user@hostname                 # Verbose mode for debugging

# Generate SSH keys
ssh-keygen -t rsa -b 4096            # Generate RSA key pair
ssh-keygen -t ed25519                # Generate Ed25519 key (modern)
ssh-copy-id user@hostname            # Copy public key to remote server

# SSH config (~/.ssh/config)
Host myserver
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/mykey
```

**Network Information:**
```bash
ip a                                 # Show IP addresses
ip addr show                         # Detailed IP information
hostname                             # Show hostname
hostname -I                          # Show IP addresses
```

**Security Notes:**
- SSH encrypts all data (preferred over Telnet)
- Telnet transmits in plain text (insecure, avoid)
- Use SSH keys instead of passwords when possible

## System Documentation

### Man Pages

**Basic Usage:**
```bash
man command                          # View manual page
man -k keyword                       # Search man pages by keyword
man -f command                       # Short description of command
apropos keyword                      # Search command descriptions
whatis command                       # One-line description

# Man page sections:
# 1 - User commands
# 2 - System calls
# 3 - Library functions
# 4 - Special files (devices)
# 5 - File formats and conventions
# 6 - Games
# 7 - Miscellaneous
# 8 - System administration commands

man 5 passwd                         # View passwd file format (section 5)
man 1 passwd                         # View passwd command (section 1)
```

**Navigation in Man Pages:**
```bash
Space                                # Page down
b                                    # Page up
/pattern                             # Search forward
?pattern                             # Search backward
n                                    # Next search result
N                                    # Previous search result
q                                    # Quit
```

### Other Documentation

**Info Pages:**
```bash
info command                         # GNU info documentation
info coreutils                       # Core utilities documentation
```

**Built-in Help:**
```bash
command --help                       # Quick help for most commands
help                                 # List bash built-in commands
help cd                              # Help for bash built-ins
type command                         # Show command type
which command                        # Show command path
whereis command                      # Show binary, source, and man pages
```

**Documentation Directories:**
```bash
/usr/share/doc/                      # Package documentation
/usr/share/man/                      # Manual pages
/usr/share/info/                     # Info pages
```

**Command Auto-completion:**
```bash
# Press TAB for auto-completion
systemctl st<TAB>                    # Complete command
ls /usr/sh<TAB>                      # Complete path
```

## File and Directory Management

### Creating Files and Directories

```bash
touch file.txt                       # Create empty file or update timestamp
touch file1 file2 file3              # Create multiple files
mkdir directory                      # Create directory
mkdir -p /path/to/nested/dir         # Create nested directories
mkdir -m 755 directory               # Create with specific permissions
```

### Copying Files and Directories

```bash
cp source dest                       # Copy file
cp -r dir1 dir2                      # Copy directory recursively
cp -i file1 file2                    # Interactive (prompt before overwrite)
cp -p file1 file2                    # Preserve attributes (mode, ownership, timestamps)
cp -a dir1 dir2                      # Archive mode (preserve all attributes)
cp -v source dest                    # Verbose (show what's being copied)
cp -u source dest                    # Copy only when source is newer
```

### Moving and Renaming

```bash
mv old new                           # Rename file or move
mv file /path/to/destination/        # Move to directory
mv -i old new                        # Interactive mode
mv -v old new                        # Verbose mode
mv file1 file2 file3 /destination/   # Move multiple files
```

### Deleting Files and Directories

```bash
rm file                              # Remove file
rm -i file                           # Interactive removal
rm -f file                           # Force removal (no prompts)
rm -r directory                      # Remove directory recursively
rm -rf directory                     # Force recursive removal (use with caution!)
rmdir directory                      # Remove empty directory
```

**Safety Tips:**
- Always use `-i` flag when learning
- Double-check before using `-rf`
- Consider using `rm -I` (uppercase i) to prompt once for 3+ files

## Links (Hard and Soft)

### Hard Links

```bash
ln file hardlink                     # Create hard link
ls -li                               # View inode numbers (-i flag)
stat file                            # View inode and link count
```

**Hard Link Characteristics:**
- Same inode number as original
- Cannot cross filesystem boundaries
- Cannot link to directories
- If original deleted, data still accessible via hard link
- Link count increases

**Example:**
```bash
$ touch original.txt
$ ln original.txt hardlink.txt
$ ls -li
12345 -rw-r--r-- 2 user group 0 Jan 22 10:00 original.txt
12345 -rw-r--r-- 2 user group 0 Jan 22 10:00 hardlink.txt
# Same inode (12345), link count is 2
```

### Symbolic Links (Soft Links)

```bash
ln -s target symlink                 # Create symbolic link
ln -s /path/to/file /path/to/link    # Absolute paths
ln -s ../file symlink                # Relative path
readlink symlink                     # Show link target
ls -l                                # Shows link and target (link -> target)
```

**Symbolic Link Characteristics:**
- Different inode number from original
- Can cross filesystem boundaries
- Can link to directories
- If original deleted, link becomes broken (dangling)
- Link points to path, not inode

**Example:**
```bash
$ touch original.txt
$ ln -s original.txt symlink.txt
$ ls -li
12345 -rw-r--r-- 1 user group 0 Jan 22 10:00 original.txt
67890 lrwxrwxrwx 1 user group 12 Jan 22 10:01 symlink.txt -> original.txt
# Different inodes, symlink shows as 'l' type and points to target
```

### Comparison

| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Inode | Same as original | Different |
| Cross filesystems | No | Yes |
| Link to directories | No | Yes |
| Survives deletion of original | Yes | No (becomes dangling) |
| Size | Same as original | Size of path string |

## File Permissions

### Understanding Permissions

**Permission Types:**
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

**Permission Groups:**
- User (u) - Owner
- Group (g) - Group members
- Other (o) - Everyone else

**Display Format:**
```bash
-rwxr-xr--
| |  |  |
| |  |  +-- Other permissions (r--)
| |  +----- Group permissions (r-x)
| +-------- User permissions (rwx)
+---------- File type (- = file, d = directory, l = link)
```

### Changing Permissions (chmod)

**Numeric Mode:**
```bash
chmod 755 file                       # rwxr-xr-x
chmod 644 file                       # rw-r--r--
chmod 600 file                       # rw-------
chmod 777 file                       # rwxrwxrwx (avoid!)
chmod -R 755 directory               # Recursive
```

**Common Permission Sets:**
- 755 = rwxr-xr-x (executables, directories)
- 644 = rw-r--r-- (regular files)
- 600 = rw------- (private files)
- 700 = rwx------ (private directories)

**Symbolic Mode:**
```bash
chmod u+x file                       # Add execute for user
chmod g-w file                       # Remove write for group
chmod o=r file                       # Set other to read-only
chmod a+r file                       # Add read for all (a = all)
chmod u+x,g+x,o-w file               # Multiple changes
chmod +x file                        # Add execute for all
```

### Changing Ownership

```bash
chown user file                      # Change owner
chown user:group file                # Change owner and group
chown -R user:group directory        # Recursive
chgrp group file                     # Change group only
```

### Special Permissions

**SUID (Set User ID) - 4xxx:**
```bash
chmod u+s file                       # Set SUID
chmod 4755 file                      # Numeric mode with SUID
ls -l                                # Shows as 'rws' or 'rwS'
```
- File executes with owner's permissions (not executor's)
- Example: `/usr/bin/passwd` (allows users to change their password)
- Shows as `s` in user execute position

**SGID (Set Group ID) - 2xxx:**
```bash
chmod g+s directory                  # Set SGID on directory
chmod 2755 directory                 # Numeric mode with SGID
```
- On file: Executes with group's permissions
- On directory: New files inherit directory's group
- Shows as `s` in group execute position

**Sticky Bit - 1xxx:**
```bash
chmod +t directory                   # Set sticky bit
chmod 1777 directory                 # Numeric mode with sticky bit
```
- On directory: Only owner can delete their files (even if others have write)
- Common on `/tmp` directory
- Shows as `t` in other execute position

**Example:**
```bash
$ ls -ld /tmp
drwxrwxrwt 10 root root 4096 Jan 22 10:00 /tmp
# Sticky bit (t) prevents users from deleting others' files
```

### Default Permissions (umask)

```bash
umask                                # View current umask
umask 022                            # Set umask (temporary)
umask -S                             # Symbolic display

# Default file: 666 - umask
# Default dir:  777 - umask
# Common umask: 022 = files 644, dirs 755
```

## Searching for Files

### find Command

**Basic Syntax:**
```bash
find /path -name "pattern"           # Search by name
find . -name "*.txt"                 # Find all .txt files
find /home -user john                # Files owned by john
find / -group staff                  # Files owned by group staff
```

**By Type:**
```bash
find /path -type f                   # Regular files
find /path -type d                   # Directories
find /path -type l                   # Symbolic links
find /path -type s                   # Sockets
find /path -type b                   # Block devices
```

**By Size:**
```bash
find / -size +100M                   # Larger than 100MB
find / -size -1M                     # Smaller than 1MB
find / -size 50M                     # Exactly 50MB
find / -empty                        # Empty files or directories
```

**By Time:**
```bash
find / -mtime -7                     # Modified in last 7 days
find / -mtime +30                    # Modified more than 30 days ago
find / -atime -1                     # Accessed in last day
find / -ctime -7                     # Status changed in last 7 days
find / -mmin -60                     # Modified in last 60 minutes
```

**By Permissions:**
```bash
find / -perm 755                     # Exact permissions
find / -perm -644                    # At least these permissions
find / -perm /u+w,g+w                # User OR group writable
```

**Execute Commands:**
```bash
find . -name "*.log" -delete         # Delete found files
find . -name "*.txt" -exec rm {} \;  # Execute command on each file
find . -type f -exec chmod 644 {} \; # Change permissions
find . -name "*.bak" -ok rm {} \;    # Interactive confirmation
```

**Complex Searches:**
```bash
find / -name "*.conf" -type f -user root  # Multiple conditions (AND)
find / -name "*.txt" -o -name "*.log"     # OR condition
find / -size +10M ! -name "*.mp4"         # NOT condition
find / -name "*.tmp" -o -name "*.cache" -delete  # Delete multiple patterns
```

### locate Command

```bash
locate filename                      # Fast search using database
locate -i filename                   # Case-insensitive
locate -c pattern                    # Count matches
locate -e filename                   # Only show existing files
sudo updatedb                        # Update locate database
```

**Difference: find vs locate**
- `find`: Real-time search, slower, more options
- `locate`: Database search, faster, requires updatedb

### Other Search Commands

```bash
which command                        # Show command path in PATH
whereis command                      # Show binary, source, man pages
type command                         # Show command type
```

## Text File Content Viewing

### Basic Viewing Commands

```bash
cat file                             # Display entire file
cat file1 file2                      # Concatenate multiple files
cat -n file                          # Number lines
cat -A file                          # Show all characters (including hidden)
tac file                             # Display in reverse (last line first)
```

### Pagers

**less (recommended):**
```bash
less file                            # View file with pagination
# Navigation:
# Space/f - Page down
# b - Page up
# /pattern - Search forward
# ?pattern - Search backward
# n/N - Next/previous match
# G - Go to end
# g - Go to beginning
# q - Quit
```

**more:**
```bash
more file                            # Basic pager (less features than less)
```

### Head and Tail

```bash
head file                            # First 10 lines
head -n 20 file                      # First 20 lines
head -20 file                        # Same as above
tail file                            # Last 10 lines
tail -n 20 file                      # Last 20 lines
tail -f file                         # Follow file (real-time updates)
tail -F file                         # Follow file, retry if file is rotated
```

### Comparing Files

```bash
diff file1 file2                     # Show differences
diff -u file1 file2                  # Unified format (better for patches)
diff -y file1 file2                  # Side-by-side comparison
diff -r dir1 dir2                    # Compare directories
cmp file1 file2                      # Byte-by-byte comparison
comm file1 file2                     # Compare sorted files line by line
```

## Text Processing

### grep (Search Text)

**Basic Usage:**
```bash
grep "pattern" file                  # Search for pattern
grep -i "pattern" file               # Case-insensitive
grep -v "pattern" file               # Invert match (exclude lines)
grep -n "pattern" file               # Show line numbers
grep -c "pattern" file               # Count matches
grep -l "pattern" *.txt              # Show only filenames
grep -r "pattern" /path              # Recursive search
grep -w "word" file                  # Match whole words only
grep -A 3 "pattern" file             # Show 3 lines after match
grep -B 3 "pattern" file             # Show 3 lines before match
grep -C 3 "pattern" file             # Show 3 lines before and after
```

**Multiple Patterns:**
```bash
grep -e "pattern1" -e "pattern2" file    # Multiple patterns (OR)
grep "pattern1\|pattern2" file           # Alternative OR syntax
grep -E "pattern1|pattern2" file         # Extended regex OR
```

### Regular Expressions

**Basic Regex (BRE):**
```bash
^                                    # Start of line
$                                    # End of line
.                                    # Any single character
*                                    # Zero or more of previous
[abc]                                # Any character in brackets
[^abc]                               # Any character NOT in brackets
[a-z]                                # Range of characters
\                                    # Escape special character

# Examples:
grep "^root" /etc/passwd             # Lines starting with "root"
grep "bash$" /etc/passwd             # Lines ending with "bash"
grep "^$" file                       # Empty lines
grep "[0-9]" file                    # Lines with numbers
```

**Extended Regex (ERE) - use `grep -E` or `egrep`:**
```bash
+                                    # One or more
?                                    # Zero or one
|                                    # OR
()                                   # Grouping
{n}                                  # Exactly n times
{n,}                                 # n or more times
{n,m}                                # Between n and m times

# Examples:
grep -E "^[A-Z]+$" file              # Lines with only uppercase letters
grep -E "(error|warning)" file       # Lines with error OR warning
grep -E "[0-9]{3}-[0-9]{4}" file     # Phone numbers (format: 123-4567)
```

### sed (Stream Editor)

**Basic Substitution:**
```bash
sed 's/old/new/' file                # Replace first occurrence on each line
sed 's/old/new/g' file               # Replace all occurrences (global)
sed 's/old/new/2' file               # Replace second occurrence
sed 's/old/new/gi' file              # Global, case-insensitive
sed -i 's/old/new/g' file            # In-place edit (modifies file)
sed -i.bak 's/old/new/g' file        # In-place with backup
```

**Line Selection:**
```bash
sed -n '5p' file                     # Print line 5
sed -n '1,5p' file                   # Print lines 1-5
sed -n '/pattern/p' file             # Print lines matching pattern
sed '5d' file                        # Delete line 5
sed '/pattern/d' file                # Delete lines matching pattern
```

**Multiple Commands:**
```bash
sed 's/old/new/g; s/foo/bar/g' file  # Multiple substitutions
sed -e 's/old/new/g' -e 's/foo/bar/g' file  # Alternative syntax
```

### awk (Pattern Scanning and Processing)

**Basic Usage:**
```bash
awk '{print}' file                   # Print all lines
awk '{print $1}' file                # Print first field
awk '{print $1,$3}' file             # Print fields 1 and 3
awk '{print $NF}' file               # Print last field
awk '{print NF}' file                # Print number of fields
awk '{print NR, $0}' file            # Print line number and line
```

**Field Separator:**
```bash
awk -F: '{print $1}' /etc/passwd     # Use : as field separator
awk 'BEGIN{FS=":"} {print $1}' file  # Set field separator in BEGIN
```

**Patterns and Conditions:**
```bash
awk '/pattern/' file                 # Print lines matching pattern
awk '$3 > 100' file                  # Print lines where field 3 > 100
awk '$1 == "root"' /etc/passwd       # Print lines where field 1 equals "root"
awk 'NR==5' file                     # Print line 5
awk 'NR>=5 && NR<=10' file           # Print lines 5-10
```

**Built-in Variables:**
```bash
NR                                   # Current line number
NF                                   # Number of fields
FS                                   # Input field separator (default: space)
OFS                                  # Output field separator
RS                                   # Input record separator (default: newline)
ORS                                  # Output record separator
FILENAME                             # Current filename
```

**Examples:**
```bash
awk -F: '{print $1,$3}' /etc/passwd  # Print username and UID
awk '{sum+=$1} END {print sum}' file # Sum first column
awk 'length($0) > 80' file           # Lines longer than 80 characters
```

### Other Text Tools

```bash
sort file                            # Sort alphabetically
sort -n file                         # Sort numerically
sort -r file                         # Reverse sort
sort -k2 file                        # Sort by second field
sort -u file                         # Sort and remove duplicates
uniq file                            # Remove consecutive duplicates
uniq -c file                         # Count occurrences
cut -d: -f1 /etc/passwd              # Cut fields (delimiter :, field 1)
tr 'a-z' 'A-Z' < file                # Translate lowercase to uppercase
tr -d '0-9' < file                   # Delete all digits
wc file                              # Count lines, words, characters
wc -l file                           # Count lines
wc -w file                           # Count words
wc -c file                           # Count bytes
```

## Vi/Vim Text Editor

### Modes

- **Normal Mode**: Navigation and commands (default)
- **Insert Mode**: Text editing (press `i`, `a`, `o`)
- **Visual Mode**: Text selection (press `v`, `V`, `Ctrl+v`)
- **Command Mode**: Save, quit, search (press `:`)

### Basic Commands

**Opening Files:**
```bash
vi file                              # Open file
vim file                             # Open with Vim
vi +10 file                          # Open at line 10
vi + file                            # Open at last line
```

**Modes:**
```bash
Esc                                  # Return to normal mode
i                                    # Insert before cursor
a                                    # Insert after cursor
A                                    # Insert at end of line
o                                    # Insert new line below
O                                    # Insert new line above
```

**Navigation (Normal Mode):**
```bash
h, j, k, l                           # Left, down, up, right
w                                    # Next word
b                                    # Previous word
0                                    # Beginning of line
$                                    # End of line
gg                                   # First line
G                                    # Last line
:10                                  # Go to line 10
Ctrl+f                               # Page down
Ctrl+b                               # Page up
```

**Editing (Normal Mode):**
```bash
x                                    # Delete character
dd                                   # Delete line
dw                                   # Delete word
d$                                   # Delete to end of line
yy                                   # Copy (yank) line
yw                                   # Copy word
p                                    # Paste after cursor
P                                    # Paste before cursor
u                                    # Undo
Ctrl+r                               # Redo
.                                    # Repeat last command
```

**Search and Replace:**
```bash
/pattern                             # Search forward
?pattern                             # Search backward
n                                    # Next match
N                                    # Previous match
:s/old/new/                          # Replace first on current line
:s/old/new/g                         # Replace all on current line
:%s/old/new/g                        # Replace all in file
:%s/old/new/gc                       # Replace all with confirmation
```

**Save and Quit (Command Mode):**
```bash
:w                                   # Save
:q                                   # Quit
:wq                                  # Save and quit
:q!                                  # Quit without saving
:x                                   # Save and quit (if changes made)
ZZ                                   # Save and quit (normal mode)
:w newfile                           # Save as newfile
```

## Archive, Compression, and Backup

### tar (Tape Archive)

**Creating Archives:**
```bash
tar -cvf archive.tar files           # Create tar archive
tar -czvf archive.tar.gz files       # Create with gzip compression
tar -cjvf archive.tar.bz2 files      # Create with bzip2 compression
tar -cJvf archive.tar.xz files       # Create with xz compression
tar -czf backup.tar.gz /home/user    # Backup directory

# Flags:
# c - create
# x - extract
# t - list contents
# v - verbose
# f - file
# z - gzip
# j - bzip2
# J - xz
```

**Extracting Archives:**
```bash
tar -xvf archive.tar                 # Extract tar
tar -xzvf archive.tar.gz             # Extract gzip
tar -xjvf archive.tar.bz2            # Extract bzip2
tar -xJvf archive.tar.xz             # Extract xz
tar -xvf archive.tar -C /destination # Extract to specific directory
```

**Listing Contents:**
```bash
tar -tvf archive.tar                 # List contents
tar -tzvf archive.tar.gz             # List gzip contents
```

**Appending and Updating:**
```bash
tar -rvf archive.tar newfile         # Append file to archive
tar -uvf archive.tar file            # Update file in archive if newer
```

### Compression Tools

**gzip:**
```bash
gzip file                            # Compress file (creates file.gz, removes original)
gzip -k file                         # Keep original file
gzip -9 file                         # Maximum compression
gunzip file.gz                       # Decompress
gzip -d file.gz                      # Decompress (alternative)
gzip -c file > file.gz               # Compress to stdout
```

**bzip2:**
```bash
bzip2 file                           # Compress (creates file.bz2)
bzip2 -k file                        # Keep original
bunzip2 file.bz2                     # Decompress
bzip2 -d file.bz2                    # Decompress (alternative)
```

**xz:**
```bash
xz file                              # Compress (creates file.xz)
xz -k file                           # Keep original
unxz file.xz                         # Decompress
xz -d file.xz                        # Decompress (alternative)
```

**zip/unzip:**
```bash
zip archive.zip files                # Create zip archive
zip -r archive.zip directory         # Recursive zip
unzip archive.zip                    # Extract zip
unzip -l archive.zip                 # List contents
unzip archive.zip -d /destination    # Extract to directory
```

### Compression Comparison

| Tool | Extension | Compression Ratio | Speed | Common Use |
|------|-----------|------------------|-------|------------|
| gzip | .gz | Good | Fast | General purpose |
| bzip2 | .bz2 | Better | Slower | Better compression |
| xz | .xz | Best | Slowest | Best compression |
| zip | .zip | Good | Fast | Cross-platform |

### Remote Backup

**scp (Secure Copy):**
```bash
scp file user@host:/path             # Copy file to remote
scp user@host:/path/file .           # Copy file from remote
scp -r directory user@host:/path     # Copy directory recursively
scp -P 2222 file user@host:/path     # Use specific port
scp -p file user@host:/path          # Preserve timestamps and permissions
```

**rsync (Efficient Sync):**
```bash
rsync -av /source /destination       # Archive mode, verbose
rsync -avz /source user@host:/dest   # With compression
rsync -av --delete /src /dest        # Delete files not in source
rsync -av --exclude='*.log' /src /dest  # Exclude pattern
rsync -avP /source /destination      # Show progress
rsync -av --dry-run /src /dest       # Test without changes
rsync -av --backup /src /dest        # Backup existing files
rsync -av -e "ssh -p 2222" /src user@host:/dest  # Custom SSH

# Common flags:
# -a = archive (recursive, preserve permissions, timestamps, etc.)
# -v = verbose
# -z = compression
# -P = progress and partial
# --delete = delete files not in source
# --exclude = exclude files/patterns
# --dry-run = test without making changes
```

**sftp (Secure FTP):**
```bash
sftp user@host                       # Connect to SFTP
# SFTP commands:
put file                             # Upload file
get file                             # Download file
ls                                   # List remote directory
lls                                  # List local directory
cd /path                             # Change remote directory
lcd /path                            # Change local directory
mkdir dirname                        # Create remote directory
bye                                  # Exit
```

## Input/Output Redirection

### Standard Streams

- **stdin (0)**: Standard input (keyboard)
- **stdout (1)**: Standard output (screen)
- **stderr (2)**: Standard error (screen)

### Output Redirection

```bash
command > file                       # Redirect stdout (overwrites)
command >> file                      # Append stdout
command 2> file                      # Redirect stderr
command 2>> file                     # Append stderr
command &> file                      # Redirect both stdout and stderr
command > file 2>&1                  # Redirect both (older syntax)
command >> file 2>&1                 # Append both
command > /dev/null                  # Discard output
command 2> /dev/null                 # Discard errors
command &> /dev/null                 # Discard all output
```

### Input Redirection

```bash
command < file                       # Read input from file
command << EOF                       # Here document (multi-line input)
line 1
line 2
EOF

command <<< "string"                 # Here string (single line)
```

### Pipes

```bash
command1 | command2                  # Pipe stdout of cmd1 to stdin of cmd2
command1 | tee file | command2       # Save to file and pipe
command1 |& command2                 # Pipe both stdout and stderr
```

### Examples

```bash
# Redirect output to file
ls -l > listing.txt

# Append to file
date >> log.txt

# Redirect errors
find / -name "*.txt" 2> /dev/null

# Save output and errors to different files
command > output.txt 2> error.txt

# Save both to same file
command > all_output.txt 2>&1

# Use output as input
sort < unsorted.txt > sorted.txt

# Multiple pipes
cat file | grep "pattern" | sort | uniq

# Save intermediate results
cat file | grep "error" | tee errors.txt | wc -l
```

## SSL Certificates

### OpenSSL Commands

**Generate Private Key:**
```bash
openssl genrsa -out private.key 2048               # 2048-bit RSA key
openssl genrsa -out private.key 4096               # 4096-bit RSA key
openssl genrsa -aes256 -out private.key 2048       # Encrypted key
```

**Generate CSR (Certificate Signing Request):**
```bash
openssl req -new -key private.key -out request.csr
openssl req -new -newkey rsa:2048 -nodes -keyout private.key -out request.csr
```

**Generate Self-Signed Certificate:**
```bash
openssl req -x509 -newkey rsa:2048 -nodes -keyout key.pem -out cert.pem -days 365
openssl req -x509 -key private.key -in request.csr -out certificate.crt -days 365
```

**View Certificate:**
```bash
openssl x509 -in certificate.crt -text -noout     # View details
openssl x509 -in certificate.crt -noout -subject  # View subject
openssl x509 -in certificate.crt -noout -issuer   # View issuer
openssl x509 -in certificate.crt -noout -dates    # View validity dates
openssl x509 -in certificate.crt -noout -fingerprint  # View fingerprint
```

**Verify Certificate:**
```bash
openssl verify certificate.crt                     # Verify certificate
openssl x509 -in certificate.crt -noout -checkend 86400  # Check if expires in 24h
```

**Convert Certificate Formats:**
```bash
openssl x509 -in cert.pem -outform DER -out cert.der     # PEM to DER
openssl x509 -in cert.der -inform DER -out cert.pem      # DER to PEM
```

**Test SSL Connection:**
```bash
openssl s_client -connect example.com:443          # Test HTTPS connection
openssl s_client -connect example.com:443 -showcerts  # Show certificate chain
```

**View Certificate from Server:**
```bash
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -text -noout
```

## Git Operations

### Basic Git Setup

```bash
git --version                        # Check Git version
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor vim
git config --list                    # View configuration
git config user.name                 # View specific setting
```

### Repository Initialization

```bash
git init                             # Initialize new repository
git init project-name                # Create directory and initialize
git clone https://github.com/user/repo.git        # Clone repository
git clone https://github.com/user/repo.git mydir  # Clone to specific directory
git clone --depth 1 url              # Shallow clone (faster, less history)
```

### Basic Workflow

**Checking Status:**
```bash
git status                           # Show working tree status
git status -s                        # Short status
git status -uno                      # Don't show untracked files
```

**Staging Changes:**
```bash
git add file                         # Stage specific file
git add .                            # Stage all changes in current directory
git add -A                           # Stage all changes in repository
git add *.txt                        # Stage files by pattern
git add -p                           # Interactive staging (patch mode)
git reset file                       # Unstage file
git reset                            # Unstage all
```

**Committing:**
```bash
git commit -m "Commit message"       # Commit staged changes
git commit -am "Message"             # Stage and commit tracked files
git commit --amend                   # Amend last commit
git commit --amend -m "New message"  # Change last commit message
```

**Viewing History:**
```bash
git log                              # Show commit history
git log --oneline                    # Compact view
git log --graph --oneline --all      # Visual branch graph
git log -n 5                         # Last 5 commits
git log --author="name"              # Commits by author
git log --since="2024-01-01"         # Commits since date
git log --grep="pattern"             # Search commit messages
git log -p                           # Show diffs
git show commit-hash                 # Show specific commit
git show HEAD                        # Show last commit
```

**Viewing Changes:**
```bash
git diff                             # Unstaged changes
git diff --staged                    # Staged changes
git diff --cached                    # Same as --staged
git diff HEAD                        # All changes (staged + unstaged)
git diff branch1 branch2             # Compare branches
git diff commit1 commit2             # Compare commits
```

### Branch Management

**Creating and Switching:**
```bash
git branch                           # List local branches
git branch -a                        # List all branches (local + remote)
git branch -r                        # List remote branches
git branch new-branch                # Create new branch
git checkout -b new-branch           # Create and switch to branch
git switch new-branch                # Switch to branch (modern)
git switch -c new-branch             # Create and switch (modern)
git checkout branch-name             # Switch to branch (traditional)
```

**Managing Branches:**
```bash
git branch -d branch-name            # Delete branch (safe)
git branch -D branch-name            # Force delete branch
git branch -m old-name new-name      # Rename branch
git branch -v                        # Show branches with last commit
```

**Merging:**
```bash
git merge branch-name                # Merge branch into current branch
git merge --no-ff branch-name        # Merge with merge commit
git merge --squash branch-name       # Squash merge
git merge --abort                    # Abort merge
```

**Merge Conflicts:**
```bash
# When conflict occurs:
git status                           # See conflicted files
# Edit files to resolve conflicts
git add resolved-file                # Mark as resolved
git commit                           # Complete merge
```

### Remote Repositories

**Managing Remotes:**
```bash
git remote                           # List remotes
git remote -v                        # List remotes with URLs
git remote add origin url            # Add remote
git remote remove origin             # Remove remote
git remote rename origin upstream    # Rename remote
git remote show origin               # Show remote info
git remote set-url origin new-url    # Change remote URL
```

**Fetching and Pulling:**
```bash
git fetch                            # Fetch from default remote
git fetch origin                     # Fetch from specific remote
git fetch --all                      # Fetch from all remotes
git pull                             # Fetch and merge
git pull origin branch               # Pull specific branch
git pull --rebase                    # Pull with rebase instead of merge
```

**Pushing:**
```bash
git push                             # Push to default remote
git push origin branch               # Push to specific branch
git push -u origin branch            # Set upstream and push
git push --all                       # Push all branches
git push --tags                      # Push tags
git push --force                     # Force push (dangerous!)
git push origin --delete branch      # Delete remote branch
```

### Undoing Changes

```bash
git checkout -- file                 # Discard changes in working directory
git restore file                     # Discard changes (modern)
git restore --staged file            # Unstage file (modern)
git reset HEAD file                  # Unstage file (traditional)
git reset --soft HEAD~1              # Undo last commit, keep changes staged
git reset --mixed HEAD~1             # Undo last commit, keep changes unstaged
git reset --hard HEAD~1              # Undo last commit, discard changes
git revert commit-hash               # Create new commit that undoes changes
```

### Tagging

```bash
git tag                              # List tags
git tag v1.0.0                       # Create lightweight tag
git tag -a v1.0.0 -m "Version 1.0"   # Create annotated tag
git tag -d v1.0.0                    # Delete tag
git push origin v1.0.0               # Push specific tag
git push origin --tags               # Push all tags
```

### Stashing

```bash
git stash                            # Stash changes
git stash save "message"             # Stash with message
git stash list                       # List stashes
git stash pop                        # Apply and remove last stash
git stash apply                      # Apply last stash (keep in stash)
git stash apply stash@{0}            # Apply specific stash
git stash drop stash@{0}             # Delete specific stash
git stash clear                      # Delete all stashes
```

### Best Practices

1. **Commit Messages**: Clear, descriptive, present tense ("Add feature" not "Added feature")
2. **Branch Names**: Descriptive (feature/user-auth, bugfix/login-error)
3. **Commit Often**: Small, logical commits are better than large ones
4. **Pull Before Push**: Always pull latest changes before pushing
5. **Review Before Commit**: Use `git diff` to review changes
6. **Avoid Force Push**: Especially on shared branches (main/master)
7. **.gitignore**: Use to exclude files (node_modules/, *.log, .env)

### Common Workflows

**Feature Branch Workflow:**
```bash
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "Add new feature"
git push -u origin feature/new-feature
# Create pull request
# After merge:
git checkout main
git pull
git branch -d feature/new-feature
```

**Fixing Mistakes:**
```bash
# Wrong commit message:
git commit --amend -m "Correct message"

# Forgot to add file:
git add forgotten-file
git commit --amend --no-edit

# Committed to wrong branch:
git reset HEAD~1
git stash
git checkout correct-branch
git stash pop
git add .
git commit -m "Message"
```

## Quick Reference Summary

**Must-Know Commands:**
- `ssh user@host` - Remote login
- `man command` - Documentation
- `ls -la` - List files with details
- `chmod 755 file` - Change permissions
- `find / -name "file"` - Search files
- `grep "pattern" file` - Search text
- `tar -czf archive.tar.gz files` - Create compressed archive
- `rsync -avz /src user@host:/dest` - Efficient sync
- `command > file` - Redirect output
- `git status` - Check repository status
- `git commit -m "message"` - Commit changes

**File Permissions Cheat Sheet:**
- 755 (rwxr-xr-x) - Executables/directories
- 644 (rw-r--r--) - Regular files
- 600 (rw-------) - Private files
- 777 (rwxrwxrwx) - Avoid! (too permissive)

**Vim Quick Exit:**
- `:wq` - Save and quit
- `:q!` - Quit without saving
- `ZZ` - Save and quit (normal mode)
