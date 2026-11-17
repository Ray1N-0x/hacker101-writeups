# Hacker101 Writeup — Petshop Pro

> **Target:** Petshop Pro
> **Category:** WEB
> **Difficulty:** Easy 
> **Flags:** 2/3 captured

---

# flag1
<img width="856" height="484" alt="Снимок экрана 2026-10-13 200541" src="https://github.com/user-attachments/assets/db4082be-bfb3-4301-82e3-11a159d4c3e8" />
Nothing interesting so far; we’re beginning reconnaissance.
<img width="226" height="462" alt="Снимок экрана 2026-10-13 200640" src="https://github.com/user-attachments/assets/f563732c-9b33-4c1a-bd28-b0f507529b63" />
In the `checkout` endpoint, the `body` contains `cart=[]` — let’s experiment with this.
When attempting to make a purchase, JSON data is passed within our array.
<img width="423" height="156" alt="Снимок экрана 2026-10-13 200916" src="https://github.com/user-attachments/assets/25fd3403-e905-495a-ac96-7b94fecfe4e1" />
And the name is displayed on the page, which means this is a potential entry point for us.
<img width="396" height="115" alt="Снимок экрана 2026-10-13 201155" src="https://github.com/user-attachments/assets/f22eb1c8-a855-41ca-9a37-9d2d22acd9f1" />
We break out of the tags—this is an XSS vulnerability, but it doesn't yield a flag.
However, the core of the challenge lies in bypassing business logic by changing the price to a value of our choosing, which earns us one flag.
<img width="1045" height="339" alt="Снимок экрана 2026-10-13 201252" src="https://github.com/user-attachments/assets/3b69207a-3aac-49f5-b55d-24f28c3a86be" />

# flag2
There is a login page here
<img width="618" height="318" alt="Снимок экрана 2026-10-13 202344" src="https://github.com/user-attachments/assets/9e6df569-3aa9-4638-8ac5-10c99c1e8b79" />
While enumerating directories, I discovered `/edit`. Accessing it returned a 400 Bad Request error. After considering what could be edited, I realized it likely referred to the "cats and dogs" page, where entries are managed by ID. I began testing various parameters until I tried `?id`. From there, I simply injected an XSS payload into the `name` field—and that was it: the second flag.
<img width="1081" height="617" alt="Снимок экрана 2026-10-13 210403" src="https://github.com/user-attachments/assets/a82668e8-3a1f-40c2-92b7-434b2cc47777" />
<img width="667" height="230" alt="Снимок экрана 2026-10-13 210436" src="https://github.com/user-attachments/assets/f9d4291e-7073-43ad-a34f-fd71de8200e8" />
I suspect the third flag is in the admin panel, but I don’t know how to get in yet. I’ve tried a lot of things—for instance, sending a JSON payload makes the error message disappear, though that doesn’t really get me anywhere. 
I tried cycling through parameter values, wondering if it might be an IDOR vulnerability, but I’m increasingly leaning toward the idea that a brute-force attack is the easiest path—even if it is incredibly tedious. 
Anyway, a lot of time has passed and I still haven’t found anything, so I’ll just have to brute-force the admin panel; there’s really no other option left. 
Actually, I don’t really want to brute-force it, but I guess I have to. In short, you’ll find the third flag once you manage to access the admin panel. You have two main approaches: 
    Brute-force the username first—using the "invalid username" error message as a guide—and then move on to the password. 
    Alternatively, you could try to brute-force both the username and password at once (which takes longer, but covers both). 
    If you’re using Hydra, the command would look something like this: 
    hydra -L usernames.txt -P rockyou.txt target_ip http-post-form “/login/login:username=^USER^&password=^PASS^:Invalid login”

    ---

    # Author: Ray1N-0x
