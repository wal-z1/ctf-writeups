# Gamer Challenge

**Category:** Forensics / Disk Analysis
**Flag:** `SK-CERT{4lm057_5Ucc355fUl_53cR37_F1l3_7r4n5F3R}`

---

## Description

Niki was supposed to finish a client assignment.
Instead, the evidence points to something else: an encrypted archive, a personal mailbox, and a gaming profile used as a secret communication channel.

The challenge gives you a segmented AD1 disk image. Somewhere inside it is proof of the file transfer and the password needed to open the archive.

The important part is not just finding the flag.
It is proving how the file moved, where the password came from, and why the whole thing was almost successful.

---

## Step 1

Before jumping into artifacts ... identify what you actually have.
It is a segmented AD1 forensic disk image.

Extract the provided archive first, then load the AD1 image into FTK Imager.

```text
Evidence archive -> extracted AD1 segments -> loaded into FTK Imager -> mounted/exported filesystem
```

![Screenshot: extracted evidence folder showing segmented AD1 image files](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/01_ad1_segments_placeholder.png)

Once loaded into FTK Imager and mounted, the user profile becomes visible:

```text
C:\Users\niki\
```

![Screenshot: FTK Imager showing C:\Users\niki profile directory](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/02_user_profile_placeholder.png)

That gives the investigation a clear target.
From here on, the question becomes simple:

```text
What did niki do?
Where did the file go?
Where is the password?
```

---

## Step 2

Use FTK Imager to browse installed applications, user activity, and anything that hints at file movement.

Several artifacts stand out early:

```text
Steam installed
Microsoft Office installed
Outlook configured
VMware artifacts present
Browser history showing WinRAR and Discord activity
```

![Screenshot: FTK Imager overview showing installed applications and user artifacts](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/03_initial_artifacts_placeholder.png)

This already gives a few leads:

```text
Steam may be the communication channel
Outlook may contain the transfer
WinRAR or 7-Zip may explain archive creation
Discord a second channel
```

---

## Step 3

The Steam artifacts are the first real lead.
Use FTK Imager to navigate to the Steam installation directory and open the config and log files. An account identifier appears:

```text
[U:1:773515074]
```

![Screenshot: Steam config or log file showing [U:1:773515074]](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/04_steam_accountid_placeholder.png)

That value is the Steam AccountID.
To reconstruct the public SteamID64, use the standard offset:

```text
SteamID64 = 76561197960265728 + AccountID
SteamID64 = 76561197960265728 + 773515074
SteamID64 = 76561198733780802
```

Now build the profile URL:

```text
https://steamcommunity.com/profiles/76561198733780802
```

The profile is public.
The username is `NiKo` and the profile summary contains the password.

---

## Step 4

The Steam profile summary is the break in the case.
It says:

```text
pw for my file:
N6bVJjF4BqTeWW7wNKnAHJxc2r2jpLgcMxXEvH34yDKtXBw8wmLS%CsFpNYTMCfkFy8#m&9nVtox4#^7w2CNQUxRw!8w!7wyaDQ5vBi^8Cemy
```

![Screenshot: Steam profile summary showing the password text](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/06_steam_password_placeholder.png)

That line changes the direction of the investigation.
This is not just gaming activity anymore.

The working hypothesis now writes itself:

```text
Niki used Steam as the password channel
An encrypted file exists somewhere in the evidence
The password on the Steam profile unlocks that file
The file transfer likely happened through Outlook
```

---

## Step 5

Next, check Outlook.
Use FTK Imager to navigate to the AppData directory. A local Outlook mailbox file is present:

```text
C:\Users\niki\AppData\Local\Microsoft\Outlook\mr.niki.123@hotmail.com.ost
```

![Screenshot: FTK Imager showing Outlook directory with mr.niki.123@hotmail.com.ost](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/07_ost_file_placeholder.png)

This confirms a personal Hotmail account was configured on the machine.
That matters because personal email is a clean path for taking files out of a controlled environment.

Export the OST file from FTK Imager, then process it using `libpff`:

```bash
pffexport -f html -m all -t /tmp/ost_export mr.niki.123@hotmail.com.ost
```

![Screenshot: pffexport output showing successful OST mailbox export](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/08_pffexport_placeholder.png)

The mailbox export gives you message folders, HTML bodies, and extracted attachments.
Now search the exported mailbox for archives.

---

## Step 6

The archive is found in Sent Items:

```text
/tmp/ost_export.export/Root - Mailbox/IPM_SUBTREE/Sent Items/Message00002/Attachments/1_reports.7z
```

![Screenshot: exported OST folder showing Sent Items Message00002 attachment 1_reports.7z](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/09_sent_attachment_placeholder.png)

That location is important.

Sent Items.

The file was attached to an outbound email.
That turns the archive from a suspicious object into transfer evidence.

```text
Archive name: 1_reports.7z
Mailbox: mr.niki.123@hotmail.com
Folder: Sent Items
Meaning: outbound file transfer
```

![Screenshot: sent email body showing 1_reports.7z attached](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/10_sent_email_placeholder.png)

Now test the password recovered from Steam.

---

## Step 7

Use the Steam profile password against the archive:

````

That confirms the Steam profile password belongs to the archive.
Extract it:

```bash
7z x -p'N6bVJjF4BqTeWW7wNKnAHJxc2r2jpLgcMxXEvH34yDKtXBw8wmLS%CsFpNYTMCfkFy8#m&9nVtox4#^7w2CNQUxRw!8w!7wyaDQ5vBi^8Cemy' 1_reports.7z -o/tmp/reports_unpacked
````

The archive drops a folder `reportfiles/` containing `f4.txt`.

---

## Step 8

Open the recovered file:

```bash
cat /tmp/reports_unpacked/reportfiles/f4.txt
```

![Screenshot: terminal showing f4.txt contents with the flag](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/13_flag_placeholder.png)

```
SK-CERT{4lm057_5Ucc355fUl_53cR37_F1l3_7r4n5F3R}
```
