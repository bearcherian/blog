---
comments: true
date: 2014-06-18T00:00:00Z
image:
  credit: null
  creditlink: null
  feature: null
modified: 2014-06-18 09:20:44 -0400
share: true
tags:
- CQ
- AEM
- images
- ADA compliance
- acessibility
title: Smart ALT field validation
url: /2014/06/18/smart-alt-field-validation/
---

We had a requirement to enforce alt attributes on images. Sounds simple, just set the alt field to required. The problem is when you do that for components that use images, like Text & Image. If a user decides to enter text, but not use the image portion the ALT text is still required, and the alert for a missing field still appears.

To fix this we add a function to the 'validator' property of the ALT Text field using CQs ExtJS. The function below invalidates the field only if the field is empty, and the image component has data. If there is no image, then the field value is not checked.

```
function(value) {
    var imgComponent = this.findParentByType('dialog').findByType('html5smartimage');
    var imgHasData = imgComponent[0].hasData();
    return ( !imgHasData || (imgHasData &amp;&amp; value != '') );
}
```
