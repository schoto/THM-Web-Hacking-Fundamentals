<h4>Introduction</h4>

This room breaks each OWASP topic down and includes details on the vulnerabilities, how they occur, and how you can exploit them. You will put the theory into practice by completing supporting challenges.

1.Broken Access Control

2.Cryptographic Failures

3.Injection

4.Insecure Design

5.Security Misconfiguration

6.Vulnerable and Outdated Components

7.Identification and Authentication Failures

8.Software and Data Integrity Failures

9.Security Logging & Monitoring Failures

10.Server-Side Request Forgery (SSRF)

The room has been designed for beginners and assumes no previous security knowledge.

<h4>Broken Access Control</h4>

Websites have pages that are protected from regular visitors. For example, only the site's admin user should be able to access a page to manage other users. If a website visitor can access protected pages they are not meant to see, then the access controls are broken.

A regular visitor being able to access protected pages can lead to the following:

- Being able to view sensitive information from other users

- Accessing unauthorized functionality

Simply put, broken access control allows attackers to bypass authorisation, allowing them to view sensitive data or perform tasks they aren't supposed to.

For example, a vulnerability was found in 2019, where an attacker could get any single frame from a Youtube video marked as private. The researcher who found the vulnerability showed that he could ask for several frames and somewhat reconstruct the video. Since the expectation from a user when marking a video as private would be that nobody had access to it, this was indeed accepted as a broken access control vulnerability.

<h4>Broken Access Control (IDOR Challenge)</h4>

**Insecure Direct Object Reference**

IDOR or Insecure Direct Object Reference refers to an access control vulnerability where you can access resources you wouldn't ordinarily be able to see. This occurs when the programmer exposes a Direct Object Reference, which is just an identifier that refers to specific objects within the server. By object, we could mean a file, a user, a bank account in a banking application, or anything really.

For example, let's say we're logging into our bank account, and after correctly authenticating ourselves, we get taken to a URL like this ```https://bank.thm/account?id=111111```. On that page, we can see all our important bank details, and a user would do whatever they need to do and move along their way, thinking nothing is wrong.

![idor1](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/f8301989-b0c9-47fd-a400-00d0a6c5ad8b)

There is, however, a potentially huge problem here, anyone may be able to change the ```id``` parameter to something else like ```222222```, and if the site is incorrectly configured, then he would have access to someone else's bank information.

![idor2](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/26bbd8ac-6eac-470d-add0-571e9df0a80d)

The application exposes a direct object reference through the ```id``` parameter in the URL, which points to specific accounts. Since the application isn't checking if the logged-in user owns the referenced account, an attacker can get sensitive information from other users because of the IDOR vulnerability. Notice that direct object references aren't the problem, but rather that the application doesn't validate if the logged-in user should have access to the requested account.

**Questions / Answers**

Read and understand how IDOR works.

Done

Deploy the machine and go to http://10.10.87.163 - Login with the username noot and the password test1234.

![idro4](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/16004b29-44b3-47c5-81ce-79c0e5af227f)

Look at other users' notes. What is the flag?

![idro5](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/6f2555bf-1ac1-4b01-b325-f125205b91f6)

```flag{fivefourthree}```

<h4> 2. Cryptographic Failures</h4>

**Cryptographic Failures**

A cryptographic failure refers to any vulnerability arising from the misuse (or lack of use) of cryptographic algorithms for protecting sensitive information. Web applications require cryptography to provide confidentiality for their users at many levels.

Take, for example, a secure email application:

- When you are accessing your email account using your browser, you want to be sure that the communications between you and the server are encrypted. That way, any eavesdropper trying to capture your network packets won't be able to recover the content of your email addresses. When we encrypt the network traffic between the client and server, we usually refer to this as encrypting data in transit.

- Since your emails are stored in some server managed by your provider, it is also desirable that the email provider can't read their client's emails. To this end, your emails might also be encrypted when stored on the servers. This is referred to as encrypting data at rest.

Cryptographic failures often end up in web apps accidentally divulging sensitive data. This is often data directly linked to customers (e.g. names, dates of birth, financial information), but it could also be more technical information, such as usernames and passwords.

At more complex levels, taking advantage of some cryptographic failures often involves techniques such as "Man in The Middle Attacks", whereby the attacker would force user connections through a device they control. Then, they would take advantage of weak encryption on any transmitted data to access the intercepted information (if the data is even encrypted in the first place). Of course, many examples are much simpler, and vulnerabilities can be found in web apps that can be exploited without advanced networking knowledge. Indeed, in some cases, the sensitive data can be found directly on the web server itself.

The web application in this box contains one such vulnerability. To continue, read through the supporting material in the following tasks.

<h4>Cryptographic Failures (Supporting Material 1)</h4>

The most common way to store a large amount of data in a format easily accessible from many locations is in a database. This is perfect for something like a web application, as many users may interact with the website at any time. Database engines usually follow the Structured Query Language (SQL) syntax.

In a production environment, it is common to see databases set up on dedicated servers running a database service such as MySQL or MariaDB; however, databases can also be stored as files. These are referred to as "flat-file" databases, as they are stored as a single file on the computer. This is much easier than setting up an entire database server and could potentially be seen in smaller web applications. Accessing a database server is outwith the scope of today's task, so let's focus instead on flat-file databases.

