# Hacker101 Writeup — RTFM

> **Target:** RTFM
> **Category:** WEB
> **Difficulty:** Medium 
> **Flags:** 8/8 captured

---

# flag1
<img width="386" height="113" alt="Снимок экрана 2026-10-14 183048" src="https://github.com/user-attachments/assets/7d768aab-278a-4971-8b2f-e2d9fc539b2d" />
Let’s try out different API versions and take a look at the endpoints—let’s go!

While iterating through all possible endpoints under the `/api/v1/...` path...

I find `config`.
<img width="899" height="23" alt="Снимок экрана 2026-10-14 185802" src="https://github.com/user-attachments/assets/214e56f8-8993-4633-9e3d-f0b9c8135510" />
By following which, you get 1 flag
<img width="576" height="98" alt="Снимок экрана 2026-10-14 185852" src="https://github.com/user-attachments/assets/22f00612-3cd6-4a94-9664-502b07d205a3" />

# flag2

By the way, the key has a strange name—"private"—which implies there should be a "public" one.
Anyway, I switched the API version from 1 to 2 and continued fuzzing. I didn't find anything, so I started thinking about the key name and decided to check for potential endpoints, like `swagger.json`.
And sure enough, I managed to find something in version 2.
<img width="1215" height="222" alt="Снимок экрана 2026-10-14 192853" src="https://github.com/user-attachments/assets/a5910fa8-721d-4e41-b9d2-be489cec01f5" />

# flag3
Thanks to this file, we have a complete list of API endpoints.
Next, once we determine which fields—and in what format—an endpoint accepts, we can proceed to register and obtain a flag. We can also check which usernames are already taken; for instance, "admin" is taken.
<img width="976" height="337" alt="Снимок экрана 2026-10-14 193312" src="https://github.com/user-attachments/assets/dcd5babd-e0c8-41d5-addf-0169e8176e54" />

# flag4

Next, we retrieve the cookie from another endpoint.
<img width="939" height="366" alt="Снимок экрана 2026-10-14 193450" src="https://github.com/user-attachments/assets/9865ffff-b635-4473-a7f2-40865c0ba8c5" />
So, by iterating through the endpoints from the file
<img width="926" height="385" alt="Снимок экрана 2026-10-14 194238" src="https://github.com/user-attachments/assets/4a345346-81ae-4383-81f5-efe4c38780ea" />
When I use the post IDs, I get a message stating that the post doesn't belong to me; plus, there are many endpoints that require an admin session.
But there is something interesting.
<img width="516" height="411" alt="Снимок экрана 2026-10-14 194500" src="https://github.com/user-attachments/assets/b632fd57-a838-4ebe-9f53-7684d6210fd6" />
The "Format" value is an MD5 hash of an empty string; I suspect it’s generated from something else.
I’m trying to log in as an admin.
<img width="1154" height="295" alt="Снимок экрана 2026-10-14 195327" src="https://github.com/user-attachments/assets/0d01e2fb-0eaf-427b-be0e-e5415998c25e" />
But what I’m interested in isn’t the flag, but the username—
the one that was already taken.
Nothing I’ve tried has worked.
Basically, in version 1, an `x-Token` header appears during registration, and we are issued a token when we log in.

# flag 5
<img width="1016" height="434" alt="Снимок экрана 2026-10-14 202006" src="https://github.com/user-attachments/assets/b2b159ba-db36-4b42-86d8-373c9a61eed8" />
<img width="1263" height="300" alt="Снимок экрана 2026-10-14 201547" src="https://github.com/user-attachments/assets/3e42b1bf-ab7e-429a-b64d-60e42f621a09" />
I'm doing everything on API version 1.
Version 1 is very different from version 2.
<img width="1096" height="429" alt="Снимок экрана 2026-10-14 202124" src="https://github.com/user-attachments/assets/b731743d-d50c-428a-b801-7c6f38605413" />
At the very least, a PUT request works here.
I started going through all the possible parameters it might accept, and it accepted 'avatar'.
<img width="1320" height="428" alt="Снимок экрана 2026-10-14 202325" src="https://github.com/user-attachments/assets/a3cc0a9f-a3d7-424f-a996-1b6c5f35c0c9" />
Looks like there’s a potential SSRF here.
<img width="1240" height="446" alt="Снимок экрана 2026-10-14 202415" src="https://github.com/user-attachments/assets/a80a61de-91b9-400b-9815-5242b4175bde" />
Well, that’s something, at least.
Let’s try asking it to access its own internal content—for instance, pages that we don’t have access to.
<img width="1162" height="499" alt="Снимок экрана 2026-10-14 202538" src="https://github.com/user-attachments/assets/db91e119-56ae-4502-a408-bd0d4020bdc3" />

# flag6

I took a hint that developers sometimes hide things on a page.
I spent a long time thinking and checking, and eventually started fuzzing the parameters until I found what I was looking for.
<img width="1100" height="361" alt="Снимок экрана 2026-10-14 202946" src="https://github.com/user-attachments/assets/1c00d1c0-7b1f-4945-805c-9bdc886e75cd" />

# flag7
Anyway, regarding version 2, I did some digging and found a post containing a message that doesn't belong to me.
<img width="1155" height="411" alt="Снимок экрана 2026-10-14 203131" src="https://github.com/user-attachments/assets/fc2b1868-e5c7-48bf-9dbd-272c8b2f4cf1" />
I was switching sessions here while testing my hypotheses, but that’s not the main point—earlier versions often have vulnerabilities; I switched the version to 1 and got a response.
<img width="1171" height="394" alt="Снимок экрана 2026-10-14 203301" src="https://github.com/user-attachments/assets/af547e34-b82c-497c-b7e7-5b5399f6b2c2" />

# flag8
Remember how we created a user using version 1 of the API? Well, if you plug that user's token into the `X-session` header...
<img width="1360" height="707" alt="Снимок экрана 2026-10-14 203737" src="https://github.com/user-attachments/assets/eb8cd6e7-6165-4014-9bb9-6e65bba0bfc6" />
If you make a request to this endpoint, you will get something like this, along with a list of users and a flag.
<img width="1027" height="388" alt="Снимок экрана 2026-10-14 204120" src="https://github.com/user-attachments/assets/1189d367-0dc5-48d8-b1c5-026d4c1dbcaa" />
You can get something like a hash.
Let’s try following it.
<img width="1178" height="357" alt="Снимок экрана 2026-10-14 204216" src="https://github.com/user-attachments/assets/ddf9c5ec-b27a-49f1-9532-6042582639a7" />
Nothing interesting.
At this stage, I got stuck because I had no idea where to go or how to proceed. I looked at a couple of write-ups, and almost all of them offered the same solution for this flag.
<img width="1325" height="330" alt="Снимок экрана 2026-10-14 204442" src="https://github.com/user-attachments/assets/b5135aa0-2dc8-4379-af76-0a0b413cdc13" />
<img width="1001" height="334" alt="Снимок экрана 2026-10-14 204409" src="https://github.com/user-attachments/assets/d74ecfc6-4a7f-4c7d-b0d8-ebd40dff6de5" />
Honestly, I’m speechless. This flag has completely stumped me—I was expecting a lot of things, but certainly not this.
Anyway, thanks everyone.

---

# Author: Ray1N-0x 
