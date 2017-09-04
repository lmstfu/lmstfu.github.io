# Laying out your SecRules

A SecRule is the most common configuration item you'll create for ModSecurity. The SecRule has the following syntax:

```ApacheConf
SecRule VARIABLES OPERATOR [ACTIONS]
```
	
Even though there are 3 arguments to the rule, this is a side-effect of the Apache configuration syntax - actually you create multiple arguments within these.

A simple example is this rule for blocking access to a url:

```ApacheConf
SecRule  REQUEST_FILENAME "@rx /order/details/" \
  "id:11101,phase:1,deny,log,\
  t:none,t:lowercase,t:normalisePath,\
  msg:'Blocking access to %{MATCHED_VAR}'"
```

We recommend following the below stylistic conventions:

* Split the rule onto multiple lines with `\`
* Split the `ACTION` argument across multiple lines
* First Line:
	* The first line of the rule contains just the `VARIABLES` and the `OPERATOR`
* Second Line:
	* 	`id` Put the ID first to make it easy to find rules by ID
	*  *Always* specify the `phase` that the rule is running in - omitting it will lead to confusing bugs! (See [Processing Phases](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual#Processing_Phases))
	*  Specify the Disruptive Action (deny, block, pass, allow)
	*  Specify whether you want to log, nolog, auditlog or noauditlog
*  Third line:
	*  Use this line for text transformations, always starting with t:none
*  Fourth line:
	*  A message to output in the logs
	*  Any tags you want to add
*  Fifth line:
	* `chain` if you want to join this to the next SecRule
