# 🏴‍☠️ OverTheWire: Bandit — My Walkthrough

> **Personal notes from completing the Bandit wargame by OverTheWire**  
> This isn't just a list of commands — I tried to explain *why* everything works the way it does.  
> Website: [overthewire.org/wargames/bandit](https://overthewire.org/wargames/bandit/)

---

## What is Bandit?

Bandit is a wargame (a CTF-style learning game) aimed at absolute Linux beginners. Each level is a separate user account on a remote server (`bandit0`, `bandit1`, ..., `bandit33`). Your job is to find the password for the next user and log in as them.

The game gradually teaches Linux fundamentals: filesystems, file permissions, network utilities, archives, and even Git. It takes a few hours to complete, but it builds a solid foundation.

### How to connect

```bash
ssh banditN@bandit.labs.overthewire.org -p 2220
```

Where `N` is the level number (0, 1, 2...). Port is always `2220`.

---

## Level 0 — First Login

**Goal:** Connect to the server via SSH.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
```

**What is SSH?**  
SSH (Secure Shell) is a protocol for secure remote server management over an encrypted connection. The `-p 2220` flag specifies the non-standard port (the default is 22).

After logging in, you're greeted by the OverTheWire welcome banner. You're in!

---

## Level 0 → 1

**Goal:** The password is stored in a file called `readme` in the home directory.

```bash
ls           # see what's in the current directory
cat readme   # read the file contents
```

**Explanation:**
- `ls` — lists files in the current directory (from *list*)
- `cat` — prints file contents to the terminal (from *concatenate*)
- When logging in via SSH, you automatically land in the home directory (`~`)

The password is right there in the `cat` output. Write it down and move on.

---

## Level 1 → 2

**Goal:** The password is in a file named `-`.

I ran `ls` and saw a file called `-`. Tried `cat -` — the terminal hung. Turns out, `-` is a special symbol meaning standard input (stdin), and `cat` started waiting for keyboard input.

```bash
cat ./-
# or
cat < -
```

**Explanation:**
- `./` explicitly specifies the file path, preventing the shell from treating `-` as a flag
- `< -` also works — it redirects the file contents to stdin
- Lesson: files with unusual names need an explicit path via `./`

---

## Level 2 → 3

**Goal:** The password is in a file called `spaces in this filename`.

I tried `cat spaces in this filename` — bash treats each word as a separate argument and complains that files `spaces`, `in`, `this`, and `filename` don't exist.

```bash
cat "spaces in this filename"
# or
cat spaces\ in\ this\ filename
# or just start typing and press Tab!
```

**Explanation:**
- A space is an argument separator in bash
- Quotes `"..."` group multiple words into one argument
- Backslash `\` escapes a special character
- **Pro tip:** Tab autocomplete automatically adds `\` before spaces

---

## Level 3 → 4

**Goal:** The password is in a hidden file in the `inhere` directory.

```bash
cd inhere
ls        # nothing shows up!
ls -a     # now we see: . .. .hidden
cat .hidden
```

**Explanation:**
- In Linux, files starting with `.` are hidden — `ls` doesn't show them by default
- The `-a` flag (all) reveals every file, including hidden ones
- `cd` — change directory

Hidden files aren't real security — they're just a convention. Usually these are config files (`.bashrc`, `.gitconfig`, etc.).

---

## Level 4 → 5

**Goal:** The password is in the only human-readable file in `inhere`. There are 10 files named `-file00` through `-file09`.

```bash
cd inhere
file ./*
```

Output:
```
./-file00: data
./-file01: data
...
./-file07: ASCII text   ← that's the one!
...
```

```bash
cat ./-file07
```

**Explanation:**
- `file` analyzes the actual content of a file and determines its type (doesn't rely on extension)
- `*` is a wildcard meaning "all files in the directory". We use `./` prefix so filenames aren't confused with flags
- "ASCII text" = human-readable text. The rest are binary data

---

## Level 5 → 6

**Goal:** The password file is somewhere in `inhere` (20 subdirectories!) with these properties: human-readable, 1033 bytes in size, not executable.

```bash
cd inhere
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```

**Explanation:**
- `find` — search for files by conditions
- `-type f` — only files, not directories
- `-size 1033c` — size is 1033 bytes (`c` = bytes, `k` = KB, `M` = MB)
- `! -executable` — not executable (`!` is negation)

`find` is one of the most powerful Linux utilities. Worth learning well.

---

## Level 6 → 7

**Goal:** The password file is **somewhere on the server** with these properties: owned by user `bandit7`, owned by group `bandit6`, 33 bytes in size.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

**Explanation:**
- We start the search at `/` — the root of the entire filesystem
- `-user bandit7` — the file is owned by user `bandit7`
- `-group bandit6` — the group owner is `bandit6`
- `2>/dev/null` — redirects stderr (error stream, file descriptor `2`) to `/dev/null` — Linux's "trash bin". Without this, the output is flooded with hundreds of "Permission denied" lines

---

## Level 7 → 8

**Goal:** The password is in a huge file `data.txt`, next to the word `millionth`.

```bash
grep "millionth" data.txt
```

Returns one line immediately: `millionth    <password>`.

**Explanation:**
- `grep` — searches for lines matching a pattern (Global Regular Expression Print)
- Finds all lines containing the given text and prints them
- Without `grep` you'd have to scroll through thousands of lines manually

---

## Level 8 → 9

**Goal:** The password is the only line in `data.txt` that appears exactly once. All other lines are repeated many times.

```bash
sort data.txt | uniq -u
```

**Explanation:**
- `sort` — sorts lines alphabetically. This is critical for `uniq` to work correctly
- `uniq -u` — outputs only lines with no duplicates (`-u` = unique)
- `|` — pipe: passes the output of one command as input to another. Fundamental Unix concept!

Without sorting first, `uniq` won't detect duplicates that are scattered throughout the file.

---

## Level 9 → 10

**Goal:** The password is in `data.txt` (mostly binary) — one of the readable strings, preceded by several `=` characters.

```bash
strings data.txt | grep "^="
```

**Explanation:**
- `strings` — extracts all readable ASCII character sequences from a file, even binary ones
- `grep "^="` — finds lines starting with `=` (`^` in regex = beginning of line)
- Again using the pipe `|` to chain commands

---

## Level 10 → 11

**Goal:** `data.txt` contains data encoded in Base64.

```bash
cat data.txt    # shows: VGhlIHBhc3N3b3JkIGlzIGR...
base64 -d data.txt
```

**Explanation:**
- Base64 encodes binary data as an ASCII string (A-Z, a-z, 0-9, +, /)
- Easy to spot: a string of seemingly random characters, often ending with `=` or `==`
- `-d` — decode mode
- Base64 is **not encryption**! It's just a different representation of data, trivially reversible

---

## Level 11 → 12

**Goal:** The password in `data.txt` is encrypted with ROT13.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

**Explanation:**
- ROT13 shifts each letter 13 positions in the alphabet: A→N, B→O, ..., N→A
- `tr` (translate) — replaces characters according to a translation table
- `'A-Za-z'` → `'N-ZA-Mn-za-m'` describes the ROT13 mapping for upper and lower case
- Applying ROT13 twice returns the original text (symmetric operation)

---

## Level 12 → 13

**Goal:** `data.txt` is a hexdump of a file that was compressed multiple times. Need to unpack it all.

The most tedious level — multiple decompression steps needed. Patience is key.

```bash
mkdir /tmp/mywork && cp data.txt /tmp/mywork/ && cd /tmp/mywork

# Convert the hexdump back to a binary file
xxd -r data.txt > data.bin

# Check the type
file data.bin
```

Then decompress iteratively:

```bash
# If gzip:
mv data.bin data.gz && gzip -d data.gz

# If bzip2:
mv data data.bz2 && bzip2 -d data.bz2

# If tar:
tar xf data.tar

# Check again:
file data        # repeat until "ASCII text"
```

**Explanation:**
- `xxd -r` — converts a hexdump back to binary
- `gzip -d` / `bzip2 -d` — decompresses the respective formats
- `tar xf` — extracts a tar archive (`x` = extract, `f` = file)
- At each step, `file` tells you what format you're dealing with next

---

## Level 13 → 14

**Goal:** A private SSH key for `bandit14` is sitting in the home directory. No explicit password needed.

```bash
ls
# sshkey.private

ssh -i sshkey.private bandit14@localhost -p 2220
```

**Explanation:**
- SSH supports key-based authentication instead of passwords — more secure
- `-i` (identity file) — specifies the private key
- `localhost` — the very server we're already connected to
- How it works: you hold the private key, the server has the public key. The server verifies you own the matching private key without you ever transmitting it

---

## Level 14 → 15

**Goal:** Submit the current password (`bandit14`) to `localhost:30000` via netcat and get the next one.

```bash
cat /etc/bandit_pass/bandit14      # read the current password
echo "<bandit14_password>" | nc localhost 30000
```

**Explanation:**
- `nc` (netcat) — a TCP/UDP connection utility, the "Swiss army knife" of networking
- `echo "text" | nc host port` — sends a string to a server
- The service on port 30000 accepts the password, validates it, and responds with the next one

---

## Level 15 → 16

**Goal:** Same as before, but port `30001` requires an SSL/TLS connection.

```bash
openssl s_client -connect localhost:30001
# Enter the bandit15 password, press Enter
```

**Explanation:**
- Regular `nc` doesn't speak SSL — the connection would be rejected
- `openssl s_client` — an SSL/TLS client for testing encrypted connections
- After connecting, we just type the password as usual — OpenSSL handles the encryption transparently

---

## Level 16 → 17

**Goal:** Somewhere in port range `31000-32000`, there's an SSL service. Submit the password and receive a private SSH key.

```bash
# Scan the port range
nmap -sV localhost -p 31000-32000

# Try the SSL-enabled ports (looking for one that responds with a key, not just echo)
openssl s_client -connect localhost:31790
# Enter the bandit16 password

# Save the received RSA key
mkdir /tmp/mykey && nano /tmp/mykey/key.pem   # paste the key
chmod 400 /tmp/mykey/key.pem    # IMPORTANT: SSH requires restricted key permissions

ssh -i /tmp/mykey/key.pem bandit17@localhost -p 2220
```

**Explanation:**
- `nmap -sV` — scans ports and detects service versions
- Some ports just echo back what you send — the right one responds differently
- `chmod 400` — read-only for the owner only. SSH refuses to use a key with broader permissions (protection against accidental exposure)

---

## Level 17 → 18

**Goal:** Two files in home directory: `passwords.old` and `passwords.new`. The password is the line that appeared in `new`.

```bash
diff passwords.old passwords.new
```

**Explanation:**
- `diff` — compares two files line by line
- Lines with `<` are only in the first file, `>` only in the second
- We want the line marked with `>` (from `passwords.new`) — that's the new password

---

## Level 18 → 19

**Goal:** The password is in `readme`, but `.bashrc` is configured to immediately kick you out on login.

I log in — and get immediately logged out with "Byebye!" We need to bypass `.bashrc`.

```bash
# Pass a command directly over SSH without starting an interactive session
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"

# Or force a different shell
ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh
```

**Explanation:**
- `.bashrc` only runs during interactive bash sessions
- Passing a command directly to SSH (`ssh user@host "command"`) runs it without an interactive shell
- `-t /bin/sh` — forces `/bin/sh` instead of bash, bypassing `.bashrc` entirely

---

## Level 19 → 20

**Goal:** There's a SUID binary called `bandit20-do` in the home directory. Use it to read the password.

```bash
ls -la
# -rwsr-x--- bandit20-do (notice the 's' in permissions)

./bandit20-do cat /etc/bandit_pass/bandit20
```

**Explanation:**
- SUID (Set User ID) — a special permission bit. A program with SUID runs as its **owner**, not as the user who launches it
- `bandit20-do` is owned by `bandit20` → running it grants `bandit20`'s privileges
- The letter `s` instead of `x` in permissions (`rws`) is the SUID indicator
- This lets us read `/etc/bandit_pass/bandit20` even though we have no direct access

---

## Level 20 → 21

**Goal:** A SUID binary `suconnect` connects to a port on localhost, reads the current password, and returns the next one. We need to "host" a server for it.

```bash
# Start a netcat listener in the background
echo "<bandit20_password>" | nc -l -p 5555 &

# Run suconnect
./suconnect 5555
```

**Explanation:**
- `nc -l -p 5555` — netcat in listening (server) mode on port 5555
- `&` — runs the process in the background, freeing up the terminal
- `suconnect` connects, receives our password, validates it, and responds with the next one
- Any port above 1024 works (below requires root)

---

## Level 21 → 22

**Goal:** Find a cron job, understand what it does, and get the password.

```bash
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
# Shows: * * * * * bandit22 /usr/bin/cronjob_bandit22.sh

cat /usr/bin/cronjob_bandit22.sh
# chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
# cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv

cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

**Explanation:**
- `cron` — Linux's task scheduler. `* * * * *` means "every minute"
- The script regularly copies the password into a `/tmp/` file with public read permissions
- `/etc/cron.d/` — directory containing cron job configurations

---

## Level 22 → 23

**Goal:** Another cron job, but this time we need to compute the name of the password file ourselves.

```bash
cat /etc/cron.d/cronjob_bandit23
cat /usr/bin/cronjob_bandit23.sh
```

The script does:
```bash
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

We replicate this logic for user `bandit23`:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
# Returns a hash, e.g.: 8ca319486bfbbc3663ea0fbe81326349

cat /tmp/8ca319486bfbbc3663ea0fbe81326349
```

**Explanation:**
- `md5sum` — computes the MD5 hash of a string
- `cut -d ' ' -f 1` — takes the first field (space-delimited), i.e., the hash itself without trailing spaces
- We're simulating what the script would do when run as `bandit23`

---

## Level 23 → 24

**Goal:** Write a bash script that cron will execute as `bandit24`, causing it to write the password somewhere we can read it.

```bash
mkdir /tmp/b24work
chmod 777 /tmp/b24work

cat > /tmp/b24work/getpass.sh << 'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/b24work/password
chmod 777 /tmp/b24work/password
EOF

chmod 777 /tmp/b24work/getpass.sh
cp /tmp/b24work/getpass.sh /var/spool/bandit24/foo/

# Wait up to 60 seconds for cron to execute it
sleep 60
cat /tmp/b24work/password
```

**Explanation:**
- `/var/spool/bandit24/foo/` — cron for `bandit24` executes all scripts dropped in this directory
- Our script reads the protected password file (running as `bandit24`) and writes it to a file we can access
- `chmod 777 /tmp/b24work` — gives cron write permissions to our working directory
- Heredoc `<< 'EOF'` — a convenient way to write multiline text to a file

---

## Level 24 → 25

**Goal:** A daemon on port `30002` accepts the `bandit24` password + a 4-digit PIN (0000–9999). Need to brute-force it.

```bash
# Generate all 10000 combinations
for i in $(seq 0 9999); do
    echo "<bandit24_password> $(printf '%04d' $i)"
done > /tmp/pins.txt

# Send them all in one connection
cat /tmp/pins.txt | nc localhost 30002 | grep -v "Wrong"
```

**Explanation:**
- `seq 0 9999` — generates numbers from 0 to 9999
- `printf '%04d'` — formats the number with leading zeros (0001, 0042, 9999...)
- One large file + one connection is far more efficient than 10000 separate connections
- `grep -v "Wrong"` — hides "Wrong PIN" lines, leaving only the successful response

---

## Level 25 → 26

**Goal:** `bandit26` has a non-standard shell — we need to escape from it.

```bash
cat /etc/passwd | grep bandit26
# bandit26:x:11026:11026::/home/bandit26:/usr/bin/showtext

cat /usr/bin/showtext
# A script that runs 'more' and exits
```

**The trick:** Shrink your terminal window to just 4-5 rows so `more` can't fit the text and pauses.

```bash
ssh -i bandit26.sshkey bandit26@localhost -p 2220
# In 'more' mode, press 'v' → vim opens!

# In vim:
:set shell=/bin/bash
:shell
```

**Explanation:**
- If the text fits on screen, `more` exits immediately → the script exits too
- A tiny terminal forces `more` to pause → press `v` to open vim
- Vim can launch a shell: `:set shell` sets the shell type, `:shell` opens it
- This is a classic "restricted shell escape" via a text editor

---

## Level 26 → 27

**Goal:** Already inside `bandit26` (from the previous trick). There's a SUID binary `bandit27-do`.

```bash
# From inside vim:
:set shell=/bin/bash
:shell

# Now in regular bash:
./bandit27-do cat /etc/bandit_pass/bandit27
```

Same SUID principle as in level 19→20.

---

## Level 27 → 28

**Goal:** A Git repository at `ssh://bandit27-git@localhost/home/bandit27-git/repo`. Find the password.

```bash
mkdir /tmp/g27 && cd /tmp/g27
git clone ssh://bandit27-git@localhost:2220/home/bandit27-git/repo
cd repo
cat README
```

**Explanation:**
- `git clone` — copies the repository to your local machine
- The password for cloning is the same as `bandit27`'s password
- In this case, the password is right there in `README`

---

## Level 28 → 29

**Goal:** The repository had a password, but it was overwritten. Look through the commit history.

```bash
mkdir /tmp/g28 && cd /tmp/g28
git clone ssh://bandit28-git@localhost:2220/home/bandit28-git/repo
cd repo

cat README.md        # password: xxxxxxxxxx (redacted)

git log --oneline    # view commit history
git show HEAD~1      # or: git show <commit_hash>
```

**Explanation:**
- `git log` — full commit history. `--oneline` gives a compact view
- `git show <hash>` — shows exactly what changed in that commit
- In Git, data almost never disappears permanently — it lives in the history

---

## Level 29 → 30

**Goal:** The README says "no passwords in production". Check other branches.

```bash
mkdir /tmp/g29 && cd /tmp/g29
git clone ssh://bandit29-git@localhost:2220/home/bandit29-git/repo
cd repo

git branch -a           # all branches, including remote ones
git checkout dev        # switch to dev branch
cat README.md
```

**Explanation:**
- `git branch -a` — shows local and remote branches (`remotes/origin/...`)
- No passwords in production, but dev branch has them
- This actually happens in real projects!

---

## Level 30 → 31

**Goal:** The password is hidden in a Git tag.

```bash
mkdir /tmp/g30 && cd /tmp/g30
git clone ssh://bandit30-git@localhost:2220/home/bandit30-git/repo
cd repo

git tag              # list tags → we see: secret
git show secret      # read the tag contents
```

**Explanation:**
- `git tag` — labels on specific commits or objects. Used for release versions
- Annotated tags can store arbitrary data — in our case, the password

---

## Level 31 → 32

**Goal:** Push a file `key.txt` containing `May I come in?` to the `master` branch.

```bash
mkdir /tmp/g31 && cd /tmp/g31
git clone ssh://bandit31-git@localhost:2220/home/bandit31-git/repo
cd repo

echo "May I come in?" > key.txt
cat .gitignore       # *.txt — .txt files are ignored!

git add -f key.txt   # -f = force, bypass .gitignore
git commit -m "adding key.txt"
git push
```

**Explanation:**
- `.gitignore` — lists what Git should ignore
- `-f` in `git add` forces the file to be staged, ignoring `.gitignore`
- After `push`, the server validates the file and returns the password

---

## Level 32 → 33

**Goal:** After logging in, we're in an "UPPERCASE SHELL" — everything you type gets converted to uppercase. `ls` becomes `LS` — command not found.

```bash
$0
```

Now we're in a regular `sh`:

```bash
cat /etc/bandit_pass/bandit33
```

**Explanation:**
- `$0` — a special variable holding the name of the current shell or script
- The UPPERCASE shell doesn't touch variable references — `$0` stays `$0` (already "uppercase")
- When the shell sees `$0`, it launches the current shell binary (`/bin/sh`) — and we escape

---

## Level 33 — The End! 🎉

```bash
cat README.txt
```

```
Congratulations on solving the last level of this game!

At this moment, there are no more levels to play in this game.
However, we are constantly working on new levels and will most
likely expand this game with more levels soon.
Keep an eye out for an announcement on our usual communication channels!
In the meantime, you could play some of our other wargames.
If you have an idea for an awesome new level, please let us know!
```

**Bandit is complete!** 🏆

---

## 📚 Command Cheat Sheet

| Command | What it does |
|---------|-------------|
| `ssh user@host -p port -i key` | SSH connection (with key) |
| `ls`, `ls -a`, `ls -la` | List files / with hidden / with permissions |
| `cat file` | Print file contents |
| `file file` | Determine file type |
| `find / -user X -group Y -size Nc` | Find files by properties |
| `grep "pattern" file` | Search lines by pattern |
| `sort file \| uniq -u` | Find unique lines |
| `strings file` | Extract readable text from binary |
| `base64 -d file` | Decode Base64 |
| `tr 'A-Za-z' 'N-ZA-Mn-za-m'` | ROT13 decode |
| `xxd -r file` | Hexdump → binary |
| `gzip -d`, `bzip2 -d`, `tar xf` | Decompress archives |
| `nc host port` | TCP connection via netcat |
| `nc -l -p port` | Netcat server/listener mode |
| `openssl s_client -connect host:port` | SSL/TLS connection |
| `nmap -sV host -p range` | Port scanning |
| `diff file1 file2` | Compare two files |
| `chmod 400 key` | Read-only for owner |
| `2>/dev/null` | Suppress error output |
| `command &` | Run in background |
| `git log --oneline` | Compact commit history |
| `git branch -a` | All branches |
| `git show tag/hash` | View commit or tag |

---

## 🔗 What's Next?

After Bandit, I'd recommend these OverTheWire wargames:

- **[Natas](https://overthewire.org/wargames/natas/)** — web security (SQL injection, XSS, LFI...)
- **[Leviathan](https://overthewire.org/wargames/leviathan/)** — Linux privileges, short and sweet
- **[Krypton](https://overthewire.org/wargames/krypton/)** — classical cryptography

Useful resources:
- [ExplainShell](https://explainshell.com/) — explains any shell command visually
- [Linux man pages](https://man7.org/linux/man-pages/) — official documentation
- [RegexOne](https://regexone.com/) — interactive regex tutorial
- [GTFOBins](https://gtfobins.github.io/) — escaping restricted shells via standard utilities

---

*Completing Bandit is a great first step into the world of cybersecurity. The most important thing: don't look up solutions too early — struggle with each level yourself first!*