As mentioned previously, flat-file databases are stored as a file on the disk of a computer. Usually, this would not be a problem for a web app, but what happens if the database is stored underneath the root directory of the website (i.e. one of the files accessible to the user connecting to the website)? Well, we can download and query it on our own machine, with full access to everything in the database. Sensitive Data Exposure, indeed!

That is a big hint for the challenge, so let's briefly cover some of the syntax we would use to query a flat-file database.

The most common (and simplest) format of a flat-file database is an SQLite database. These can be interacted with in most programming languages and have a dedicated client for querying them on the command line. This client is called ```sqlite3``` and is installed on many Linux distributions by default.

Let's suppose we have successfully managed to download a database:

```
user@linux$ ls -l 
-rw-r--r-- 1 user user 8192 Feb  2 20:33 example.db
                                                                                                                                                              
user@linux$ file example.db 
example.db: SQLite 3.x database, last written using SQLite version 3039002, file counter 1, database pages 2, cookie 0x1, schema 4, UTF-8, version-valid-for 1
```

We can see that there is an SQLite database in the current folder.

To access it, we use ```sqlite3 <database-name>```:

```
user@linux$ sqlite3 example.db                     
SQLite version 3.39.2 2022-07-21 15:24:47
Enter ".help" for usage hints.
sqlite> 
```

From here, we can see the tables in the database by using the ```.tables``` command:

```
user@linux$ sqlite3 example.db                     
SQLite version 3.39.2 2022-07-21 15:24:47
Enter ".help" for usage hints.
sqlite> .tables
customers
```

At this point, we can dump all the data from the table, but we won't necessarily know what each column means unless we look at the table information. First, let's use ```PRAGMA table_info(customers)```; to see the table information. Then we'll use ```SELECT * FROM customers```; to dump the information from the table:

```
sqlite> PRAGMA table_info(customers);
0|cudtID|INT|1||1
1|custName|TEXT|1||0
2|creditCard|TEXT|0||0
3|password|TEXT|1||0

sqlite> SELECT * FROM customers;
0|Joy Paulson|4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99
1|John Walters|4671 5376 3366 8125|fef08f333cc53594c8097eba1f35726a
2|Lena Abdul|4353 4722 6349 6685|b55ab2470f160c331a99b8d8a1946b19
3|Andrew Miller|4059 8824 0198 5596|bc7b657bd56e4386e3397ca86e378f70
4|Keith Wayman|4972 1604 3381 8885|12e7a36c0710571b3d827992f4cfe679
5|Annett Scholz|5400 1617 6508 1166|e2795fc96af3f4d6288906a90a52a47f
```


We can see from the table information that there are four columns: ```custID```, ```custName```, ```creditCard``` and ```password```. You may notice that this matches up with the results. Take the first row:

```0|Joy Paulson|4916 9012 2231 7905|5f4dcc3b5aa765d61d8327deb882cf99```

We have the custID (0), the custName (Joy Paulson), the creditCard (4916 9012 2231 7905) and a password hash (5f4dcc3b5aa765d61d8327deb882cf99).

In the next task, we'll look at cracking this hash.

<h4>Cryptographic Failures (Supporting Material 2)</h4>

We saw how to query an SQLite database for sensitive data in the previous task. We found a collection of password hashes, one for each user. In this task, we will briefly cover how to crack these.

When it comes to hash cracking, Kali comes pre-installed with various tools. If you know how to use these, then feel free to do so; however, they are outwith the scope of this material.

Instead, we will be using the online tool: Crackstation. This website is extremely good at cracking weak password hashes. For more complicated hashes, we would need more sophisticated tools; however, all of the crackable password hashes used in today's challenge are weak MD5 hashes, which Crackstation should handle very nicely.

When we navigate to the website, we are met with the following interface:

![cryptog1](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/6392bbb4-fb89-4112-8d92-c4f6307181e8)

Let's try pasting the password hash for Joy Paulson, which we found in the previous task (```5f4dcc3b5aa765d61d8327deb882cf99```). We solve the Captcha, then click the "Crack Hashes" button:

![cryptog2](https://github.com/schoto/THM-Web-Hacking-Fundamentals/assets/69323411/00cfb3a0-e9de-4a0e-94b6-44f84f80edc2)

We see that the hash was successfully broken, and the user's password was "password". How secure!

It's worth noting that Crackstation works using a massive wordlist. If the password is not in the wordlist, then Crackstation will not be able to break the hash.

The challenge is guided, so if Crackstation fails to break a hash in today's box, you can assume that the hash has been specifically designed not to be crackable.

<h4>Cryptographic Failures (Challenge)</h4>

It's now time to put what you've learnt into practice! For this challenge, connect to the web application at http://MACHINE_IP:81/.

**Questions / Answers**

Have a look around the web app. The developer has left themselves a note indicating that there is sensitive data in a specific directory. 

What is the name of the mentioned directory?

```/assets```

Navigate to the directory you found in question one. What file stands out as being likely to contain sensitive data?

```webapp.db```

Use the supporting material to access the sensitive data. What is the password hash of the admin user?

```6eea9b7ef19179a06954edd0f6c05ceb```

Crack the hash.

What is the admin's plaintext password?

```qwertyuiop```

Log in as the admin. What is the flag?

```THM{Yzc2YjdkMjE5N2VjMzNhOTE3NjdiMjdl}```
