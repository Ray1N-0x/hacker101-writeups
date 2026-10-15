# Ticketastic: Live Instance

**Difficulty:** Moderate
**Flags:** 2/2
**Date:** October 2026
**Platform:** Hacker101 CTF

---

## flag1
Main page 
<img width="807" height="243" alt="Снимок экрана 2026-10-15 200758" src="https://github.com/user-attachments/assets/83c71bc2-2c87-42c1-a274-d7f821215585" />
<img width="476" height="292" alt="Снимок экрана 2026-10-15 201257" src="https://github.com/user-attachments/assets/8b64196d-1005-4fdd-9347-c4e5927f459f" />
This is admin login page
<img width="388" height="209" alt="Снимок экрана 2026-10-15 201426" src="https://github.com/user-attachments/assets/12c4673f-7355-404b-b51e-fc5c063eb46a" />
<img width="1042" height="379" alt="Снимок экрана 2026-10-15 201518" src="https://github.com/user-attachments/assets/1b99618b-9286-42ea-b84f-762b9d3d9e9f" />
So, the user 'admin' exists; now all that’s left is to figure out the password.
<img width="298" height="154" alt="Снимок экрана 2026-10-15 202611" src="https://github.com/user-attachments/assets/a8ae8655-ea9f-42e5-9bf9-502b7fc736f6" />
Hydra command
hydra -l admin -P /path/to/wordlist -f -V <target> https-post-form "/login:username=^USER^&password=^PASS^:F=Invalid"
Login attempt
Hydra misled me; I made a mistake in the text string for an incorrect password, which caused it to throw an error. The correct command is below:
hydra -l admin -P /path/to/wordlist -f -V <target> https-post-form "/login:username=^USER^&password=^PASS^:F=Invalid password"
I never managed to figure out the password.
While fuzzing, I discovered an interesting endpoint: `newUser`
<img width="1025" height="354" alt="Снимок экрана 2026-10-15 205942" src="https://github.com/user-attachments/assets/5ef84855-66ba-4d61-a00d-39a6807b3e48" />
Based on previous rooms, I suspect that the tickets we submit are sent via the server to an admin panel, and I want to try experimenting with this.
I believe this endpoint (`newUser`) is intended for an administrator—who has the ability to add users—so I’ll try including some HTML or a script in the ticket body to see if I can trigger it.
After a lot of experimentation, I finally got it to work by combining an `<a>` (anchor) tag with a `<script>` tag. Upon further testing, I discovered that there are actually many different ways this could play out.
I sent the following request:
<img width="840" height="326" alt="Снимок экрана 2026-10-15 211403" src="https://github.com/user-attachments/assets/ca73b8d9-7f74-4bf0-ad50-3dec3df54889" />
It’s encoded directly into the URL—essentially a link to the `newUser` endpoint that routes through `localhost`, carrying `usrname` and `password` as parameters.
And as soon as I access the admin panel, I’m greeted by the XSS payload I had previously sent—which, in that earlier instance, was included in the request body.
<img width="1115" height="479" alt="Снимок экрана 2026-10-15 210804" src="https://github.com/user-attachments/assets/7c8dedd0-3992-43be-b726-34ece42b1b7e" />

---

## flag2

And the most interesting part is that it’s the `title` parameter that’s at play; it’s evidently not being handled correctly and is displayed exactly as-is—which is where the XSS comes from.
Inside we have an interesting parameter /ticket?id=2
<img width="810" height="271" alt="Снимок экрана 2026-10-15 211702" src="https://github.com/user-attachments/assets/c507a2f6-2333-45e5-9538-3cd72b9f3d44" />
We change it to 1, and that’s it.
We get an error when attempting to inject a command.
<img width="840" height="248" alt="Снимок экрана 2026-10-15 212335" src="https://github.com/user-attachments/assets/8f96bf13-ce60-4baa-8faa-4f8ec953704d" />
Our input is inserted into an SQL query without being processed—I’m 100% sure this is an SQL injection.
I am testing a parameter; we have full control over it, but it is not being validated on the other end.
The query selects 3 columns, so the `UNION` must also involve 3 columns.
<img width="904" height="365" alt="Снимок экрана 2026-10-15 212615" src="https://github.com/user-attachments/assets/e5ccaf6e-20c2-429e-81d2-91279d0a5de6" />
`UNION SELECT 1,database(),version()`
<img width="341" height="337" alt="Снимок экрана 2026-10-15 212725" src="https://github.com/user-attachments/assets/58fba003-3906-4601-8b0a-3a55ed16f85d" />
We’ve rolled out a version; from here, we’ll keep pushing in this direction and try to pull in all the data.
`UNION SELECT 1,groupconcat(tablename),3 FROM informationschema.tables WHERE table_schema=database() `
<img width="216" height="298" alt="Снимок экрана 2026-10-15 213144" src="https://github.com/user-attachments/assets/2d42d47a-c9b4-4e48-9717-645f4ab55902" />
`UNION SELECT 1, groupconcat(columnname),3 FROM informationschema.columns WHERE table_name=’users’ `
<img width="573" height="214" alt="Снимок экрана 2026-10-15 213303" src="https://github.com/user-attachments/assets/b6e259a3-439e-485a-bf95-35748a8d3fd4" />
Now we need to extract the logins and passwords.
`UNION SELECT 1, group_concat(username,0x3a,password),3 FROM level7.users `
<img width="438" height="226" alt="Снимок экрана 2026-10-15 213632" src="https://github.com/user-attachments/assets/04830f8b-1f00-45ee-bdbc-05f9410becca" />
Where we got the flag — this is the 2nd flag out of 2.

---

# Author: Ray1N-0x
