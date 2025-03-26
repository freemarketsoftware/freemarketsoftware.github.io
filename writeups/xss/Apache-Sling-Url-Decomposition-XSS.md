
Using Apache Sling URL decomposition to obtain XSS

```

function switchLanguage() {
  const newLocation = location.href.replaceAll("en", "fr")
  sendAnalyticsEvent(newLocation)
}

function sendAnalyticsEvent(link, site, page, value) {
    try {
        _gaq.push(['_trackEvent', site, page, value]);
        window.setTimeout("window.location.href='" + link + "'", 200);
        return false;
    } catch (err) {}
}
```
