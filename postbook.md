# Postbook

**Difficulty:** Easy
**Flags:** 7/7
**Date:** October 2026
**Platform:** Hacker101 CTF


---

#flag1
As soon as I logged in, I started looking at the page source and found some interesting endpoints.
<img width="654" height="457" alt="screenshot_261002_134625" src="https://github.com/user-attachments/assets/7ec979cc-1f7b-43a8-b935-3418591c222b" />
Then I decided to fuzz the directories using ffuf.
<img width="510" height="525" alt="screenshot_261002_134949" src="https://github.com/user-attachments/assets/b35c7ae4-abb6-41e3-9214-ff6e49b4a8aa" />
<img width="186" height="119" alt="screenshot_261002_134958" src="https://github.com/user-attachments/assets/9ee04575-d9e8-45e8-946b-8b5b4d035198" />
Interesting endpoint, though many files didn't open at all.
I created an account, started exploring what was there, and stumbled upon the post creation feature.
<img width="344" height="550" alt="screenshot_261002_135638" src="https://github.com/user-attachments/assets/2c40b1ef-05bb-412b-ae82-caf535eee7d7" />
I wrote a test post and saw that our text was hitting an interesting endpoint ID.
<img width="448" height="43" alt="screenshot_261002_135741" src="https://github.com/user-attachments/assets/e4dd5db5-b4ed-40ba-a28b-0b5381e37feb" />
I decided to change the numbers in the parameter, hoping to see other people's posts—and success!
<img width="1014" height="551" alt="screenshot_261002_135902" src="https://github.com/user-attachments/assets/3bb9dba5-d732-467d-a3ea-8036c930c62d" />

---

#flag2
The homepage has a feature for writing quick posts—you know, "how are things" and all that—and naturally, we’re trying XSS there.
<img width="691" height="283" alt="screenshot_261002_142709" src="https://github.com/user-attachments/assets/1fc00be8-18a9-4a5e-b183-feb9d9738750" />
<img width="1436" height="821" alt="screenshot_261002_142615" src="https://github.com/user-attachments/assets/484e90e0-5f7a-49be-9887-ad3042cb87c8" />
and we get 2 flags

---

#flag3
Later, I found an XSS vulnerability when deleting a post, though there’s no flag awarded for that.

I suspect this XSS might stem from the previous flag on the home page—I’m not certain, but I’ll definitely look into it.
<img width="788" height="822" alt="screenshot_261002_143059" src="https://github.com/user-attachments/assets/aa8721a7-ab36-4942-ab1b-9f07f291f2a7" />
As we continue browsing and changing the `id` parameter, we come across a post by a certain user; why not try logging in as that user?
<img width="1075" height="401" alt="screenshot_261002_143750" src="https://github.com/user-attachments/assets/cd6c3958-d008-4a97-867d-3db9b37f192c" />
After logging out and logging back in with the `user:password` credentials—essentially relying on luck—we successfully access the account.

We see the flag confirming that we are logged in as a different user; we have a button to delete their post, and, of course, our XSS payload executed upon login.
<img width="902" height="695" alt="screenshot_261002_143934" src="https://github.com/user-attachments/assets/a71ba320-dcc2-4e24-b0a9-f6d76af8c8fc" />

---

#flag4
That’s when I decided to take a hint, and it turned out to be a good move—it really surprised me, and had I not known that, I definitely wouldn't have bothered with it.
<img width="366" height="83" alt="screenshot_261002_144423" src="https://github.com/user-attachments/assets/fe075d94-13e8-4ad3-a288-fa076de8cd10" />
I definitely wouldn't have fuzzed that many parameters, but it's a good thing it was there.
<img width="1279" height="445" alt="screenshot_261002_144545" src="https://github.com/user-attachments/assets/9856182d-11fc-4320-a90d-6ff597c488f9" />

---

#flag5
To be honest, I really feel like it’s possible to add a custom parameter, though I haven't managed to get anything worthwhile out of it so far.

Then I remembered we can edit our own posts and wondered—what if...? Is it possible to edit other people's posts?
<img width="1113" height="885" alt="screenshot_261002_145108" src="https://github.com/user-attachments/assets/47558bd1-d95d-4221-82b0-e610dba6c06f" />
It turns out that unchecking the box is another potential way to get the flag.
After clicking the "Save post" button, you will see the flag.

---

#flag6
I finally got around to the cookies (specifically the `id` one); they had been bothering me from the start, but I hadn't known what to do about them. After Googling, I realized it was an MD5 hash of the number 2, so let's try changing it to a hash of other numbers—like 1, 3, 4, 5, and so on.
We simply take the new MD5 hash for 1, 3, etc.

And insert it in place of the cookie.

I created a new account, then intercepted the request in Burp and changed the session value to...

C4ca4238a0b923820dcc509a6f75849b (this is the MD5 hash for 1).
<img width="506" height="298" alt="screenshot_261002_145505" src="https://github.com/user-attachments/assets/a70ee346-20d2-444d-a7f5-c95cabee211e" />
<img width="1331" height="951" alt="screenshot_261002_150125" src="https://github.com/user-attachments/assets/6e11ce0e-ecf6-4de3-b2c3-b80624608c7e" />
<img width="491" height="411" alt="screenshot_261002_145942" src="https://github.com/user-attachments/assets/004b64e1-65b4-4027-b652-ce4387408788" />
We change the ID—I created a new account, for instance, though I don't think it's actually necessary—and the flag is ours.

---

#flag7
And one was left.
I took a hint, and it turned out to be related to deleting posts.
<img width="890" height="83" alt="screenshot_261002_152232" src="https://github.com/user-attachments/assets/6966f21d-a9a7-42b3-aa41-34aba223089b" />
So I was right—there *is* something there. I just took the wrong step and triggered an XSS vulnerability, whereas I should have done something else to figure out what that identifier was—it was the same ID as in the cookie, and also a hash.
<img width="1323" height="401" alt="screenshot_261002_152150" src="https://github.com/user-attachments/assets/1b4328dd-1511-4f3f-9d85-83bd4dc2f8f4" />
The system here is the same as with the cookie IDs; it's a hash, but based on "3".
I decided to try the hash based on "1" again.
C4ca4238a0b923820dcc509a6f75849b
<img width="1479" height="358" alt="screenshot_261002_152520" src="https://github.com/user-attachments/assets/465c5acd-b126-4851-81f2-56329674165f" />
and our flag

---
Author: Ray1N-0x





















