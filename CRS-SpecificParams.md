# Running CRS rules only on certain parameters

The OWASP ModSecurity Core Rule Set has a lot of protections for common web attacks built in, and is tuned to cause a minimum of false alerts. Use the guidance in the documentation, a high anomaly score threshold, a low paranoia level setting and you’re unlikely to have legitimate traffic blocked when you first install and set up ModSecurity with the CRS.

However, sometimes you only want protection on certain known vulnerable parameters, perhaps only on certain pages. The Core Rule Set isn’t really designed for this use case, and is intended for site-wide use.

For my scenario, I only want to block SQL injection on one page – as I’m certain that no other SQL injection exists on our site. I want users to be able to enter text freely in other text fields of our website, even if they happen to be writing a message with an SQL snippet it it.

The url with the vulnerability is: https://example.org/Product/Search?SearchTerm=userinput

If the user enters an SQL fragment into the URL, then the SQL injection vulnerability means they can potentially run arbitrary commands against my database (see OWASP’s SQL Injection information).

## Running the rules only on an individual parameter

I want to use the Core Rule Set, but disable the SQL injection rules unless the input is the SearchTerm argument:

```ApacheConf
Include modsecurity/crs-setup.conf
Include crs/*.conf
	
# Configure-Time: Disable CRS's SQLi rules:
	
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES"
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS"
SecRuleUpdateTargetByID 942100-942999 "!XML"

# Configure-Time: Only test SQLi for the SearchTerm parameter
	
SecRuleUpdateTargetByID 942100-942999 "ARGS:SearchTerm"
```

Let’s dissect what we’ve done here.

The rules we are adding are Configure-Time rules. This means that they run once when ModSecurity initialises, rather than Runtime rules which run on every request.

Configure-Time rules need to be placed after you include the Core Rule set configuration files, so they can permanently modify the rules that have been included.

Rules in the Core Rule Set v3 are grouped into different files according to attack type, and there’s a pretty solid and understandable numbering scheme for rules within each file. SQL injection rules are in the file: `REQUEST-942-APPLICATION-ATTACK-SQLI.conf`

The “942” in the filename gives you a hint that all the rules in that file are numbered from 942000-942999. Each file has a set of structural rules from 94×000-94×099 (i.e. 942000-942099 for the SQLi file) that should be left unaltered, but all the rules above 100 are fair game for us to alter!

An example rule in the file is 942100, which checks for SQL injection using the libinjection library:

```ApacheConf
SecRule REQUEST_COOKIES|!REQUEST_COOKIES:/__utm/|REQUEST_COOKIES_NAMES|REQUEST_HEADERS:User-Agent|REQUEST_HEADERS:Referer|ARGS_NAMES|ARGS|XML:/* "@detectSQLi" \
"msg:'SQL Injection Attack Detected via libinjection',\
Id:942100,\
...
```

You can see that the rule scans across several collections of inputs, looking for SQL injection: REQUEST\_COOKIES, REQUEST\_COOKIE_NAMES, ARGS\_NAMES, ARGS and XML, as well as a couple of specific inclusions and exclusions.

The SQL injection rules are numbered from 942100-94299, and most of them scan across the same collections of inputs. I want to disable scanning across these broad collections, so that any values can be entered into them without fear of SQL injection alerts triggering.

Putting this information together, we remove these collections from scanning for the SQL injection rules numbered 942100-942999:

```ApacheConf
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES"
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS"
SecRuleUpdateTargetByID 942100-942999 "!XML"
```

Now we can test our site, and see that we can enter SQL fragments into any field. We now need to block SQL injection from entry into our SearchTerms parameter.

Luckily this is very easy — all we need to do is turn back on the rules for just the one target we care about:

```ApacheConf
SecRuleUpdateTargetByID 942100-942999 "ARGS:SearchTerm"
```

In summary, our snippet of rules alters the SQL injection ruleset to remove the generic collections from being scanned, and adds back just our particular argument.

##Running Core Rule Set rules only on a certain page and parameter

You’ll note that the above rules enable SQL injection checking across our entire site – that means that if the SearchTerm parameter name is used on other pages, that will also get scanned for SQLi.

In many cases this is what you want, but sometimes you want to only check a parameter on a single url.

This needs to be done at runtime, by inspecting the request and looking at the requested URI in order to make a decision. To do this, you combine the Configure-Time rules with a Runtime rule that checks the url is correct:

```ApacheConf
# Runtime: Only run the SQLi rules against SearchTerm
# if on the Product Search page
	
SecRule REQUEST_FILENAME "!@beginsWith /product/search" \
"id:2011,phase:2,\
pass,nolog,\
t:none,t:lowercase,t:normalisePath,\
ctl:ruleRemoveTargetById=942100-942999;ARGS:SearchTerm"
	
Include modsecurity/crs-setup.conf
Include crs/*.conf
	
# Configure-Time: Disable CRS's SQLi rules:
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES"
SecRuleUpdateTargetByID 942100-942999 "!REQUEST_COOKIES_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS_NAMES"
SecRuleUpdateTargetByID 942100-942999 "!ARGS"
SecRuleUpdateTargetByID 942100-942999 "!XML"
	
# Configure-Time: Only test SQLi for the SearchTerm & foo parameters
SecRuleUpdateTargetByID 942100-942999 "ARGS:SearchTerm"
```

The new rule is the first SecRule above. To understand why this needs to go before the includes of the CRS *.conf files, we need to consider how execution occurs.

For a given phase (such as phase 2 in our example), the rules are processed in the order that they appear in the configuration. When the SQL injection rules are executed they will alert immediately, so if we want to alter the situations they will execute in, we need to modify them before the rules run.

To modify rules at runtime we use the ctl: actions, which can change rules that haven’t yet executed. By placing our SecRule before the CRS *.conf files are included, we can guarantee that our rule will run first and modify the CRS SQL injection rules before they run.

Dissecting this new rule, it checks the path of the request, and for all other URL’s that aren’t the one we want scanned, it removes the SearchTerm argument. It feels back-to-front I know, but this seems to be the only way to achieve what we want!

Summarising our new rules, we alter the Core Rule Set’s SQL injection rules to stop them scanning across all parameters, and then add back just the SearchTerm parameter. To stop that parameter from running on all pages, we then remove it again if the current request isn’t for the Product Search page.

Many thanks to Christian Folini for helping me come up with these rules, and to the other folks on the OWASP ModSecurity Core Rule Set mailing list.