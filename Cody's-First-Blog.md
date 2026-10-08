# Hacker101 Writeup — [Cody's First Blog]

> **Target:** Cody's First Blog
> **Category:** WEB
> **Difficulty:** Medium 
> **Flags:** 3/3 captured

---

#flag1

We arrive and are greeted by this page; we’ll immediately test the comment form for XSS.
<img width="1344" height="559" alt="Снимок экрана 2026-10-08 132435" src="https://github.com/user-attachments/assets/275f7e89-3d9f-4492-b0d2-004710499811" />
We see an interesting endpoint in Burp.
We'll come back to it later.
<img width="570" height="277" alt="Снимок экрана 2026-10-08 132545" src="https://github.com/user-attachments/assets/a7be1a71-aab4-4684-8445-5dda59508546" />
Very interesting—the approval might actually be granted by the admin endpoint we already found.
<img width="342" height="105" alt="Снимок экрана 2026-10-08 132704" src="https://github.com/user-attachments/assets/53b5f494-73c9-4ef6-b4a0-b2fc97444b6f" />
<img width="659" height="466" alt="Снимок экрана 2026-10-08 132815" src="https://github.com/user-attachments/assets/542146c5-a958-4613-a2ef-e270c18c377c" />
Since the site runs on PHP, I'm trying to call up the help section using our `page` parameter.
<img width="1007" height="517" alt="Снимок экрана 2026-10-08 132939" src="https://github.com/user-attachments/assets/821ee2f3-e5be-4223-95f5-d77e098a8c05" />
These are not errors, but warnings indicating an undefined variable `title` in the file `/app/index.php`.
The `include` function failed to open the stream for the file `/app/index.php` because the file or directory does not exist.
This tells us that the application is located in the `/app/` folder.
We can see that it is attempting to include files dynamically, most likely based on a parameter.
<img width="1517" height="593" alt="Снимок экрана 2026-10-08 133801" src="https://github.com/user-attachments/assets/a000f26c-f57d-488e-89ce-fb661f6b0032" />
While attempting an LFI attack, I came across a post where the author explains why they chose this specific approach.
By the way, I forgot to mention the double extension; it might hint at how the developer constructs the file path. I imagine the code looks something like this:
Include($_GET[‘page’] . ‘.php’)
<img width="906" height="274" alt="Снимок экрана 2026-10-08 134832" src="https://github.com/user-attachments/assets/2ef093b0-aa7a-433f-9f73-9aae10c13f23" />
We passed `?page=/app/index`, and the server automatically appended `.php` to the end, as dictated by its code. It then attempted to execute that file; however, since the file was already running, a recursion loop occurred where the script called itself indefinitely. Eventually, the allocated memory was exhausted, and PHP threw this error.
Still, this is a good sign—it means LFI is working and we can include various files.
We’ll use `php://filter`.
This is the most reliable method; it allows us to read the file's source code in an encoded format (such as base64) instead of executing the PHP code itself.
That didn't work for me; I just saw the same message as before, stating that using `include` poses no security risk.
Since the site runs on PHP, I decided to test a specific feature in the post input field:
`<?php system(‘id’); ?>`
And give 1 flag
<img width="1255" height="468" alt="Снимок экрана 2026-10-08 140032" src="https://github.com/user-attachments/assets/da9abece-3f4b-4c5a-b06e-a7953f886f77" />

---

#flag2
?page=admin.auth.inc
`admin.auth.login` looks strange.
I tried removing the middle part and leaving just `admin.auth`.
<img width="800" height="661" alt="Снимок экрана 2026-10-08 141452" src="https://github.com/user-attachments/assets/4ccf0ca8-2be5-406f-badb-79d3d88a7808" />
And yes, it worked—we're in.
We find the flag at the very bottom of the page.
<img width="586" height="229" alt="Снимок экрана 2026-10-08 141530" src="https://github.com/user-attachments/assets/7e4eddc4-0760-4887-a8e6-d01bedc435c5" />
That makes it 2 out of 3.
By the way, we confirmed an XSS vulnerability on the homepage using `<img src=x onerror=alert(...)>`.
<img width="580" height="233" alt="Снимок экрана 2026-10-08 141641" src="https://github.com/user-attachments/assets/3624754c-0627-4749-8a60-8b97f5faeda8" />
But you don't get a flag for that
If we leave a comment on the admin's page
<img width="430" height="77" alt="Снимок экрана 2026-10-08 142340" src="https://github.com/user-attachments/assets/f6524cdb-53d3-4769-8acb-6f9e1c839108" />
I tried reading the application's source code, and it turns out our comment ends up inside `<p></p>`.
I broke out of the tag and pulled off another XSS.
<img width="466" height="104" alt="Снимок экрана 2026-10-08 142515" src="https://github.com/user-attachments/assets/6248d7c5-ec53-43cf-92fd-133cc27a4e80" />
The one that didn't yield a flag
My comments aren't showing up anywhere, but based on previous flags, I assume the server executes PHP code. So, for it to execute, the server likely needs to include my comment via an `include` statement; if it executes alongside the page code—becoming part of the page itself—we effectively introduce a second argument. If the server handles this incorrectly, we might be able to execute code or read a file.

