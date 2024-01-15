# Cybertalent CTF 2023

Etteretningstjenesten 2023 CTF (https://ctf.cybertalent.no/)


The CTF consists of the following tasks:

1. Grunnleggende / Basics
- 1.1_scoreboard
- 1.2_setuid
- 1.3_injection
- 1.4_overflow
- 1.5_nettverk
- 1.6_reversing
- 1.7_path_traversal
- 1.8_path_traversal_bonus

2. Initiell aksess / Initial access
- 2.0.1_manipulaite_1
- 2.0.2_anvilticket_1
- 2.0.3_anvilticket_2
- 2.0.4_manipulaite_2
- 2.0.5_pcap
- 2.0.6_dep-gw

- 2.1. Department of Development and Test
- 2.2. Department of Cryptography
- 2.3. Department of Research
- 2.4. Department of Intelligence
- 2.5. Department of Security

---

This write up covers parts of the Initial access and some of the tasks of the different departments.

The context of the initial access can be found [here](https://github.com/simonsynnes/Cybertalent-CTF-2023/blob/main/INTREP.txt)

# 2.0_1_manipulaite_1

This task consisted of social engineering training available on port 8880
Connected with the following command:

```jsx
nc manipulaite 8880
```
After some persuation I was handed the flag.

# 2.0.2_anvilticket_1

The inital website recon findings showed that there was 4 different pages.
A login page, a sign up page, an about page and a page for tickets.
To get access to the tickets page I was redirected to the login page.

I Tried to login with some default credentials first, this did not work.

 Then I signed up on the website with a new account and gained access to the ticket system.

---

The only ticket found was made by the admin telling us that our employer needs to add us to the correct group (hint).

```jsx
Welcome to AnvilTicket!
Please get your current employer to add you to your correct group so you can get access to your tickets!

- Admin
```

Found that it was a flask application by checking Wappalyzer.
Checked some commonly found vulnerabilities for flask, also including common web vulnerabilities like XSS/SQLi/CSRF etc.

Decoded the session token:

```jsx
Session decodes to {'admin': False, 'groupid': 0, 'username': 'testuser'}
```

Tried unsigning the session cookie by brute forcing the secret, this did not work.

After some further enumeration inside the ticket system I found that there was a comment refetch every 5 seconds going on. That made me look further into comments and how they are set up.
I started considering *Insecure Direct Object References* (IDOR) was a possibility and it turned out to be correct.

```jsx
https://anvilticket.cybertalent.no/comment/7
```

This comment had the following information:

```jsx
{"count": 1, "comments": [["admin", "Hi Carla,\n\nNew user account created:\nthenewguy:FLAG{...}\n\nPlease share these credentials securely with our Eva. Let us know if further assistance is needed.\n\nThanks,\nAdmin"]]}
```

A flag and and a pair of credentials were given to us.

# 2.0.3_anvilticket_2

Tried to login with these login credentials, these were valid 🙂

The user we just got access to called thenewguy was part of the IT group and had more tickets available in the list, mostly IT related. 
It also had access to update the password of the user.

We could enter a different username, tried with **admin** as it was previously found to be a valid user. This did not work. 

Tried to run the request through **Burp Suite** to see what happens.

```jsx
POST /update_user HTTP/1.1
Host: anvilticket.cybertalent.no
Cookie: userauth=ey... session=...
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 777
Origin: https://anvilticket.cybertalent.no
Referer: https://anvilticket.cybertalent.no/update_user
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
Connection: close

username=thenewguy&password=FLAG%7B41%7D&password2=HTTP%2F1.1+302+FOUND++server%3A+Werkzeug%2F3.0.1+Python%2F3.11.2++date%3A+Sat%2C+06+Jan+2024+21%3A49%3A44+GMT++content-type%3A+text%2Fhtml%3B+charset%3Dutf-8++content-length%3A+223++location%3A+%2Ftickets%3Fupdate_ok++vary%3A+Cookie++set-cookie%3A+session%3DeyJhZG1pbiI6ZmFsc2UsImdyb3VwaWQiOjIsInVzZXJuYW1lIjoidGhlbmV3Z3V5In0.ZZnK-A.KyFVJFYRFZPn0Z2YAz6R6bwDkjc%3B+HttpOnly%3B+Path%3D%2F++connection%3A+close+++%3C%21doctype+html%3E+%3Chtml+lang%3Den%3E+%3Ctitle%3ERedirecting...%3C%2Ftitle%3E+%3Ch1%3ERedirecting...%3C%2Fh1%3E+%3Cp%3EYou+should+be+redirected+automatically+to+the+target+URL%3A+%3Ca+href%3D%22%2Ftickets%3Fupdate_ok%22%.
```

Added **&admin=1** to the request, which gave us admin access and a new flag.

---

# 2.0.5_pcap

We now gained access to more tickets.

One contained the following information:

```jsx
Investigate data related to an alarm from our IDS. 
A ticket for the record with drop.pcap as attachment.
Its a small sample, propably false-positive as last time...
https://anvilticket.cybertalent.no/assets/anomaly/drop.pcap
```

Analyzed the pcap file and eventually got another flag.

---

After checking one of the other new tickets I found an AI chat bot that was responding to comments.

After persuading the bot to give us a flag in clear text, it was the answer to **2.0.4_manipulaite_2**.

---

# 2.0.6_dep-gw

After gaining the flag from **2.0.4_manipulaite_2,** we could now create the ****id_ed25519**** private key needed to connect through ssh to get the flag for this task.

We used the command “ssh gw” as the following config file was present:

```jsx
Host gw dep-gw.utl
    HostName dep-gw.utl
    User preyz
    IdentityFile ~/.ssh/id_ed25519
```

The next flag was found through this connection.

---

I had now unlocked all of the  different departments.

I started off with the **Department of Security**.

# 2.5.1_passftp

```jsx
# passFTP

*Department of security* har generelt vært ganske nedlukket og utilgjengelig nettverksmessig, det virker som de har en høy grad av operasjonssikkerhets-fokus her.
Det er derimot oppdaget en større sikkerhetsglipp, og deres passFTP eksponeres mot dette felles departements-nettverket. Den kan kontaktes på `passftp.utl:1024`
Vi har informasjon om at denne serveren tidligere har blitt brukt til å distribuere passFTP sin kildekode, se om det er mulig å ekstrahere denne til videre analyse.
```

When connecting to the share on port 1024, it prompted for a username and password. 

Tried with different standard credentials as admin/admin etc.
Anonymous login was enabled so I proceeded with this.

A flag was found together with a share called **passFTP_shared**.

This share was password protected and the provided text hinted that there should be source code on this folder.

 Tried the following:

```jsx
cd passFTP_shared/src
```

It bypassed the password requirement and we could find the source code and a binary for a C file in this directory. The source code was for this FTP server.

The password for the directory was found in plain text after checking the source code.

Analyzed this with GDB (Gnu Project Debugger) to find out how the program works on a lower level.

---

Headed over to the **Department of Development and Test**.
We were directed to a website where the tasks consists of creating a type of assembly code to do certain stuff.

# 2.1.1_hello

Write a program that prints out "MOV to the cloud!", without
the quotation marks. The line must end with a newline
character.

The configuration must be compatible with the standard
configuration.

```jsx
DBG <- ptr
loop:
!*ptr ? NIP <- #HLT
        PRN <- *ptr
        INC <- ptr
        ptr <- INC
        NIP <- #loop

ptr:    string
string: "MOV to the cloud!",10,0
```
