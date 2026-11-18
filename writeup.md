okk
the plan in my mind is simple
build a vulnerable machine, pentest it or hack it if ya want it to sound cool

but building a vulnerable by my own would be fun but then pentest would not be fun
so i relied on my AI agent to build the machine

"Title: Clinic Portal

A web CTF set around a fictional clinic’s online portal. Explore the application, assess its security, and investigate the activity it produces through your SOC.

Broad tags: Web Application · Healthcare · CTF · Log Analysis.
  "
this is all the info provided, the next thing was setting up SIEM

i couldn't use wazuh or splunk due to my hardware limits
so I made custom one with again AI

"
- Backend: Python, Flask, and Waitress.
- Database: SQLite stores events, alerts, accounts, case notes, and audit history.
- Frontend: Server-rendered HTML/Jinja, CSS, and JavaScript for the alert queue and JSON viewer.
- Collection: Every 30 seconds, a restricted SSH account reads Ubuntu’s journal and the CTF’s JSON logs. Journal cursors and file offsets track new events.
- Detection: Python rules inspect events and correlate activity by source, IP, and time. Currently 16 rules, with duplicate suppression.
- Workflow: Separate L1, L2, and IR accounts, with role checks, password hashing, and CSRF protection. Alerts move through triage → investigation → response → closure.
- Response: A separate SSH account runs restricted evidence-collection and temporary IP-blocking commands. Response stays manual.
- Hosting: Runs on your main host at 127.0.0.1:8765 as a temporary user service.

Everything lives in soc/; 11 automated tests cover the main workflows and detection logic.
"
okk now, we can start

now lets solve the ctf

![[Pasted image 20261117191111.png]]
okeh
its a login page
![[Pasted image 20261117191226.png]]
there are many attack vectors
so lets start exploring

![[Pasted image 20261117191557.png]]XSS doesn't work

![[Pasted image 20261117191836.png]]
lets try decode the key

editing the role to admin didnt work

so lets try sqli
![[Pasted image 20261117192344.png]]

didnt go so well either

lets try the calendar feature
![[Pasted image 20261117193347.png]]![[Pasted image 20261117193354.png]]

okeh
it works, 

but before that i was able to find an SQL injection
![[Pasted image 20261117193804.png]]
> [
>   {
>     "id": 1,
>     "name": "Clinic Administrator",
>     "role": "admin"
>   },
>   ...
>   {
>     "id": 10,
>     "name": "xd",
>     "role": "user"
>   }
> ]

well i could definately use Clinic Administrator for exploiting login functions

but we do need email of admin

![[Pasted image 20261117202321.png]]
we got the columns

password_hash, email, and id

![[Pasted image 20261117203741.png]]

got the password hash

```
$2b$12$BT/8RsJuf5NoHEo9YYCKTuf2C/2Y38XW4JPoeb7ugbhNmNGYBXcOi
```

its a bcrypt hash

![[Pasted image 20261117203952.png]]

got the email

admin@clinic.example

now i requested a reset

![[Pasted image 20261117204332.png]]

now i check out the reset_tokens table 
![[Pasted image 20261117205734.png]]
we got the token, and idk what to put in bady, but hey we got the reset rigth

![[Pasted image 20261117210437.png]]

we(i) got in

![[Pasted image 20261117210616.png]]

the admin has litrally no priviledge except making other admins

but hey, we got admin acces, now i will invsestiagate the events via SIEM

![[Pasted image 20261118132205.png]]

as an l1, there are four alert. i tested one false positive alert to check if UI worked properly

![[Pasted image 20261118132820.png]]
lets check this one first

![[Pasted image 20261118132844.png]]
this was me, forgetting my account password during pentest

the detectiong rules suck, ill make better rules in the next session

as all the alerts are false positive, and it didnt even detect the SQLi

so thats it for now