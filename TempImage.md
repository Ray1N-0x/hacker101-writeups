# Hacker101 Writeup — TempImage

> **Target:** TempImage
> **Category:** WEB
> **Difficulty:** Medium 
> **Flags:** 2/2 captured

---

# flag1
Immediately greets you
<img width="436" height="284" alt="Снимок экрана 2026-10-10 174353" src="https://github.com/user-attachments/assets/0c69faa1-5f29-46ca-9eeb-0941d18f06f0" />
I think we need to upload a photo—we might be able to get away with messing around with the file extension—but first, let’s do some thorough reconnaissance.
By the way, Wappalyzer says the site runs on PHP, and this theory is becoming increasingly compelling.
When you click to upload a file, it redirects you to `upload.php`, which contains a script that looks like this...
<img width="505" height="152" alt="Снимок экрана 2026-10-10 174555" src="https://github.com/user-attachments/assets/9a6e74a8-0e53-4945-8ffd-0bea8cadaf38" />
Which indicates that it simply copies the filename into another field.
#filename

However, if this name is subsequently rendered via an unsafe function—such as `.innerHTML`—it results in an XSS (Cross-Site Scripting) vulnerability.
Furthermore, if the server uses the filename exactly as provided when saving the file, and that name contains sequences like `../../../`, it leads to a path traversal vulnerability.
Let’s pause here for now and proceed with testing.
I uploaded a test photo and encountered the following error:
<img width="448" height="137" alt="Снимок экрана 2026-10-10 175205" src="https://github.com/user-attachments/assets/8de541e9-2180-44f8-b497-3e83b4d6df7d" />
I’ll try a double extension.
Just to be safe, I uploaded a PNG file to see what happens next.
<img width="492" height="36" alt="Снимок экрана 2026-10-10 175340" src="https://github.com/user-attachments/assets/ff4ba64b-3921-45b1-8f23-9da8e0369c91" />
We got redirected to /files, which is not bad.
<img width="679" height="189" alt="Снимок экрана 2026-10-10 175429" src="https://github.com/user-attachments/assets/363a6ff9-65de-4784-8095-acd3a6da8650" />
Although the page won't let me in,
the double extension trick didn't work—it’s possible it’s checking the very end of the filename.
I’ll try uploading a PHP file, as well as a file with "xss" in its name.
<img width="561" height="81" alt="Снимок экрана 2026-10-10 175823" src="https://github.com/user-attachments/assets/d2572c5a-e258-4d62-848c-2bc4f6ca12a8" />
File.php produced this result—interesting.
<img width="1166" height="379" alt="Снимок экрана 2026-10-10 180752" src="https://github.com/user-attachments/assets/90476a50-dae6-4ecc-9482-e58c30321c11" />
When attempting to upload a file with "xss" in its name, the same error message appears; the response length is 42, which is exactly the length of the message: "ERROR: Only PNG format supported in trial."

This suggests that the file never reaches the processing stage.

As it stands, it seems that an XSS attack via the filename won't work—apparently, the system doesn't even look at the filename. However, since the site is built on PHP, I’m going to try creating a file that passes the validation—meaning it will be a valid PNG—but contains PHP code inside.
<img width="583" height="144" alt="Снимок экрана 2026-10-10 181627" src="https://github.com/user-attachments/assets/033a49ac-99f3-4c3a-81e8-d7dbeb43805a" />
The most interesting thing is that this file passed the check, yet it still displayed only as an image. This means the validation isn't based on the file extension—after all, for PHP code to execute, it would need a `.php` extension (which it had). It seems they are actually checking the initial bytes of the file, which reveal what kind of file is being uploaded.
<img width="356" height="31" alt="Снимок экрана 2026-10-10 182704" src="https://github.com/user-attachments/assets/a4cd17f1-6e3b-4cf5-9596-8e3a7a2b21ae" />
By the way, here’s our payload at the end of the file.
It turns out the server is serving the file as static content—that’s not good for us.
I’ll try a path traversal attack in the filename.
<img width="410" height="94" alt="Снимок экрана 2026-10-10 184249" src="https://github.com/user-attachments/assets/ddd83755-02ea-4a7c-87ab-e138412b927e" />
Anyway, I spent a long time playing around with this parameter, trying to perform a path traversal, but the filename never changed; so, I decided to check the filename elsewhere.
<img width="1114" height="696" alt="Снимок экрана 2026-10-10 184358" src="https://github.com/user-attachments/assets/6f34462a-5a48-4202-ae19-95fe2fca41bb" />
I scrolled to the very bottom and saw two `Content-Disposition` headers; below them was a filename, which appeared to be encoded in bytes. Since I know the server reads the raw bytes, I changed it to `../../../../picture.png`. This triggered an error, and along with it, the flag appeared.

