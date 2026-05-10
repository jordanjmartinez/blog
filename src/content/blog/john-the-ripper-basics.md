---
title: 'John the Ripper Basics'
description: 'Beginner notes on John the Ripper. Covering cryptography prerequisites before getting into the tool itself.'
pubDate: 'May 10 2026'
---

Today I started learning about the basics of John the Ripper. It is a popular and well-known hash-cracking tool. I will dive into some pre-requisites and basic knowledge of cryptography terms before I cover the basics of the tool.

## Hashing

Most modern systems do not store user passwords in plain text. Instead, they store a hashed version of the password.

A hash is the result of a one-way mathematical function. You give the function an input, such as a password, and it produces a fixed-length string of characters.

The important part is that hashing is designed to be one-way. You should not be able to take the hash and simply reverse it back into the original password.

For example, if a user’s password is hashed, the system stores the hash instead of the actual password. When the user logs in, the system hashes the password they typed and compares it to the stored hash. If both hashes match, the password is correct.

Hashing is also deterministic. That means the same input will always produce the same output when the same hashing algorithm is used.

So if the password is:

`letmein`

and it is hashed with the same algorithm every time, it will always produce the same hash.

That predictability is what makes password cracking possible.

## Wordlists and rockyou.txt

One common way to crack hashes is with a wordlist.

A wordlist is a file full of possible passwords. John reads each line, hashes it, and checks whether it matches the target hash.

One of the most famous wordlists is:

`rockyou.txt`

`rockyou.txt` came from the 2009 RockYou breach, where millions of user passwords were exposed because they were stored in plain text. The list contains over 14 million real-world passwords.

Because the passwords came from real users, `rockyou.txt` is useful for understanding common password habits. It contains many weak, reused, and predictable passwords based on common patterns.

## Basic John the Ripper Syntax

The basic John syntax looks like this:

`john [options] [file path]`

A common example is running John with a wordlist:

`john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt`

In this command:

- `john` starts John the Ripper.
- `--wordlist=/usr/share/wordlists/rockyou.txt` tells John to use the `rockyou.txt` wordlist.
- `hashes.txt` is the file containing the hash or hashes you want to crack.

John can sometimes detect the hash type automatically, but this does not always work perfectly. If you already know the hash format, it is better to tell John directly.

## Cracking Windows Hashes

Windows systems commonly use password hashes known as NT hashes. You will also see these referred to as NTLM hashes, although technically NTLM is the authentication protocol and the NT hash is the actual stored password hash format.

To crack a Windows NT hash with John, you can use:

`john --format=nt hashes.txt`

If you want to use `rockyou.txt` as the wordlist, the command would look like this:

`john --wordlist=/usr/share/wordlists/rockyou.txt --format=nt hashes.txt`

The important option here is:

`--format=nt`

That tells John the hashes are Windows NT hashes.

## Cracking Linux /etc/shadow Hashes

Linux handles password storage differently.

User account information is stored in:

`/etc/passwd`

Password hashes are stored in:

`/etc/shadow`

The `/etc/passwd` file contains general user account details, such as usernames, user IDs, home directories, and login shells.

The `/etc/shadow` file contains the actual password hashes and other password-related information, such as password expiration details.

Because `/etc/shadow` contains sensitive password hash data, it is normally only readable by root or privileged users.

If you have access to both files, John usually needs them combined into one format before it can crack the hashes properly.

That is where unshadow comes in.

## Using Unshadow

unshadow is a tool included with John the Ripper. It combines the `/etc/passwd` and `/etc/shadow` files into a format John can understand.

The syntax is:

`unshadow [path to passwd] [path to shadow]`

For example:

`unshadow local_passwd local_shadow > unshadowed.txt`

In this example:

- `local_passwd` is a copied version of `/etc/passwd`.
- `local_shadow` is a copied version of `/etc/shadow`.
- The output is saved into `unshadowed.txt`.

That new file can now be passed to John.

## Cracking the Unshadowed File

Once the files are combined, you can run John against the unshadowed file:

`john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt`

Here, John is using the `rockyou.txt` wordlist and treating the hashes as sha512crypt, which is a common Linux password-hashing format.

The command breaks down like this:

- `john` runs the tool.
- `--wordlist=/usr/share/wordlists/rockyou.txt` uses the `rockyou.txt` password list.
- `--format=sha512crypt` tells John what hash format to expect.
- `unshadowed.txt` is the combined passwd and shadow file created with unshadow.

## Final Thoughts

John the Ripper can look intimidating at first, but the basic workflow is straightforward.

You start with a hash. Then John takes password guesses, hashes them, and compares the results against the target hash. If one of the guesses produces the same hash, the password has been cracked.
