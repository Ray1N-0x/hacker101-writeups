# Grayhatcon CTF

**Difficulty:** Moderate  
**Flags:** 1/4  
**Date:** October 2026  
**Platform:** Hacker101 CTF

---

## flag1
<img width="1614" height="581" alt="screenshot_261003_110754" src="https://github.com/user-attachments/assets/3c58aa33-44b8-4545-8a2e-24d34588a99c" />
Ok, let's go
I started examining the page source, where I found some very interesting information, and also began fuzzing for directories to see if I could find anything else of interest.
<img width="894" height="814" alt="screenshot_261003_111019" src="https://github.com/user-attachments/assets/c399fa29-c2d9-4451-a70c-1b4566dfdfdd" />
<img width="306" height="194" alt="screenshot_261003_111526" src="https://github.com/user-attachments/assets/c28677c7-6717-4033-b600-63c0dda49278" />
ffuf found robots.txt, and upon visiting it, I discovered an interesting endpoint.
<img width="227" height="69" alt="screenshot_261003_111615" src="https://github.com/user-attachments/assets/ef5ce5aa-bb26-465c-a1a7-cc4cf7d227c0" />
If you follow this link... well, who said it would be that easy?
<img width="412" height="190" alt="screenshot_261003_111701" src="https://github.com/user-attachments/assets/03a83d9c-2aa2-43ca-b4cb-5d9116c7d215" />
There is a password-change function that allows us to determine whether a user exists; this will help us in the future.
<img width="804" height="228" alt="screenshot_261003_112507" src="https://github.com/user-attachments/assets/08fb12fa-f04a-4609-9b69-451e604778ef" />
I created an account and logged in, only to be immediately met with suspicious information.
<img width="936" height="261" alt="screenshot_261003_112627" src="https://github.com/user-attachments/assets/9fd01650-202f-4689-9811-ab8eb3a85164" />
interesting information hash
By looking at the page source, we find even more interesting things.
<img width="1517" height="1089" alt="screenshot_261003_112904" src="https://github.com/user-attachments/assets/a093282c-9880-4b15-8668-d814dd40cb4a" />
potentially, this is
An AJAX request that loads data for the auction. There are two potential hooks here:
1. XSS via v.question
Look at this line:
`javascript
 $('.auction_questions').append('<div style="margin-top:7px"><label>' + v.question + '</label></div>...');`
If v.question is user input (e.g., an auction title or a field you can fill in) and it isn't sanitized, you can inject this:
html
<img src=x onerror=alert(1)>
When the question is displayed on the page, the XSS will trigger. This is Stored XSS because you save it to the database, and it is displayed to others.
IDOR via id
The request goes to auctions/questions?id= + i. If you can manipulate the id and retrieve data about other people's auctions, that’s IDOR.
2. XSS via v.title
Look further down:
`javascript
$('.other_auctions').append('<li><a href="../auction/' + v.id + '">...' + v.title + '</a></li>');`
If v.title is the auction title and you can control it (e.g., by creating an auction with that title), you can inject XSS there. It will trigger when someone views the list of auctions.
Let's try
So far, this hasn't yielded anything; I'm going to keep looking.
<img width="978" height="394" alt="screenshot_261003_113448" src="https://github.com/user-attachments/assets/4d747df7-f272-4d9a-98a9-9d89499bfe68" />
The auction is closed—that’s a shame; let’s move on.
<img width="744" height="440" alt="screenshot_261003_113656" src="https://github.com/user-attachments/assets/44e04350-d81a-4271-ba2d-7017fb0a0a88" />
I spent a long time looking for leads but found nothing, so I decided to check the main page again—I was certain I’d seen someone selling something there.
<img width="910" height="489" alt="screenshot_261003_123046" src="https://github.com/user-attachments/assets/66705c9b-2f45-4393-9d37-298accfcc479" />
<img width="970" height="495" alt="screenshot_261003_123344" src="https://github.com/user-attachments/assets/e6961830-5173-467a-a895-0ea4b87a4cac" />
The posts contain usernames; I tried the ones from all the posts, but only one worked—`hunter2`. That must have been the one the pop-up message referred to.
I was testing them on the password reset page.
<img width="881" height="357" alt="screenshot_261003_123453" src="https://github.com/user-attachments/assets/578b0ce0-b4cd-42c5-80a2-f55ab05422af" />
After entering a random name, I opened Burp and saw a hash there, assuming it belonged to hunter2.
Cf505baebbaf25a0a4c63eb93331eb36
<img width="637" height="312" alt="screenshot_261003_123526" src="https://github.com/user-attachments/assets/a592a54a-7a95-4c27-bf71-c93421dace01" />
Upon logging in, we are given a hash, and I wanted to try swapping ours for his.