# flag2
Honestly, for now, I’m considering either XSS via a filename or a PHP script that would somehow need to get executed.
<img width="766" height="519" alt="Снимок экрана 2026-10-10 184715" src="https://github.com/user-attachments/assets/5a8f01e7-db63-486d-8bbb-18fff8a16ca8" />
Hmm... angle brackets are getting encoded.
Let's try to work around this.
When I tried to use URL encoding, it just copied it as-is, treating it as a filename.
<img width="1517" height="664" alt="Снимок экрана 2026-10-10 184927" src="https://github.com/user-attachments/assets/1c4d3d66-5438-4566-8ae8-118144f493b3" />
So, the server isn't decoding it properly.
Remember when we uploaded `shell.php`? I was trying to see if I could execute it.
<img width="987" height="540" alt="Снимок экрана 2026-10-10 190102" src="https://github.com/user-attachments/assets/9964daa9-a725-4d7d-b1fa-89a3a60aa9ce" />
But I ended up with the same flag I already had, so I’m not sure where to look next. I’ll keep trying to exploit the LFI, maybe see if I can read the source code or escalate to RCE—not sure yet.

Ultimately, I decided to start over by injecting a payload into an image file.

I’m going to try and push this vector further.

I started with something simple: `<?php system('id'); ?>`
Forget about the flag—we’ve already got one; the main thing is the redirect.
<img width="1559" height="775" alt="Снимок экрана 2026-10-10 190623" src="https://github.com/user-attachments/assets/219b5779-d9b3-4923-b7a2-724f1317f315" />
<img width="987" height="540" alt="Снимок экрана 2026-10-10 190102" src="https://github.com/user-attachments/assets/37c22c78-011c-437a-97b8-a4794c5fca1b" />
<img width="1525" height="755" alt="Снимок экрана 2026-10-10 193326" src="https://github.com/user-attachments/assets/822f2588-14ff-4222-a539-31173419f2b7" />
<img width="1100" height="681" alt="Снимок экрана 2026-10-10 193249" src="https://github.com/user-attachments/assets/75f3a1e3-c719-446c-bdca-5fc11b089582" />
Where this is no longer just a standard response—as we’ve seen before—but something else entirely.
And where a pleasant surprise awaits at the very bottom.
<img width="473" height="228" alt="Снимок экрана 2026-10-10 193415" src="https://github.com/user-attachments/assets/19e9acf3-1a65-483b-9a7a-97257c363041" />
I’m continuing to explore this vector because it’s an RCE, and I believe it’s possible to execute almost any command—for example, `cat /etc/passwd` and much more.
<img width="486" height="338" alt="Снимок экрана 2026-10-10 193605" src="https://github.com/user-attachments/assets/11ef4923-727b-4a4a-9424-0effd434d2c2" />
Now, as is tradition, let’s take a look at the application’s source code.
<img width="509" height="182" alt="Снимок экрана 2026-10-10 193659" src="https://github.com/user-attachments/assets/a99bdf06-a74e-4b2d-8d6d-5dd270552841" />
This is the output of the payload <?php system(‘ls’); ?>
Followed by <?php system(‘cat index.php’); ?>
<img width="668" height="346" alt="Снимок экрана 2026-10-10 193820" src="https://github.com/user-attachments/assets/a76ad1b9-73c9-4447-9942-317a077b2119" />
As for where we see a flag—well, I was, as usual, looking for something interesting that shouldn't have been there, but oh well—2 flags out of 2.
Thanks, everyone.

---

# Author: Ray1N-0x
