#Hacker101 writeups

My write-ups for challenges from the Hacker101 CTF platform. The goal is to earn 26 points to gain access to private HackerOne programs.

## Challenges

- [Micro-CMS v1](./micro-cms-v1.md) — 4/4 flags
- [Micro-CMS v2](./micro-cms-v2.md) — 2/3 flags 
- [Photo Gallery](./photo-gallery.md) — 2/3 flags 

## Progress

- **Points:** 27/26 
- **Status:** Eligible for private programs

# Micro-CMS v1
The first flag is an XSS vulnerability in the header when creating a page.
<img width="815" height="371" alt="screenshot_261002_080341" src="https://github.com/user-attachments/assets/ddffba0a-250a-4a72-aba6-0eaf715022d7" />
We use the most standard payload: <script>alert(1)</script>
<img width="524" height="229" alt="screenshot_261002_080311" src="https://github.com/user-attachments/assets/cf88c20f-a4aa-4711-a026-264ffc117e1b" />
Ultimately, you see the message and realize that the XSS attack was successful.
<img width="679" height="231" alt="screenshot_261002_083241" src="https://github.com/user-attachments/assets/c114b0be-c3c9-4d29-adb4-403249dff551" />
Afterwards, you can view the page content and find the flag there.
<img width="660" height="199" alt="screenshot_261002_080154" src="https://github.com/user-attachments/assets/ed2224f4-4fa4-4aef-9b63-9fcb9a6c0c39" />

The second flag is an SQL injection in the parameter.The second flag is an SQL injection in the parameter.
<img width="480" height="555" alt="screenshot_261002_081533" src="https://github.com/user-attachments/assets/587d28ad-1f7f-4b1d-8b1e-6aabbabd267a" />

I had to hunt for the third flag; you can change the page number in the URL, and the corresponding page is displayed based on the digit—but the page with index 4 returned a "Forbidden" error.
<img width="816" height="218" alt="screenshot_261002_082618" src="https://github.com/user-attachments/assets/da25176b-6d25-4747-b010-de9fd88dc4b2" />
We recall the URL used for the page content modification function and target the page with index 4.
<img width="1285" height="750" alt="screenshot_261002_082519" src="https://github.com/user-attachments/assets/e0436a84-7c6a-448d-8dfe-daeb1561f7ba" />
Flag 4 is also an SQL injection—at least, that’s how I found it.
When creating the page, I submitted a single quote as both the title and the body.
<img width="120" height="149" alt="screenshot_261002_080638" src="https://github.com/user-attachments/assets/8ba3f3ac-0a6e-4240-a82f-929b3d6aacb3" />
<img width="676" height="262" alt="screenshot_261002_080528" src="https://github.com/user-attachments/assets/7f1416eb-afbc-4478-8044-5cf8effe0e34" />
all 4 flags I found

# Micro-CMS v2
When attempting to log in with standard credentials, it returned "invalid user," which opens the door for a brute-force attack.
<img width="327" height="162" alt="screenshot_261002_085127" src="https://github.com/user-attachments/assets/f3964b91-8a0d-48f6-903d-7f1231a942e2" />
And naturally, I started trying SQL injection.
admin’ OR ‘1’=’1 
<img width="310" height="221" alt="screenshot_261002_085102" src="https://github.com/user-attachments/assets/4012f0ec-484e-4985-bdf8-c5225e5a91bd" />
Well, that’s something—the check went through, but now there’s still the password.
After spending a little more time on SQL, I managed to log in.
<img width="1553" height="661" alt="screenshot_261002_085337" src="https://github.com/user-attachments/assets/137e5eeb-f1e3-4e60-b5e2-651d9365e45a" />
but we are met with a message stating that the system is aware we bypassed authentication
In the response, I saw a cookie that looked an awful lot like a JWT token.
<img width="1454" height="300" alt="screenshot_261002_085542" src="https://github.com/user-attachments/assets/670c7e66-5b30-46eb-b56e-73b462028c7f" />
That’s actually what it turned out to be,
but I couldn't do anything useful with it after that.
I kept trying to get in via SQL.
‘ UNION SELECT pass AS password FROM admins WHERE ‘1’=’1
<img width="202" height="218" alt="screenshot_261002_090314" src="https://github.com/user-attachments/assets/b2dbdad6-494c-455b-828d-7b1ed4800a2b" />
And yes, the injection worked, so now we can edit pages; a new hidden page has also appeared, and that’s where the first flag is located.
Also, one of the pages had a message stating that this was a newer version and that logging in was now required to edit pages; however, since we remember the first version and the editing process used there, why not try that here as well?
curl -v -X POST https://xxx.ctf.hacker101.com/page/edit/1 
and we get 2 flags
It comes to us directly in the answer.
    [HTTP/2] [1] [:method: POST] 
    [HTTP/2] [1] [:scheme: https] 
    [HTTP/2] [1] [:authority: 6591218XXXX94.ctf.hacker101.com] 
    [HTTP/2] [1] [:path: /page/edit/1] 
    [HTTP/2] [1] [user-agent: curl/8.22.0] 
    [HTTP/2] [1] [accept: /] 
