# Validate a parameter

## Architecture layer: modsecurity

Use when you want to use a regular express to validate that a query string or form parameter contains the right data.

```ApacheConf
SecRule ARGS:/^ProductChoices\[.*\].Quantity$/ "!@rx ^\d+$" \
    "id:10050, phase:2, deny, log, \
     t:none, t:removeWhitespace, \
     msg:'Invalid quantity entered: %{MATCHED_VAR}'"
```

This rule looks at all ARGS (query string and post parameters) where the name matches the regular expression ```^ProductChoices\[.*\].Quantity$```

The value of each paramter is compared to the regular expression ```^\d+$```, and if the input is not all digits, then the rule triggers and denys the request with an error message logged.

You will need to update the ID to a number not already in use, and you may want to modify the message.
