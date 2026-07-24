# Business Logic Vulnerabilities

Business Logic Vulnerabilities (Application Logic Vulnerabilities or Logic Flaws) are flaws in an application's **design or implementation** that allow attackers to abuse legitimate functionality and cause unintended behavior.

Unlike vulnerabilities caused by coding mistakes (e.g., SQL Injection or XSS), business logic vulnerabilities result from **incorrect business rules, flawed workflows, or unsafe assumptions about user behavior**.

Their exploitation depends on understanding how the application is intended to work and finding ways to make it behave in unintended ways.

---

# What is Business Logic?

Business logic is the set of **rules** that define how an application should operate.

Examples:

- Users must pay before receiving a product.
- Discounts cannot exceed 100%.
- A user can only modify their own account.
- Stock cannot become negative.

These rules enforce what users are allowed to do.

---

# Why Do Business Logic Vulnerabilities Occur?

They usually arise because developers make **incorrect assumptions** about user behavior, such as:

- Users only interact through the web interface.
- Users follow the intended workflow.
- Users always provide valid input.
- Client-side validation is sufficient.
- Application components always receive expected data.

Attackers intentionally violate these assumptions.

---

# How Attackers Exploit Logic Flaws

Attackers abuse legitimate functionality by:

- Skipping workflow steps.
- Modifying HTTP requests (Burp Suite).
- Changing transaction-critical values.
- Sending unexpected or invalid input.
- Repeating requests.
- Combining features in unintended ways.

This causes the application to perform actions developers never intended.

---

# Why Are They Difficult to Detect?

Business logic vulnerabilities are:

- Invisible during normal application use.
- Unique to each application.
- Dependent on understanding the application's business rules.
- Difficult for automated vulnerability scanners to detect.

They are commonly discovered through **manual testing** and **bug bounty hunting**.

---

# Possible Impact

The impact depends on the affected functionality.

Examples include:

- Authentication bypass
- Privilege escalation
- Purchasing products at unintended prices
- Free products or services
- Financial fraud
- Access to sensitive information
- Business disruption
- Circumventing security controls

Even seemingly minor logic flaws should be fixed because they may lead to severe attacks.

---

# Root Causes

Business logic vulnerabilities often result from:

- Failing to anticipate unusual application states.
- Missing or weak server-side validation.
- Trusting client-side controls.
- Poor understanding of interactions between application components.
- Undocumented assumptions during development.

---

# Key Takeaway

A **Business Logic Vulnerability** occurs when an attacker abuses an application's own business rules to achieve unintended behavior by violating assumptions made by the developers.

---

# Examples

- Completing a purchase without paying.
- Applying a negative quantity to gain money.
- Modifying the product price in a request.
- Skipping required workflow steps.
- Resetting another user's password due to flawed validation.

---

# Lab: Excessive trust in client-side controls

first login

```text
wiener:peter
```

then click on

```text
Lightweight l33t leather jacket
```

product.

click

```text
add to cart
```

and intercept the request.

```http
POST /cart
..
..
productId=1&redir=PRODUCT&quantity=1&price=133700
```

change the

```text
price=10
```

then go to

```text
/ cart
```

and click purchase.

---

# Lab: 2FA broken logic

With Burp running, log in to your own account and investigate the 2FA verification process. Notice that in the `POST /login2` request, the `verify` parameter is used to determine which user's account is being accessed.

Log out of your account.

Send the `GET /login2` request to Burp Repeater. Change the value of the `verify` parameter to `carlos` and send the request. This ensures that a temporary 2FA code is generated for Carlos.

Go to the login page and enter your username and password. Then, submit an invalid 2FA code.

Send the `POST /login2` request to Burp Intruder.

In Burp Intruder:

- Set the `verify` parameter to `carlos`.
- Add a payload position to the `mfa-code` parameter.
- Brute-force the verification code.

Load the `302` response in the browser.

Click **My account** to solve the lab.

---

# Lab: High-level logic vulnerability

login with

```text
wiener:peter
```

browse the site then go to burp proxy's history and study the requests and responses.

and product

```text
2 $95.14 (what do u meme)
```

notice that if u click on

```text
-
```

button for product 2 the quantity value is a minus

```text
-1
```

change the quantity to

```text
-14
```

for product 2.

u will notice that the total price is negative.

add to cart

```text
product 1 (Lightweight "l33t" Leather Jacket) $1337.00
```

now the total price is lower than

```text
$100
```

dollars.

click purchase.

---

# Lab: Low-level logic flaw

try the previous techniques used on the previous lab such as:

