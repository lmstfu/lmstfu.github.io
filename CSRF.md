# Protect against CSRF

## Architecture layer: modsecurity

Use when you want to introduce a CSRF token and cookie to protect against Cross-Site Request Forgery.

The basic strategy for protecting against CSRF is taken from [Patching Cross-Site Request Forgery Vulnerabilities Using WAF](https://www.htbridge.com/blog/patching-complex-web-vulnerabilities-using-modsecurity-waf.html#csrf) on the HT Bridge blog.

```ApacheConf
# In summary:
# - If no token exists:
#   - Create a CSRF token using ModSecurity
#   - Send the token as a cookie
# - Add the token to the form post using inserted javascript
# - On POST, check if the cookie value matches the posted value

# If we're on a product details page, create a CSRFToken cookie if none exists:

# Skip this section if we're not on the right page:
SecRule REQUEST_FILENAME "!@beginsWith /product/details" \
    "id:2050,phase:2,pass,nolog,\
    t:none,t:lowercase,t:normalisePath,\
    skipAfter:END_CSRF_TOKEN_CREATION"

    # If there's no token cookie, create one from the UNIQUE_ID:
    # (set a var and an environment variable)
    SecRule &REQUEST_COOKIES:csrf_token "@eq 0" \
        "id:2051, phase:2, chain, pass, nolog, \
         t:none, \
         chain"
        SecRule UNIQUE_ID "^(.*)$" \
            "capture, t:none, t:sha1, t:hexEncode, setvar:TX.csrf_token=%{TX.1}, setenv:new_csrf_token=%{TX.1}"

    # Also, if there's a cookie, but it's the wrong format, create one:
    SecRule REQUEST_COOKIES:csrf_token "!^([0-9a-f]{40})$" \
        "id:2052, phase:2, pass, nolog, \
         t:none, t:lowercase, \
         chain"
        SecRule UNIQUE_ID "^(.*)$" \
        "capture, t:none, t:sha1, t:hexEncode, setvar:TX.csrf_token=%{TX.1}, setenv:new_csrf_token=%{TX.1}"

    # If an environment variable new_csrf_token is set, then send a new cookie in the headers
    Header set Set-Cookie "csrf_token=%{new_csrf_token}e; path=/; httponly; secure" env=new_csrf_token

SecMarker END_CSRF_TOKEN_CREATION

# If there is a good cookie we can grab it and set it in csrf_token
SecRule REQUEST_COOKIES:csrf_token "^([0-9a-f]{40})$" \
    "id:2053, phase:2, pass, nolog, \
        t:none, t:lowercase, setvar:TX.csrf_token=%{REQUEST_COOKIES.csrf_token}"

# Hash the CSRF token used in the cookie with a secret, to make it ready to insert in the page
SecRule TX:csrf_token "^(.*)$" \
    "id:2054, phase:2, capture, pass, nolog, \
     t:none, \
     setvar:TX.csrf_token_sha_input=%{TX.1}REPLACESECRETHERE"
SecRule TX:csrf_token_sha_input "^(.*)$" \
    "id:2055, phase:2, capture, pass, nolog, \
     t:none, t:sha1, t:hexEncode, \
     setvar:TX.csrf_token_sha=%{TX.1}"

# If we're on a product details page, insert a CSRFToken hidden field:

# Skip this section if we're not on the right page:
SecRule REQUEST_FILENAME "!@beginsWith /product/details" \
    "id:2060,phase:2,pass,nolog,\
     t:none,t:lowercase,t:normalisePath,\
     skipAfter:END_CSRF_JS_INSERTION"

    SecContentInjection On

    SecRule &TX:csrf_token "@eq 1" \
        "id:2061, phase:2, pass, nolog, t:none, append:'\
        <script>\
            var csrf_token_sha = \'%{TX.csrf_token_sha}\';\
            document.forms[2].action += \'?csrf_token_sha=\' + csrf_token_sha;\
        </script>'"

SecMarker END_CSRF_JS_INSERTION

# If there's a post to /Product/AddComment, then check the token is correct
SecRule REQUEST_FILENAME "@beginsWith /product/addcomment" \
    "id:2062,phase:2,deny,log,\
     t:none,t:lowercase,t:normalisePath\
     msg:'CSRF Token check failed - No arg token provided Cookie:%{TX.csrf_token} SHA-Cookie:%{TX.csrf_token_sha}',\
     chain"
    SecRule REQUEST_METHOD "@streq POST" \
        "t:none,\
         chain"
        SecRule &ARGS_GET:csrf_token_sha "!@eq 1"

SecRule REQUEST_FILENAME "@beginsWith /product/addcomment" \
    "id:2063,phase:2,deny,log,\
     t:none,t:lowercase,t:normalisePath\
     msg:'CSRF Token check failed Arg:%{ARGS_GET.csrf_token_sha} Cookie:%{TX.csrf_token} SHA-Cookie:%{TX.csrf_token_sha}',\
     chain"
    SecRule REQUEST_METHOD "@streq POST" \
        "t:none,\
         chain"
        SecRule ARGS_GET:csrf_token_sha "!@streq %{TX.csrf_token_sha}"

```


Be sure to enter your own secret value in place of `REPLACESECRETHERE`.

