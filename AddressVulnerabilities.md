## How to address vulnerabilities

### Make a list

The first step is to make a list of all the vulnerabilites or weaknesses in your site, and put them in one place. This may be as simple as a Google Sheet, or somewhere you can track them over time such as JIRA or Trello.

### Assign an ID

Assign each security issue an ID, so that you can keep track of the changes you make to address each issue.

For example, here's the list of vulnerabilities and IDs for a vulnerable application:

* SQL Injection in Product search (V17)
* Cross-site scripting in Product Comments (V15)
* Cross-site Request Forgery in Product Comments (V16)
* Cookies not set with HTTPOnly and Secure (V13)
* Shopping cart allows negative quantities of each item (V5)
* Jumping past the payment screen in the shopping cart (V7)
* Disclosure of credit card information in hidden fields and order history (V8)

### Prioritise

Order your list of issues so that the most important issues can be addressed first. You might find it helpful to discuss them with other people to get multiple perspectives on which issues are the most pressing.

### Understand how to exploit the issue

For vulnerabilities, you'll need to find the payload that will trigger the vulnerability. You could discover this through reading vendor documentation, full-disclosure emails or blog posts, or by reading the source of metasploit modules.

For weaknesses you'll typically have a scenario that explains the steps a user will take and the undesirable outcome.

The main point of this step is to know how to reproduce the issue so that:

1. You can prove that it exists in the original application
2. You can test any virtual patches you make
3. You can prove that it is fixed

### Create a virtual patch

This is likely to be an iterative process, and other pages on this site may help with this.

We recommend that you collect all artifacts, configuration, rules and code required for the virtual patch in well-commented sections and files that relate back to the issue ID. That way you'll have a linkage to the underlying issue that is being addressed and it will be easy to navigate backwards and forwards.

For example, for "SQL Injection in Product search (V17)" we created the following files:

* modsecurity/crs-configtime-rules/v17-configtime.conf
* modsecurity/crs-runtime-rules/v17-runtime.conf

Of course, your folder and filename standards may be different.