- changing the quantity to a negative number -> not worked
- changing the quantity to a big number like `122` -> `"invalid parameter"`
- try `99` -> worked
- try `100` -> `"invalid parameter"`

this means that on each request the maximum number of quantity is `99`.

Note that the price of the jacket is stored in cents (`133700`).

to solve the lab we need to make the total price in a negative value.

send the request to burp intruder.

using burp intruder:

- payloads tab:
  - payload type `null payloads`
  - continue indefinitely

- resource pool:
  - max concurrent requests = `1`

so we can stop the request when we see the total price becomes negative.

start the attack:

The total price has exceeded the maximum value permitted for an integer in the back-end programming language (`2,147,483,647`). As a result, the value has looped back around to the minimum possible value (`-2,147,483,648`).

```text
quantity = 18018

price = $1337.00

Total-price = -$18859606.96
```

use the calculator:

```text
total-price / price = quantity
```

(just take the integer value of quantity)

```text
quantity / 99 = number-of requests
```

(just take the integer value to make sure u are still on the negative range)

then go to intruder:

- payloads
- payload configuration
- generate = number of requests

resource pool:

```text
max concurrent requests = default
```

start the attack.

go back to the page and u will notice that the number is going towards zero.

repeat the process until:

```text
total-price < 1337
```

now go and choose another item, for example:

```text
productId=4
price=$3.75
```

use calculator again:

```text
total-price / productID-4-price = quantity
```

and again in burp intruder change the productid to `4`.

repeat the process until:

```text
total-price < 3.75
```

now in burp repeater add a

```text
quantity = 1
```

so the total price is a positive and less than `$100`.

place order.

# Lab: Inconsistent handling of exceptional input

i used gobuster to get the admin panel page which is

```text
/admin
```

it showed that only

```text
@dontwannacry.com
```

users are only allowed to access admin panel page.

click on

```text
email client
```

button.

it showed that it can received any email address:

```text
@YOUR-EMAIL-ID.web-security-academy.net
```

and subdomains.

so what we are going to do is make

```text
dontwannacry.com
```

as a subdomain for

```text
YOUR-EMAIL-ID.web-security-academy.net
```

to get the register link massage.

since the common email length used is `255` so what if we exceeded the limit.

use this code:

```php
<?php
for($i=0;$i<255;$i++){
    echo "a";
}

echo "@YOUR-EMAIL-ID.web-security-academy.net";
```

go back to email client and click register.

sign in -> notice that the email address truncated to

```text
aaaaaaaaaaaaaaaa....a
```

now to be a dontwannacry user:

```text
"@dontwannacry.com" , length=17
255-17=238
```

```php
<?php
for($i=0;$i<238;$i++){
    echo "a";
}
echo "@dontwannacry.com.YOUR-EMAIL-ID.web-security-academy.net";
```

register and sign in -> the email address:

```text
aaaaaaaaaaaaa...@dontwannacry.com
```

notice that the admin panel exist on the home page so visit it and delete carlos user.

---

# Lab: Inconsistent security controls

i used gobuster to get the admin panel page which is

```text
/admin
```

OR

if u have a burp suite pro edition:

```text
go to target -> sitemap

right click on the website target

-> engagement tools

-> content discovery

-> click session not running
```

After a short while, look at the **Site map** tab in the dialog.

Notice that it discovered the path:

```text
/admin
```

visit the admin panel.

the error shows that only

```text
dontwannacry
```

users can access the admin panel.

go to registration page and try the previous technique used in the previous lab -> its not worked.

register with a new credentials.

go to my account page.

notice that there is an email update functionality.

change ur email to

```text
name@dontwannacry.com
```

now go back to admin panel and delete carlos user.

---

# Lab: Weak isolation on dual-use endpoint

login using

```text
wiener/peter
```

credential.

browse to

```text
my-account
```

page.

change username value to

```text
administrator
```

send the request, intercept it, send it to repeater.

it shows an error:

```text
incorrect current password
```

go to burp repeater.

delete the current password parameter and send the request.

response:

```text
password changed
```

login as administrator then delete carlos user.

---

# Lab: Password reset broken logic

browse to my-account page.

click

```text
forgot password
```

go to email client then click on the sent link.

write the new password.

go back to burp proxy's history and study the reset password request.