POST /page/edit/1 HTTP/2 Host: 65912XXXX02794.ctf.hacker101.com User-Agent: curl/8.22.0 Accept: / 
    Request completely sent off < HTTP/2 200 XXX < content-type: text/html; charset=utf-8 < content-length: 76 < server: openresty/1.31.1.1 <  
    Connection #0 to host 659121821ca24ec18085a8b9f3d02794.ctf.hacker101.com:443 left intact ^FLAG^4e7f89bfa009xxxxxxxXXXa7c6ae0dc3d6d3991551519d54c22$FLAG$% 

Unfortunately, I haven't found the third flag yet, but I'm working on it.

# Photo Gallery
At the entrance, we are greeted by several images.
While examining the page source, we notice an interesting parameter.
fetch?id=
And we'll try SQL again, considering that not a single one of my XSS payloads worked there.

This time, I decided to use sqlmap.
sqlmap –u http://x/fetch?id=1 --dbs
<img width="403" height="148" alt="screenshot_261002_113116" src="https://github.com/user-attachments/assets/cc576b68-0205-497f-85e0-7fecbda21317" />
We see several databases and start pulling tables and columns.
sqlmap –u http://x/fetch?id=1 --dbs –-batch 
sqlmap –u http://x/fetch?id=1 -D <Base_name> --tables –-batch 
sqlmap –u http://x/fetch?id=1 -D <Base_name> -T <Table_name> --columns –-batch –-threads=10  
sqlmap –u http://x/fetch?id=1 -D <Base_name> -T <Table_name> --dump –-batch –-threads=10 
after which we get the flag
<img width="956" height="306" alt="screenshot_261002_114439" src="https://github.com/user-attachments/assets/1579dfc5-055e-496f-b4eb-80a441f7818e" />

To get the second flag, I had to use a hint stating that the application runs on the uwsgi-nginx-flask-docker-image.
I decided to study some documentation on uWSGI configuration.
and I found this in this repository: https://github.com/tiangolo/uwsgi-nginx-flask-docker
<img width="589" height="736" alt="screenshot_261002_131623" src="https://github.com/user-attachments/assets/58621bbd-8370-48f6-8ef0-0a50ca99446a" />
<img width="582" height="247" alt="screenshot_261002_131601" src="https://github.com/user-attachments/assets/eaefec3f-4d89-4246-ba7d-047d7b8349d7" />
Thanks to this, I got the flag after just two requests.
<img width="1408" height="773" alt="screenshot_261002_131800" src="https://github.com/user-attachments/assets/bcf31ba7-6c9d-435f-aede-c813cb47a69b" />
<img width="1171" height="573" alt="screenshot_261002_131817" src="https://github.com/user-attachments/assets/f90548c7-8654-4bd4-92fc-64d052a01588" />


You can also find another flag in the image-loading task: simply check the "Network" tab in the developer tools and you'll see that only one file—the image—is being loaded; examine the request body, and the flag will be there.
Based on the results, you will have 27 points, which is enough to receive an invitation to HackerOne's private programs.