I tried for a long time without success; then I noticed a message on the homepage stating that the account wasn't verified, so I started trying to add different fields during registration.

That didn't work either, but then I remembered the feature for adding users to assist with the auction—I don't have an auction, but hunter2 does, and I even have his hash.
<img width="908" height="438" alt="screenshot_261003_124307" src="https://github.com/user-attachments/assets/0ced1940-02e8-42f1-b037-2cd795502142" />
<img width="832" height="189" alt="screenshot_261003_124421" src="https://github.com/user-attachments/assets/00527d0d-ad14-42bd-8077-4637db11d7fb" />
The server is apparently checking for it; I tried a different approach using cookies—basically putting that hash anywhere I could.
<img width="714" height="160" alt="screenshot_261003_125349" src="https://github.com/user-attachments/assets/6e12d724-0000-4d9b-bdcc-dd46198828d6" />

<img width="479" height="146" alt="screenshot_261003_131029" src="https://github.com/user-attachments/assets/825cb78a-0756-44d5-a3a7-bae689209814" />

<img width="489" height="228" alt="screenshot_261003_131038" src="https://github.com/user-attachments/assets/dfb43e81-0a7f-47fa-8fc4-9d5d6a471000" />
I noticed a certain pattern: upon registration, you are given a hash that is subsequently displayed in your profile.
Upon registration, we receive a `userhash` that remains visible to us.

I need to find a way to become a "hunter"; simply swapping the parameter and the cookie didn't work.

So, I’ll try other methods: I’ll attempt to pass extra parameters during registration, login, or user addition; I’ll try swapping parameters to see how the server reacts; and I’ll try changing parameter names or sending data other than what is expected.
After about an hour, I figured out how to use it. I’ll try to explain the steps now—it’s actually much simpler; I just missed a couple of details.

Go to the login page and log in with your user account.
We intercept the outgoing request in Burp and add the field `account_hash=cf505baebbaf25a0a4c63eb93331eb36`; this is the hash for user `hunter2`.
<img width="847" height="512" alt="screenshot_261003_132944" src="https://github.com/user-attachments/assets/28d8fa92-0563-4f2b-8750-a8ee9f9f6972" />
Next, we wait for responses from the server and update the hashes for each response; depending on the response, the name will change from `account_hash` to `user_hash`.
<img width="751" height="489" alt="screenshot_261003_133007" src="https://github.com/user-attachments/assets/91b09386-c5c4-4556-9083-cb2e605b9051" />

<img width="857" height="528" alt="screenshot_261003_133032" src="https://github.com/user-attachments/assets/4a3628ba-b23e-4003-bd4d-f5cd1ffdddca" />
after which we also check our flag
<img width="830" height="397" alt="screenshot_261003_132604" src="https://github.com/user-attachments/assets/ce6a2097-66e6-4239-9e5f-d44b43f3ec2a" />

---

I already have some ideas on where to look for the remaining three flags; feel free to re-read this write-up, and I think you'll understand. I’ll be updating this file as I exploit the rest. Thanks, everyone.

---

# Author: Ray1N-0x
