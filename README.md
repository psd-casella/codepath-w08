# Project 8 - Pentesting Live Targets

Time spent: **5** hours spent in total

> Objective: Identify vulnerabilities in three different versions of the Globitek website: blue, green, and red.

The six possible exploits are:
* Username Enumeration
* Insecure Direct Object Reference (IDOR)
* SQL Injection (SQLi)
* Cross-Site Scripting (XSS)
* Cross-Site Request Forgery (CSRF)
* Session Hijacking/Fixation

Each version of the site has been given two of the six vulnerabilities. (In other words, all six of the exploits should be assignable to one of the sites.)

## Blue

Vulnerability #1: SQL Injection

The blue site is vulnerable to SQL injection on the path `/blue/public/staff/salespeople/show.php?id=`, where the `id` GET variable can be used as an injection point. 

For example, setting `id` to the value:
```mysql
' AND 0 UNION SELECT 1,2,3,4,GROUP_CONCAT(table_name,0x2e,column_name,"\n") FROM information_schema.columns WHERE table_schema=database(); -- 
```
allows the attacker to obtain details about the table and columns in the database. (Note: the above value should be URL encoded if appended directly to the URL.)

TODO: -GIF-



Vulnerability #2: Session Hijacking/Fixation

The blue site is vulnerable to session fixation and hijacking attacks. Using the supplied `public/hacktools/change_session_id.php` tool, a session created in one browser was successfully used in a different.

TODO: -GIF-



## Green

Vulnerability #1: Username Enumeration

The green site's login mechanism leaks information which indicates if a username exists in the database. If a username is not found in the database, the following HTML is returned:
```HTML
<span class="failed">Log in was unsuccessful.</span>
```
If a username is found in the database, an invalid login attempt returns the following HTML:
```HTML
<span class="failure">Log in was unsuccessful.</span>
```
This allows an attacker to identify valid usernames through brute force submissions to the login page.

TODO: -GIF-


Vulnerability #2: Cross-Site Scripting

The green site's contact form (found at `/green/public/contact.php`) allows arbitrary JavaScript to be submitted and later executed by authenticated users in the administration interface (found at `/green/public/staff/feedback/index.php`).

For example, submitting the following JavaScript in the "Feedback" field of the contact form will lead to a JavaScript alert appearing when feedback is viewed by an authenticated user.

```HTML
<script>alert('zmh68 - XSS');</script>
```

TODO: -GIF-



## Red

Vulnerability #1: Insecure Direct Object Reference (IDOR)

The red site is vulnerable to an IDOR attack, as inactive salespeople's detail pages are accessible by altering the value of the `id` GET parameter in the URL `/red/public/salesperson.php?id=`. Other versions of the site redirect requests for non-public salesperson profiles back to the general public listing of salespeople.

TODO: -GIF-



Vulnerability #2: Cross-Site Request Forgery (CSRF)

The red site does not require a valid CSRF token to be submitted when updating a salesperson's information via the URL `/red/public/staff/salespeople/edit.php?id=`, where `id` is the salesperson's ID. This enables covert editing of data through hiding requests to this endpoint in other pages an authenticated user might visit. 

For example, if an authenticated user loaded a page with the following sample HTML, the phone number of the salesperson with ID 5 would be updated, without the action being directly visible to the user.

TODO: -HTML SNIPPET-
TODO: -GIF-




## Optional

Bonus Objective 1: Build on Objective #3 (SQL Injection). Experiment to see what other kinds of information you can get the database to reveal.

Using a progression of injected SQL statements several table and column names stored in the database can be identified.

Progressively increasing the ORDER BY value allows detection of the number of columns in the table:

```mysql
' OR 1 ORDER BY 5-- -
```
Next, the names of tables and columns can be directed to the expected output fields that will display on the page by using the query:

```mysql
' AND 0 UNION SELECT 1,2,3,4,GROUP_CONCAT(table_name,0x2e,column_name,"\n") FROM information_schema.columns WHERE table_schema=database(); -- 
```
Resulting in the following information being displayed on the page:

```
countries.id,
countries.name,
countries.code,

failed_logins.id,
failed_logins.username,
failed_logins.count,
failed_logins.last_attempt,

feedback.id,
feedback.name,
feedback.email,
feedback.feedback,
feedback.created_at,

salespeople.id,
salespeople.first_name,
salespeople.last_name,
salespeople.phone,
salespeople.email,
salespeople_territories.territory_id
```

TODO: -GIF-


Bonus Objective 2: Build on Objective #4 (Cross-Site Scripting). Experiment to see if you can use XSS to: a) direct the user to a new URL, b) read cookie data, c) set cookie data.

Provided cookies are not properly secured, all of these can be accomplished by submitting JavaScript into the Contact form:

TODO: JavaScript to Inject
TODO: -GIF-


Advanced Objective: Build on Objectives #4 and #6 (XSS, hijacking/fixation). Are you able to execute a session hijacking or fixation attack using XSS instead of using the provided PHP script?

It may be possible, depending on the restrictions set on the cookie (e.g., HTTPOnly, sameSite, etc.)

TODO: Try this maybe
TODO: -GIF Maybe-




## Notes

No major challenges were encountered while completing this assignment.


## Concept Review

Which attacks were easiest to execute? Which were the most difficult?

IDOR attacks were the easiest. Surprisingly the "user enumeration" attack was puzzling, mostly because the assignment did not make it clear that this should be done without logging in to the site. 


What is a good rule of thumb which would prevent accidentally username enumeration vulnerabilities like the one created here?

Return the same errors for usernames that exist and usernames that do not exist. Also, respond in roughly the same amount of time to avoid leaking account existence via timing.


Since you should be somewhat familiar with the CMS and how it was coded, can you think of another resource which could be made vulnerable to an Insecure Direct Object Reference? What code could be removed which would expose it? (Hint: It was also the answer to the first bonus objective to the Weekly Assignment for week 3.)

I am not familiar with how the CMS was coded, because I don't seem to have access to its code. I think this question is for an older version of the course maybe.


Many SQL Injections use OR as part of the injected code. (For example: ' OR 1=1 --'.) Could AND work just as well in place of OR? (For example: ' AND 1=1 --'.) Why or why not?

AND can work, if you satisfy the first portion of logical statement and that is sufficient for the data you are seeking to extract from the overall query. Usually it seems like OR is an easier approach.


A stored XSS attack requires patience because it could be stored for months before being triggered. Because of this, what important ingredient would an attacker most likely include in a stored XSS attack script?

The attack should somehow indicate it has been executed to the attacker. The simplest way would be to send data to an attacker controlled server.


Imagine that one of your classmates is an authorized admin for the site's CMS and you are not. How would you get them to visit the self-submitting, hidden form page you created in Objective #5 (CSRF)?

Incorporate the self-submitting form into something essential or appealing that they would likely visit in the same browser session they are logged into the CMS with. Hopefully they don't use a strict container system for their browsing.


Compare session hijacking and session fixation. Which attack do you think is easier for an attacker to execute? Why? One of them is much easier to defend against than the other. Which one and why?

Session hijacking seems easier in most cases. Obtaining a session token via packet sniffing or another attack channel seems more likely to be achievable than establishing a session with a service and connecting the victim to that session and hoping the service does not have countermeasures. 

