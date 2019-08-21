---
title: "SSL Strip Attack"
teaching: 10
exercises: 30
questions:
- "What is SSL Stripping?"
- "How does ARP Poisoning help with SSL Stripping?"
- "What can users do to detect SSL Strip attacks?"
objectives:
- "Successfully conduct a SSL Strip attack."
keypoints:
- "User needs to have accessed the secure site before the attack and needs to return to the website before the browser cache expires"
---

## SSL Strip Attack

The SSL Strip attack is a typical *man-in-the-middle* attack used to circumvent the security enforced by SSL certificates on HTTPS-enabled websites. It's a technique that downgrades your connection from secure HTTPS to insecure HTTP and exposes you to eavesdropping and data manipulation.

The SSL (Secure Sockets Layer) protocol is a transport layer protocol targets to provide communication security and data integrity for internet. Specifically for the website browsing, it's utilized by HTTPS to protect the confidentiality and integrity of website communication with browsers. HTTPS wraps HTTP data into secured SSL packets before sending and receiving via SSL certificates. The use of SSL certificates in HTTPS makes *man-in-the-middle* attacks challenging. However, SSL Strip attacks the effectiveness of HTTPS by stripping the SSL layer.

A hacker conducting this attack intervenes in the redirection of the HTTP to the secure HTTPS protocol. A hacker can start by using ARP poisoning to first intercept communication between the victim and a server. The hacker will then continue to establish an HTTPS connection between themselves and the server, and an unsecured HTTP connection with the user, acting as a “bridge” between them.


### Demonstration

The demonstration on CHEESEHub illustrates the SSL Strip attack using three separate machines; a victim, server, and, hacker. The *sslstrip* python library provided by Moxie (the corresponding the source code from [Moxie's sslstrip github](https://github.com/moxie0/sslstrip)) is used in the hacker to perform a SSL Strip attack on the victim. The hacker conducts a SSL Strip attack by first poisoning the ARP table. The victims are beguiled into believing that the hacker is the server, causing all messages from the victim to the server to be routed through the hacker. The hacker then redirects all HTTPS traffic to HTTP traffic to compromise the SSL-enabled communication to access the authentication information (e.g. username and password). We verify the success of the attack in two ways.

1. We will note whether the server website is being served over HTTP or HTTPS and also verify whether the links embedded in the server website use HTTP or HTTPS.
2. We will look at *sslstrip.log* on the hacker terminal to check if we successfully obtain the username and password.

> ## prerequisites
> 
> You will need to conduct the ARP spoofing attack prior to the SSL Strip attack. If you're not familiar with it, you could refer to [Lesson: ARP Poisoning Attack]({{ page.root }}/02-arpspoof/index.html)
{: .prereq} 

> ## Getting Started
> 
> You will need to create an account on [CHEESEHub](https://www.hub.cheesehub.org) to work through this exercise.
> Each container in this demonstration has a web interface and is accessible through your web browser, no other special software 
> is needed.
{: .callout} 

We will start by first adding the SSLStrip application:

![Add SSLStrip]({{ page.root }}/fig/sslstrip/add-sslstrip.png)

Next, click the *View* link to go to the application-specific page and start the application containers:

![Start Containers]({{ page.root }}/fig/sslstrip/start-containers.png)

> ## Container Launch Errors
>
> If a container fails to start, you will see a flashing warning icon next to the container's name. Take a look at the logs to 
> determine the cause of failure and try deleting the application and restarting.
{: .callout}

Once all the containers have started, launch each container's web interface in a separate browser tab by clicking the icon 
next to the container's name.

We will start with the victim. Let's access the simple website served by the server from the victim's browser:

![Open Victim Browser]({{ page.root }}/fig/sslstrip/victim-firefox.png)

Type the server's IP address in the address bar. This should bring up a simple login page. Log in with any username and password. Note that the address bar indicates that HTTPS is being used. 
In the welcome page, there is also a hyperlink to return to the login page. Once we hover over the hyperlink, the IP address of the server will pop up at the left bottom corner of the browser. 
The URL indicates that the target link is using HTTPS. 

![Confirm HTTPS]({{ page.root }}/fig/sslstrip/victim-https-conf.png)
![Login Hover HTTPS]({{ page.root }}/fig/sslstrip/login-hover-https.png)

> ## HTTPS-based connection
> 
> The connection between the victim and the server is HTTPS-based by default. If a warning indicates this connection is not secure, 
> please ignore it and confirm the security exception. This is because of the self-signed certificates we used.
{: .callout} 

After a successful login, let's switch to the hacker. On the hacker browser tab, click on **Hacker.ipynb** to open a Jupyter notebook to conduct an ARP spoofing attack.

> ## Check whether the ARP attack is successful
>
> Visit the website in victim again to cause the ARP poisoning.
> Then type `arp -n` in the terminal of victim and check the `HWadress` field to confirm if the attack is successful.
{: .callout}

Now, let us conduct a SSL Strip attack which will cause the server website and all communication between the server and victim to employ insecure HTTP.

In the hacker window, click on **SSLStrip.ipynb** to open a Jupyter notebook with instructions on conducting the SSL Strip attack.

Keep the ARP spoofing attack running, the next step is to open a new terminal on the hacker and redirect all HTTPS traffic to a port (e.g. 8080) that SSLStrip will monitor by using the *iptables* command:
~~~
jovyan@sslstriphacker:~$ sudo iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-port 8080
~~~
{: .language-bash}

Still on the newly opened hacker terminal, conduct SSLStrip attack in the background via the *sslstrip* python library:
~~~
jovyan@sslstriphacker:~$ sslstrip -l 8080 &> /dev/null &
~~~
{: .language-bash}

Now, open another firefox browser tab in the victim browser window. Type in the server's IP address in the address bar and log in with username and password.

Notice that this time, the address bar indicates insecure HTTP. Hovering over the hyperlink in the welcome page, indicates that the URL is no longer employing HTTPS.

![Confirm HTTP]({{ page.root }}/fig/sslstrip/victim-http-conf.png)

![Login Hover HTTPS]({{ page.root }}/fig/sslstrip/login-hover-http.png)

> ## HTTP-based connection
> 
> Note that despite the browser indicating that insecure HTTP is being used instead of HTTPS, most users may not notice this warning.
{: .callout} 

Besides, we should be able to see the username and password used in the SSLStrip logs.
~~~
jovyan@sslstriphacker:~$ cat sslstrip.log
~~~
{: .language-bash}
~~~
username=admin&password=test
~~~
{: .output}

> ## SSL strip
> 
> In this lesson, the hacker is able to strip the SSL via ARP poisoning the victim. Once the ARP Poisoning is initiated, the hacker is
> able to intercept the traffic from the victim to the server, acting as the server and possibly collecting confidential information from the victim. 
{: .callout}

{% include links.md %}

