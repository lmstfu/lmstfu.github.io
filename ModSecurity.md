## About Mod Security

[ModSecurity](https://www.modsecurity.org/) is a web application firewall that runs inside a webserver. This is an open source module that allows rules to be configured to block, modify or otherwise inspect web traffic.

ModSecurity was originally an Apache HTTP Server module, but subsequent releases extended it to support IIS and nginx webservers.

The current stable version is v2.9.x, and can run in all three web servers. However, as ModSecurity was built originally to run in Apache, it is a bit less stable in the other web servers.

Version 3 of ModSecurity is a re-write that extracts the core of ModSecurity into a platform independent library called libmodsecurity. Each web server then has a connector built allowing libmodsecurity to run within that platform with the most performance.

Version 3 is currently in release candidate, and currently supports many of the functions of 2.9.x. Some functionality has not yet been ported, and there may be some instability due to the release candidate status.

ModSecurity is quite safe to use out of the box as it installs in Detection Only Mode - which means it won't block any traffic or cause false positives. 

Out of the box, ModSecurity doesn't do much - it needs rules to be configured to unleash it's power. The [OWASP ModSecurity Core Rule Set](https://coreruleset.org/) is a set of attack detection rules that have been collated and tuned by experts.

The Core Rule Set has been tuned to avoid false positives, and contains rules for some common web attacks:

* SQL Injection (SQLi)
* Cross Site Scripting (XSS)
* Local File Inclusion (LFI)
* Remote File Inclusion (RFI)
* Remote Code Execution (RCE)
* PHP Code Injection
* HTTPoxy
* Shellshock
* Session Fixation
* Scanner Detection
* Metadata/Error Leakages
* GeoIP Country Blocking

### Our ModSecurity approach

When virtual patching issues, we want to be certain that we are only fixing known issues that we have confirmed exist on our site (see [Vulnerabilities or Weaknesses](VulnerabilitiesOrWeaknesses)). We don't want to put in place blanket patches for issues that *might* exist, because then we are increasing the chances of blocking legimate traffic as false positives.

This means that when writing a security rule in ModSecurity, we will almost always be scoping it to an individual url and parameter rather than applying it site-wide.

Unfortunately, while the Core Rule Set is an awesome place to start, it doesn't support this granular approach out of the box. We need to follow the process discussed in [Running CRS rules only on certain parameters](CRS-SpecificParams) to narrowly target groups of rules.

### More information

As always, Google is your friend when finding ModSecurity information - but beware with old articles, which may be out of date.

Resources we recommend:

* [ModSecurity website](http://modsecurity.org/)
* [OWASP ModSecurity Core Rule Set](https://coreruleset.org/) 
* [ModSecurity Handbook](https://www.feistyduck.com/books/modsecurity-handbook/) by Christian Folini and Ivan Ristić - print / ebook
* [netnea Apache / ModSecurity Tutorials](https://www.netnea.com/cms/apache-tutorials/)
* [Trustwave SpiderLabs Blog](https://www.trustwave.com/Resources/SpiderLabs-Blog/?tag=ModSecurity&LangType=1033)
