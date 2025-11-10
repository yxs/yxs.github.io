+++
title = 'Set Up Blog'
date = '2025-11-09T17:15:24-08:00'
summary = ""
tags = []
categories = []
showToc = true
TocOpen = false
slug = "set-up-blog"
+++


I recommend this theme https://github.com/adityatelange/hugo-PaperMod/

add Azure Front Door(CDN)

xuesong-cxa7edcca8a7cgc4.z01.azurefd.net

add Azure Application Insights

```html
<!-- Azure Application Insights -->
<script type="text/javascript">
    (function (c, o, n, f, i, g) {
        c[f] = c[f] || function () { (c[f].q = c[f].q || []).push(arguments) };
        i = o.createElement(n); i.async = 1; i.src = "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js";
        g = o.getElementsByTagName(n)[0]; g.parentNode.insertBefore(i, g);
    })(window, document, "script", "appInsights");
    appInsights("init", {
        instrumentationKey: "YOUR_INSTRUMENTATION_KEY"
    });
    appInsights("trackPageView");
</script>
```