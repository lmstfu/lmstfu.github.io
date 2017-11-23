# Alter cookie flags

## Architecture layer: Apache

Use when a cookie is missing the HttpOnly or secure flags.

This virtual patch uses Apache's mod_headers rather than ModSecurity or node.js.

```ApacheConf
# Add missing HttpOnly flag
Header edit Set-Cookie "(?i)^(.AspNetCore.Antiforgery.(?:(?!httponly).)+)$" "$1; HttpOnly"

# Add missing Secure flag
Header edit Set-Cookie "(?i)^(.AspNetCore.Antiforgery.(?:(?!secure).)+)$" "$1; Secure"
```

This example replaces an existing Set-Cookie header sent by the origin server. It uses a regular expression to check if the original header has the httponly or secure flags, and if not, replaces that header with one including the flag.

Note also that this shows how to use a regular expression to match multiple cookie names in the case where the webserver cookie names change every response.