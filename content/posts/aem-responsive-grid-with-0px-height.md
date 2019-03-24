---
title: AEM Responsive Grid with 0px height
date: 2019-01-15T07:19:25-04:00
published: true
description: When creating a new page component in AEM, the parsys has a height of 0 and cannot be used.
tags: 
- aem
- experience manager
- HTL
- sightly
---

In AEM when you create a new page component with a editable area that uses the responsive grid (`wcm/foundation/components/responsivegrid`) you might experience an issue where the editable area appears, but has no height, and is unusable. All you get is a thin blue line where the editable area should be.

![Screenshot of unusable editable area](https://thepracticaldev.s3.amazonaws.com/i/fjyznpqt1kqgzexdvwe4.png)

The cause of this is a missing include. To initialize the AEM editing features, an JSP is loaded that initializes all of the editable areas, as well as some other things. 

The solution is to add the following in your `<head>` of your page component

```html
    <!--/* Initializes the Experience Manager authoring UI */-->
    <sly data-sly-include="/libs/wcm/core/components/init/init.jsp" data-sly-unwrap/>
```

You can remove the comment, but I like to leave it in so I, or anyone else editing the page component, knows why we have it in there. Now you'll see your editable area ready to add components.

![Screenshot of working editable area](https://thepracticaldev.s3.amazonaws.com/i/f0l4q5udxc8ueo4kqlr8.png)

Thanks to user manasip9 for posting [their answer on this thread](https://forums.adobe.com/message/9942397#9942397)  on the Adobe forums.