```http
POST /forgot-password?temp-forgot-password-token=893di6lz8gkjcxi1x91l0tmeau1xg4d7 HTTP/2
Host: 0a3d00b10432b46180154efe008c0029.web-security-academy.net
Cookie: session=tHMs15sKhbPFZ8Km2Uvvv8ikIIt7iVnr
Content-Length: 113
Cache-Control: max-age=0
Sec-Ch-Ua: "Not;A=Brand";v="8", "Chromium";v="150", "Brave";v="150"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36
Origin: https://0a3d00b10432b46180154efe008c0029.web-security-academy.net
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8
Sec-Gpc: 1
Accept-Language: en-US,en;q=0.7
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://0a3d00b10432b46180154efe008c0029.web-security-academy.net/forgot-password?temp-forgot-password-token=893di6lz8gkjcxi1x91l0tmeau1xg4d7
Accept-Encoding: gzip, deflate, br
Priority: u=0, i

temp-forgot-password-token=893di6lz8gkjcxi1x91l0tmeau1xg4d7&username=wiener&new-password-1=123&new-password-2=123
```

notice that the http body request contain a hidden input

```text
username=wiener
```

also changing

```text
temp-forgot-password-token
```

or deleting it doesn't make any sense.

change the username to

```text
carlos
```

and send the request.

go back to my account and login with carlos user.

---

# Lab: 2FA simple bypass

login with

```text
wiener:peter
```

go to email client to receive the 2fa code then submit it.

go back to burp proxy's history.

post login.

the response shows:

- a redirection to

```text
/login2
```

page which sends the mfa code to ur email client for submitting it.

- `Set-Cookie: session`

this means that we are logged in so there is no need to go to

```text
/login2
```

logout.

login with

```text
carlos:montoya
```

after login change the url manually to go back to myaccount page.

---

# Lab: Insufficient workflow validation

add the leather jacket to your cart.

click on

```text
place order
```

button and intercept it using burp proxy.

since the current money isn't enogh to buy the leather, the website redirected us to

```text
GET /car/?err=insufficient_funds
```

remove the leather jacket from the cart and add another item which is lower than `$100`.

click on

```text
place order
```

u will notice that the website redirected us to

```text
GET /cart/order-confirmation?order-confirmed=true
```

what happened here is:

```text
/cart/checkout
```

checks if our money lower than the total price.

if yes

```text
GET /car/?err=insufficient_funds
```

otherwise

```text
take from ur money

GET /cart/order-confirmation?order-confirmed=true
```

and the

```text
order-confirmation
```

page will add the order to the waiting state.

so send the

```text
order-confirmation
```

request to burp repeater.

add the leather jacket and send the order-confirmation request.

# Lab: Authentication bypass via flawed state machine

using tools like

```text
gobuster
dirbuster
```

to discover hidden file.

found admin page.

visit it.

it shows that only administrator can access this page.

login with

```text
wiener:peter
```

select role:

```text
user
```

or

```text
content author
```

go to burp proxy's history and study the sent requests.

lets try to not adhere with the sequence that the website enforce us to flow with.

turn intercept on.

login again but this time don't go to select-role page.

take the new given session from the response and then send a request to

```text
/
```

(home page)

and set the new session.

notice that admin panel link exist in the home page means the website gave us administrator role by default since we didn't select the role.

now delete carlos user.

---

# Lab: Flawed enforcement of business rules

- log in and notice that there is a coupon code:

```text
NEWCUST5
```

- At the bottom of the page, sign up to the newsletter.

You receive another coupon code:

```text
SIGNUP30
```

- Add the leather jacket to your cart.

- Go to the checkout and apply both of the coupon codes to get a discount on your order.

- Try applying the codes more than once.

Notice that if you enter the same code twice in a row, it is rejected because the coupon has already been applied.

However, if you alternate between the two codes, you can bypass this control.

- Reuse the two codes enough times to reduce your order total to less than your remaining store credit.

Complete the order to solve the lab.

---

# Lab: Infinite money logic flaw

- log in and notice at the bottom of the page, sign up to the newsletter.

You receive another coupon code:

```text
SIGNUP30
```

(30% discount)

- notice that u can buy a gift card from the given products which costs

```text
$10
```

so if u apply the coupon it will cost

```text
$7
```

then go to my account page and redeem the

```text
$10
```

gift card.

notice that u got a

```text
$3
```

extra dollar so we can make infinite money glitch by repeating the same process.

so we will use burp intruder and also using session handling rules to automate the process.

solution link:

```text
https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-infinite-money
```

---

# Lab: Authentication bypass via encryption oracle

Log in with the

```text
Stay logged in
```

option enabled and post a comment.

go to burp proxy's history tab and study the requests and responses.

- notice that the

```text
stay logged in
```

cookie is encrypted.

test the comment functionality.

