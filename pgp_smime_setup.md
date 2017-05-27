## Setting up PGP and S/MIME for Thunderbird (Linux)

>     The following assume you use Thunderbird with Enigmail plugin on
>     Windows, but generating the keys on Linux. And you may want to
>     copy/paste the following into a text file, as it contains TABs...

First, and in order to reduce beginners confusion. Yes, there are 3 widely used command-line tools 
for signing and creating various types of certificates. They are *openssl*, *gpg2* and *ssh-keygen*. 

**What are they used for and when?**

```text
openssl req -newkey : used to create a self-signed S/MIME cert
gpg2 --gen-key      : used to create a PGP keypair
ssh-keygen          : used to create an SSH login certificate
```

##### Examples:

To generate a new 4096 bit RSA key for S/MIME:

```
openssl req -newkey rsa:4096 -keyout alice.p12 -out alice.csr
```

To generate a new PGP cert:

```
gpg2 --full-gen-key --cipher-algo AES256 --digest-algo SHA512 --cert-digest-algo SHA512 -n 
```

To generate a list of random 30 character passwords from command line. Stop with `CTRL-C`.


```bash
while true; do < /dev/urandom tr -dc '[:alnum:]#@%&' | head -c${1:-30};echo; sleep .5; done;
```

---

### References:

##### S/MIME
1. https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs
2. http://www.hoylen.com/articles/it/email/security/cert-comodo.html
3. http://www.hoylen.com/articles/it/email/security/firefox.html
4. http://www.hoylen.com/articles/it/email/security/thunderbird.html
5. http://www.hoylen.com/articles/it/email/security/limitations.html

##### PGP
6. https://ssd.eff.org/en/module/how-use-pgp-windows
7. https://riseup.net/en/security/message-security/openpgp/best-practices
8. https://wiki.ubuntu.com/SecurityTeam/GPGMigration
9. https://help.ubuntu.com/community/GnuPrivacyGuardHowto
10. https://incenp.org/notes/2015/using-an-offline-gnupg-master-key.html
11. https://enigmail.wiki/Key_Management#Verification_of_public_keys.

---

### (1) Creating a new S/MIME cert (using COMODO CA)

Website:

1. https://www.comodo.com/home/email-security/free-email-certificate.php
2. https://secure.comodo.com/products/!SecureEmailCertificate_Signup

```text
FirstName: 	
LastName: 	
Email: 		alice@gmail.com
Country:	Switzerland
Key Size:	High Grade
```

**Export from (Win) Firefox (to import into Thunderbird):**
```text
Goto: 		Tools >> options >> Advanced >> Certificates >> [View Certificates]
Select:		"COMODO CA Limited ID" >> [View...] >> Details >> Subject
Check:  	it's the one with "alice@gmail.com"
Select: 	[Close] >> [Backup...]
Choose: 	<filename>.p12	       # E.g. alice_comodo.p12
```

**To import into Thunderbird (for S/MIME use):**

```text
Goto: 		Tools >> Options >> Advanced >> Certificates >> [Manage Certificates]
Select:		Your Certificates >> [Import...]
Choose: 	<filename>.p12
Enter PP:  	<certificate export password>
```

**Now you can use this cert for S/MIME (only):**
```text
Goto:	Tools >> Account Settings >> Security >> [Select...]
```

---

### (2) Creating a new PGP cert

To use PGP, we need to create a different keypair file that is
made for PGP. But we may want to change the defaults somewhat.

* We want the cipher algorithm to be `AES256`. [default AES]     (cipher used for encryption)
* We want the digest to use at least `SHA512`. [default SHA128]  (hashing used for signatures)
* We also want to set the key expiration to ~1 year.


#### (a) Generate the PGP keys

Use the `-n` flag for a test run that doesn't save anything.
Then when you know what to enter and do, remove the flag.

```
gpg2 --full-gen-key --cipher-algo AES256 --digest-algo SHA512 --cert-digest-algo SHA512 --gen-revoke -n
gpg2 --full-gen-key --cipher-algo AES256 --digest-algo SHA512 --cert-digest-algo SHA512 
```

#### (b) Export the PGP keys (for TB use)


First we need to convert the keys we have created, into something
that Thunderbird/Enigmail can read. But which keys? Both! The public
for sending to other users and the private to decrypt what they send
to you. But in what format? Usually, the easiest format to share is
an ASCII "armored" key. These can also be copy/pasted, but Enigmail 
seem to choke on big copy/paste buffers, since the 4096 bit `*.asc` files
are ~6KB in size. Instead we use the compressed export format without
the armored `-a` flag.

List the keys

```
gpg2 --list-keys
gpg2 --list-secret-keys
```

NOTE: The meaning of the letters are:
>  E=encryption, S=signing,
>  C=certification, A=authentication
You need to see at least SC and E.


**Export the public key(s):**

```
gpg2 -ao pgp-alice-public.asc --export <the-key-id>
```

**Export the private key(s):**

We use the PGP binary format:

```
gpg2 -o pgp-alice-private.gpg --export-secret-keys <the-key-id>
```

Check with:

```bash
file pgp-alice-private.gpg

pgp-emigenix-private.gpg: PGP\011Secret Key - 4096b created on Tue May  9 18:49:38 2017 - RSA (Encrypt or Sign) e=65537 hashed AES with 128-bit key Salted&Iterated S2K SHA-1

```

Now transfer the file(s) from Linux to windows if needed, and delete it after import.


#### (c) Import the PGP keys in Enigmail


To import into Enigmail plug-in for Thunderbird:
```text
Goto:		Enigmail >> Key management >>
Select:		File >> Import Keys from File > "GnuPG Files (*.asc, *.gpg, *.pgp)"
Choose:		<filename>.gpg

Enter PP:  	<certificate export password>
```

If you have problems with the Enigmail GUI, try to import the keys
manually with:

```
gpg2 --import pgp-alice-private.gpg
```

Then check that the key is actually imported into your PGP keychain:

```
gpg2 --list-secret-keys alice
```

Now you can use these Keys for PGP (only) signing, en/decryption:

```text
Goto:	Tools >> Account Settings >> OpenPGP Security >> [Select Key...]
```


#### (d) Send a test email to PGP robot

See: [https://enigmail.wiki/Signature_and_Encryption](https://enigmail.wiki/Signature_and_Encryption)

Now write a signed or PGP encrypted message to: `adele-en@gnupp.de`.


**THAT SHOULD DO IT!**
