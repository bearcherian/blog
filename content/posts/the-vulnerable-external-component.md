---
comments: true
date: 2014-11-12T00:00:00Z
image:
  credit: null
  creditlink: null
  feature: null
modified: 2014-11-12 12:52:43 -0500
share: true
tags:
- CQ
- external
- iframe
- security
title: The Vulnerable External Component
---

***Update (2014.11.24):** The post below references an issue with a Component in CQ 5.5. I finally heard back from Adobe support and the issue has been resolved in CQ 5.6.1 by removing the Adaptive feature. They provided [this forum post](http://help-forums.adobe.com/content/adobeforums/en/experience-manager-forum/adobe-experience-manager.topic.312.html/forum__75v9-hi_in_cq_5_5the.html) as a reference.*

Recently I was perusing crawl errors from [Google's Webmaster Tools](https://www.google.com/webmasters/tools/) to find any issues that had not yet been reported. I came across dozens of 500 errors coming from requests to one particular page. When I went to view the page I saw our template mangled with content from an Arabic news site. What was going on?!

The page itself, without any parameters, was simply a templated page with an external component loading a basic HTML form hosted on another server. Each 500-error request had a query parameter of `CFC__target`, which was set to a URL on an Arabic news site. So a page at `path/to/page.html` appeared as `path/to/page.html?CFC__target=http://www.somesite.com`. If you're trying this out on your CQ environment, note the double underscore.

I replaced the news site link with a link of my own. Lo' and behold, now my link loaded up, mangling the content of the original page. I edited the page and changed the mode from Adaptive to Fixed and tried using the `CFC__target` again. This time the page wasn't mangled, but the content of the URL in the `CFC__target` parameter loaded up in the iframe.

### Understanding the External Component

The External component has very little code in the JSPs. It totals about 15 lines of code across two JSPs, not including dialog, comments, and imports. Most of the work is done by [com.day.cq.wcm.foundation.External](http://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/wcm/foundation/External.html).

In the dialog, the External component has two Inclusion mode options: "Fixed" and "Adaptive". In "Fixed" mode, the target URL is placed into an iframe. Simple and easy. This is not always ideal because the layout and style of the remote content does not fit well in an iframe. 

In "Adaptive" mode, the HTML of the target is placed on the page itself, and all of the links are rewritten to be relative paths. This way the content fits in nicely on the page, you don't have to guess on the height of the content, and you don't have to worry about CORS issues.

The problem is how the adaptive mode work is done. Each link is rewritten to point to the page itself using a query parameter to read in the resource from the remote site. So if the page at the target URL includes a JavaScript file at `http://www.somesite.com/js/main.js`, it is rewritten as `path/to/page/jcr:content/par/external.spool?CFC__target=http://www.somesite.com/js/main.js`.

### What's the risk?

There doesn't seem to be a risk here to the server, at least not one that I could identify off the bat. The main risk here is to your end users.

Knowing how the External Component works, hackers, phishers, and scammers now have a way to mimic your organizations site. Just create a fake logon page, add it as a parameter via `CFC__target`, and they now have a legitimate looking logon page with a legitimate looking URL that can be emailed out to phish usernames and passwords.

### How do you fix it?

There are a few options to fix this. 

1. **Remove the code that uses CFC__Target.** This requires simply commenting out the lines that read in the request parameter and sets the external object's target. The drawback is it effectively breaks the Adaptive mode because the rewritten links no longer function correctly.
2. **Change the parameter name.** This is probably the least effective. If removed the fact that the parameter `CFC__target` can be used, but the new target parameter name is exposed by any resources referenced from the target URL.
3. **Fix the External Class.** This would be really nice, but since it is proprietary Adobe code, it's up to Adobe to fix this. I've opened a ticket with Adobe support on this and am waiting for a response.
3. **Disable the External Component.** Seems a bit drastic, but it might be the best choice for you. For us, at this point, this isn't an option since we have hundred of pages that currently use this component. If you do decide to go this route, you will probably want to write your own component for loading content via iframe. But then that would be the same as option 1.

Whatever option you choose, it's important to address this on your CQ instance to avoid your site being used to carry out malicious attacks.
