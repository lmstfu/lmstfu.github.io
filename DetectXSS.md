# Detect XSS

## Architecture layer: modsecurity

Use when you have a parameter vulnerable to XSS and you want to block any suspicious input.

This is a task that modsecurity & the core rule set excel at!

However, to scope the XSS detection to just an individual parameter currently requires a fair bit of work. See [Running CRS rules only on certain parameters](CRS-SpecificParams) for the details on tbe rationale.

Basically, you need to split your rule to address this vulnerability into two parts: Those that run before CRS is included in Apache (runtime rules), and those that run after the CRS include (config time rules). I recommend splitting the configuration into two files in two separate folders to facilitate this.

```ApacheConf
# Some vulns require customising CRS at runtime:
Include modsecurity/crs-runtime-rules/*.conf

# Include the ModSecurity Core Rule Set:
# (Note: We currently only use the SQLi and XSS attack rulesets)
Include modsecurity/crs-setup.conf
Include crs/*.conf

# Some vulns require customising CRS at config-time:
Include modsecurity/crs-configtime-rules/*.conf
```

Runtime rule:

```ApacheConf
# Rutime: Only run the XSS rules against Comment if submitting to the product add comment url
SecRule REQUEST_FILENAME "!@beginsWith /product/addcomment" \
    "id:2012,phase:2,\
        pass,nolog,\
        t:none,t:lowercase,t:normalisePath,\
        ctl:ruleRemoveTargetById=941100-941999;ARGS:Comment"
```

Config-time rule:

```ApacheConf
# The XSS rules in CRS all work on the following variables:
# REQUEST_COOKIES|!REQUEST_COOKIES:/__utm/|REQUEST_COOKIES_NAMES|REQUEST_HEADERS:User-Agent|REQUEST_HEADERS:Referer|ARGS_NAMES|ARGS|XML
# (Some don't have the request headers)

# Configure-Time: Disable CRS's XSS rules:
SecRuleUpdateTargetByID 941100-941999 "!REQUEST_COOKIES"
SecRuleUpdateTargetByID 941100-941999 "!REQUEST_COOKIES_NAMES"
SecRuleUpdateTargetByID 941100-941999 "!ARGS_NAMES"
SecRuleUpdateTargetByID 941100-941999 "!ARGS"
SecRuleUpdateTargetByID 941100-941999 "!XML"
SecRuleUpdateTargetByID 941100-941999 "!REQUEST_HEADERS:User-Agent"
SecRuleUpdateTargetByID 941100-941999 "!REQUEST_HEADERS:Referer"

# Configure-Time: Only test SQLi for the Comment parameter
SecRuleUpdateTargetByID 941100-941999 "ARGS:Comment"
```

To summarise - we let the Core Rule Set scan all inputs for XSS, however we then remove all rules and targets apart from the one page and parameter we want to check.