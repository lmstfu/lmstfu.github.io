# Detect SQL Injection

## Architecture layer: modsecurity

Use when you have a parameter vulnerable to SQL Injection and you want to block any suspicious input.

Like XSS, this is a task that modsecurity & the core rule set excel at!

However, to scope the SQLi detection to just an individual parameter currently requires a fair bit of work. See [Running CRS rules only on certain parameters](CRS-SpecificParams) for the details on tbe rationale.

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
# Runtime: Only run the SQLi rules against SearchTerm if on the Product Search page
SecRule REQUEST_FILENAME "!@beginsWith /product/search" \
    "id:2011,phase:2,\
        pass,nolog,\
        t:none,t:lowercase,t:normalisePath,\
        ctl:ruleRemoveTargetById=942100-942999;ARGS:SearchTerm"
```

Config-time rule:

```ApacheConf
# Configure-Time: Disable CRS's SQLi rules:
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES"
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS"
SecRuleUpdateTargetByID 942100-942999 "!XML"

# Configure-Time: Only test SQLi for the SearchTerm parameter
SecRuleUpdateTargetByID 942100-942999 "ARGS:SearchTerm"
```

To summarise - we let the Core Rule Set scan all inputs for SQLi, however we then remove all rules and targets apart from the one page and parameter we want to check.