# Day 3 — Windows Security Logs and Linux Basics

Today I practiced reading Windows security logs and using basic Linux commands in my Kali virtual machine. My goal was to understand what the logs actually tell me and become more comfortable working in the terminal.

## Windows Security Events

I opened **Event Viewer → Windows Logs → Security** and looked at these event IDs:

| Event ID | Meaning                    |
| -------- | -------------------------- |
| 4624     | Successful logon           |
| 4625     | Failed logon               |
| 4720     | A user account was created |

My way of remembering them:

* **4624:** Login worked.
* **4625:** Login failed.
* **4720:** New account created.

### Investigating a failed login

I intentionally entered an incorrect password using `runas` on my own laptop. Windows returned error **1326**, which means the username or password was incorrect.

At first, I couldn’t find event 4625 in Event Viewer. I checked whether failed-logon auditing was enabled by running this in an Administrator Command Prompt:

```cmd
auditpol /get /subcategory:"Logon"
```

The result showed **Success and Failure**.

I also used this command to display the most recent failed-logon event directly:

```cmd
wevtutil qe Security /q:"*[System[(EventID=4625)]]" /c:1 /rd:true /f:text
```

I eventually found the event in Event Viewer and reviewed these details:

| Field               | My observation                    |
| ------------------- | --------------------------------- |
| Event ID            | 4625                              |
| Logon type          | 2 — Interactive                   |
| Failure reason      | Unknown user name or bad password |
| Status              | `0xC000006D`                      |
| Substatus           | `0xC000006A`                      |
| Failed account name | `-` — not populated               |

The substatus indicated an incorrect password, which matched my test. Since the failed-account field was blank, I couldn’t use that field to identify the attempted account.

Something that surprised me was that this event had the level **Information**, even though the login failed. I learned that filtering only for errors or warnings could hide the event I was looking for.

**Main lesson:** A failed login does not automatically mean someone is attacking the computer. I need to look at the surrounding activity and understand why it happened.

### Creating a test account

Next, I created a temporary account called `SOCPractice`:

```cmd
net user SOCPractice * /add
```

Then I filtered the Security log for **4720** and found the account-creation event.

This helped me understand two important fields:

* **Subject:** The account that performed the action.
* **New Account:** The account that was created.

The cleanup command for this temporary account is:

```cmd
net user SOCPractice /delete
```

I still need to confirm that cleanup is complete. Removing the account does not remove the earlier account-creation event.

## Linux Commands I Practiced

In Kali Linux, I started by checking my location and exploring the log directory:

```bash
pwd
ls
cd /var/log
ls
```

These are the commands I want to remember:

| Command | What I use it for                   |
| ------- | ----------------------------------- |
| `pwd`   | Check which directory I am in       |
| `ls`    | See the files and folders there     |
| `cd`    | Move to another directory           |
| `mkdir` | Create a directory                  |
| `cat`   | Display a file’s contents           |
| `tail`  | Read the last lines of a file       |
| `grep`  | Search for matching lines in a file |

I also tried reading the package-management log:

```bash
sudo tail -n 20 /var/log/dpkg.log
sudo grep -i "install" /var/log/dpkg.log | tail -n 10
```

These commands did not show any output during my session. I did not check the file size, so I couldn’t confirm whether the file was empty.

In the second command, `grep` searches for “install,” and the pipe (`|`) passes the results to `tail`, which shows the last 10 matching lines.

## Creating My Own Practice Log

To practice reading and searching a file, I created a small sample log:

```bash
mkdir -p ~/day3-practice
cd ~/day3-practice
printf 'Login successful\nLogin failed\nUser created\n' > practice.log
```

The file contained:

```text
Login successful
Login failed
User created
```

**This was sample text I created for practice, not real login activity.**

I read the whole file with:

```bash
cat practice.log
```

Then I displayed only the last two lines:

```bash
tail -n 2 ~/day3-practice/practice.log
```

Output:

```text
Login failed
User created
```

Finally, I searched for “failed”:

```bash
grep -n "failed" ~/day3-practice/practice.log
```

Output:

```text
2:Login failed
```

The number **2** tells me the match is on line 2. It does not mean there were two failed logins.

## Mistakes That Helped Me Learn

### Looking for a file in the wrong directory

After running `cd` without a folder name, I returned to my home directory. I then tried:

```bash
cat practice.log
```

I got a “No such file or directory” error because the file was inside `day3-practice`.

This worked:

```bash
cat ~/day3-practice/practice.log
```

I learned that `~` means my home directory, and a relative filename like `practice.log` depends on where I currently am.

### Trying to read a directory with cat

I tried using `cat` on a directory and received an “Is a directory” error.

Now I remember:

* Use **`ls`** to see what is inside a directory.
* Use **`cat`** to read a text file.

### Running cat or tail without a filename

When I entered `cat` or `tail` alone, the terminal seemed to wait. These commands were waiting for keyboard input.

I used **Ctrl+C** to stop them.

### Extra characters from pasting

One pasted command included extra characters and caused a `zsh: bad pattern` error. Re-entering the command without those characters fixed it.

This reminded me to check the actual command before assuming something is wrong with the file.

## How This Helps With SOC Work

This lab gave me basic practice with looking through logs and explaining what I found.

When reviewing an event, I should ask:

* What happened?
* When did it happen?
* Which account performed the action?
* Which account was affected?
* Did the action succeed or fail?
* Is the activity expected?
* Is it happening repeatedly?

A log entry gives me evidence, but I still need context before deciding whether something is suspicious.

## Quick Revision

| Topic     | What I want to remember    |
| --------- | -------------------------- |
| 4624      | Successful logon           |
| 4625      | Failed logon               |
| 4720      | Account created            |
| `pwd`     | Where am I?                |
| `ls`      | What is here?              |
| `cd`      | Move between directories   |
| `cat`     | Read the whole file        |
| `tail`    | Read the end of the file   |
| `grep`    | Search inside the file     |
| `grep -n` | Show matching line numbers |
| `~`       | My home directory          |
| Ctrl+C    | Stop the current command   |

## What I Learned Today

I became more comfortable finding Windows security events and using the Linux terminal. I also learned that small mistakes, like being in the wrong directory, can explain an error.

The biggest takeaway for me was to slow down, read the output carefully, and separate what the evidence shows from what I assume.