u will notice that when posting a comment with an invalid email format, the response sets an encrypted notification cookie before redirecting you to the blog post.

Notice that the error message reflects your input from the email parameter in cleartext:

```text
Invalid email address: your-invalid-email
```

Deduce that this must be decrypted from the notification cookie.

Send the

```text
POST /post/comment
```

and the subsequent

```text
GET /post?postId=x
```

request (containing the notification cookie) to Burp Repeater.

In Repeater, observe that you can use the email parameter of the POST request to encrypt arbitrary data and reflect the corresponding ciphertext in the `Set-Cookie` header.

Likewise, you can use the notification cookie in the GET request to decrypt arbitrary ciphertext and reflect the output in the error message.

For simplicity, double-click the tab for each request and rename the tabs:

```text
encrypt
decrypt
```

there is a chance that notification and stay logged in cookie are encrypted using the same cipher method.

so copy the

```text
stay-logged-in
```

cookie and paste it into the notification cookie.

Send the request.

Instead of the error message, the response now contains the decrypted stay-logged-in cookie, for example:

```text
wiener:1598530205184
```

```text
username:timestamp
```

add

```text
administrator:1598530205184
```

in the encrypt request.

then take the `Set-Cookie` notification value and decrypt it.

notice that the output:

```text
Invalid email address: administrator:1598530205184
```

we want the cipher text to only contain:

```text
administrator:1598530205184
```

send the notification value to Decoder.

notice that the value seems to be URL encoded.

```text
%3d
```

decode it as

```text
URL-decode
```

notice that the result looks like Base64 encoded.

i knew it because Base64 only contain:

```text
A–Z
a–z
/
=
```

(`=` used at the end for padding)

so decode using:

```text
Base64-decode
```

the output is binary data.

suppose that each char is 1 byte.

the string

```text
Invalid email address:
```

length is

```text
23
```

characters.

click

```text
Hex
```

Select the first 23 bytes.

Right-click.

Select

```text
Delete selected bytes
```

now encode using:

```text
Base64
URL-encoding
```

then copy the result and go back to decrypt it.

the response returns internal server error which indicates that a block-based encryption algorithm is used and that the input length must be a multiple of 16.

You need to pad the

```text
Invalid email address:
```

prefix with enough bytes so that the number of bytes you will remove is a multiple of 16.

```text
2 * 16 = 32

23 + 9 = 32
```

```text
XXXXXXXXXadministrator:1598530205184
```

repeat the same process but this time delete

```text
32
```

bytes.

notice that after decryption the output shows:

```text
administrator:1598530205184
```

now go to home page and replace the

```text
stay-logged-in
```

cookie value with the encrypted

```text
administrator:1598530205184
```

and delete the

```text
session
```

cookie

(because the associated stay-logged-in cookie will not match `administrator:1598530205184`)

notice that the home page shows admin page link:

```text
/admin
```

go to

```text
/ admin
```

and delete carlos user.

# Splitting the email atom: exploiting parsers to bypass access controls

```text
https://portswigger.net/research/splitting-the-email-atom#github
```

---

# Lab: Bypassing access controls using email address parsing discrepancies

> **##read splitting the email atom research before solving this lab##**

when registering using email:

```text
foo@exploit-server.net
```

it show us an error message stating that the email domain must be

```text
ginandjuice.shop
```

Try to register an account with the following emails:

```text
=?iso-8859-1?q?=61=62=63?=foo@ginandjuice.shop
```

```text
(not worked)
```

```text
=?utf-8?q?=61=62=63?=foo@ginandjuice.shop
```

```text
(not worked)
```

the server is detecting and rejecting attempts to manipulate the registration email with encoded word encoding.

so lets try using less common encoding formats such as utf-7.

```text
=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop
```

```text
Notice that this attempt doesn't trigger an error
```

which means the server doesn't recognize UTF-7 encoding as a security threat.

now lets craft an attack that tricks the server into sending a confirmation email to your exploit server email address while appearing to still satisfy the ginandjuice.shop domain requirement.

go to CyberChef website then search for encoding text and choose UTF-7.

```text
@  -> &AEA-
whitespace -> &ACA
```

```text
=?utf-7?q?attacker&AEA-exploit-0ad600590343c5f282536e3c015e00f7.exploit-server.net&ACA-?=@ginandjuice.shop
```

```text
attacker@exploit-0ad600590343c5f282536e3c015e00f7.exploit-server.net ?=@ginandjuice.shop
```

Click **Email client**.

then click on the registration link.

login in.

visit:

```text
/admin
```

delete carlos user.