---

#flag3 continuation from flag 2
<img width="1153" height="575" alt="Снимок экрана 2026-10-08 143931" src="https://github.com/user-attachments/assets/750bb800-0573-4ae8-8f72-8c4915d3690e" />
Anyway, after a lot of experimenting, I found an RCE vulnerability. Basically, if you leave a comment containing `php system(...)`, it should execute. The trick is to force the server to call itself—essentially an SSRF-style attack—to reveal the output. You leave a comment like this:
`<?php system('ls -la'); ?>`
Then, you pass `?page=http://127.0.0.1/index` as the `?page` parameter.
You see the execution result because, in effect, you are loading another page.
<img width="489" height="267" alt="Снимок экрана 2026-10-08 144224" src="https://github.com/user-attachments/assets/baa02410-882c-46e0-8f45-6a06ad5e236a" />
Go to the admin panel and skip the comment
<img width="476" height="147" alt="Снимок экрана 2026-10-08 144254" src="https://github.com/user-attachments/assets/51644aeb-4b24-456b-a852-75e1ade9ba3c" />
Now for our mini-SSRF.
Wait, `ls` isn't working for some reason—I must be doing something wrong. But then why did it work with `id`? Or `whoami`?
Yeah... okay, I've got it figured out now. 
<img width="1352" height="200" alt="Снимок экрана 2026-10-08 145004" src="https://github.com/user-attachments/assets/74c0ab34-d372-4ac7-9014-b24c8b1211ad" />
Previously, we were sending the request from the page `?page=localhost/index`, but that didn't work; it needs to be sent from `/`.
Simply send a POST request from the root, then go to the admin panel to confirm. After that, navigate to the `?page=http://localhost/index` parameter and observe the result; below is an example of how I did it.
I also included the decoded payload to make it clearer.
<img width="1362" height="532" alt="Снимок экрана 2026-10-08 145106" src="https://github.com/user-attachments/assets/6d7bf301-80ae-498b-ac3b-eecf6a06103d" />
We know we can break the tag, so why not?
The directory structure looks something like this.
<img width="413" height="223" alt="Снимок экрана 2026-10-08 145358" src="https://github.com/user-attachments/assets/688d2f3b-de7f-4f3a-9219-75105dddb412" />
I also displayed the contents of /etc/passwd for illustrative purposes.
<img width="702" height="408" alt="Снимок экрана 2026-10-08 145518" src="https://github.com/user-attachments/assets/79d97759-5a6c-481b-a826-34bf4e22a67b" />
And now, finally, I'll be able to read index.php.
I couldn't do it using `cat`, so I'm trying a different approach.
<img width="453" height="155" alt="Снимок экрана 2026-10-08 150209" src="https://github.com/user-attachments/assets/4675259b-9f65-42ee-9d29-c84c450916e3" />
Using echo
<img width="891" height="129" alt="Снимок экрана 2026-10-08 150657" src="https://github.com/user-attachments/assets/7507d992-6d04-4787-b1b0-2ff6768c81e3" />
OK
Since I was unable to read index.php using various methods, I read setup.sh.
<img width="1116" height="91" alt="Снимок экрана 2026-10-08 151200" src="https://github.com/user-attachments/assets/dc962844-ceba-46dd-b611-5c673e87548f" />
The flag was stored in a special environment variable named `FLAGS` during the execution of `setup.sh`.
To retrieve the third flag, all that remains is to read the environment variables using `env`.
<img width="1349" height="173" alt="Снимок экрана 2026-10-08 151540" src="https://github.com/user-attachments/assets/85a75f30-16b2-498d-a6fe-720d6952fa47" />
Hmm... I'll try find to find the flag.
<img width="1052" height="96" alt="Снимок экрана 2026-10-08 152034" src="https://github.com/user-attachments/assets/9697491f-c35a-4578-bc97-94b500af1635" />
There are a few matches.
I'm going to check them.

I went through all the files and directories and didn't find the flag, but I realized I hadn't actually checked the `index.php` code. I started recalling which methods I had tried and which I hadn't, and then I began...
checked everything, and it worked
<img width="383" height="122" alt="Снимок экрана 2026-10-08 152858" src="https://github.com/user-attachments/assets/11da68a5-01a2-403c-b3ff-6c5087bd8a32" />
After that, I started looking at the code, where I found the flag in a comment.
<img width="954" height="480" alt="Снимок экрана 2026-10-08 152932" src="https://github.com/user-attachments/assets/f4bc57ec-9f37-4511-9252-fb54646b9a6b" />
All 3 out of 3 flags.
The key might not even have been RCE—but I couldn't read `index.php` right away, so I pursued a different vector, which was a mistake; had I followed through on the initial lead, I would have found the flag faster and saved time, but things turned out the way they did.
Thanks everyone; that was the methodology I used to complete this machine.

---

#Author: Ray1N-0x
