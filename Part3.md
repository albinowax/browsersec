<h1>Browser Security Handbook, part 3</h1>

  * Written and maintained by [Michal Zalewski](http://lcamtuf.coredump.cx/) <[lcamtuf@google.com](mailto:lcamtuf@google.com)>.
  * Copyright 2008, 2009 Google Inc, rights reserved.
  * Released under terms and conditions of the [CC-3.0-BY](http://creativecommons.org/licenses/by/3.0) license.

<h1>Table of Contents</h1>

  * _[← Back to browser security features](http://code.google.com/p/browsersec/wiki/Part2)_


# Experimental and legacy security mechanisms #

Through the years, browsers accrued a fair number of security mechanisms that had either fallen into disuse, or never caught on across more than a single vendor; as well as a number of ongoing proposals, enhancements, and extensions that are yet to prove their worth or become even vaguely standardized. This section provides a brief overview of several technologies that could fall into this class.

_Fun fact: when it comes to newly proposed features, many of them essentially introduce new security boundaries and permission systems mostly orthogonal, yet intertwined, with same-origin controls. Although this appears to be a good idea at first sight, some researchers [warn about the pitfalls](http://crypto.stanford.edu/websec/origins/fgo.pdf) of finer-grained origins as difficult to understand to users, and hard to fully examine for potential side effects and interactions with existing code._

## HTTP authentication ##

HTTP authentication is an ancient mechanism most recently laid out in [RFC 2617](http://www.ietf.org/rfc/rfc2617). It is a simple extension of HTTP:

  * Any resource for which the server requires the user to provide valid credentials would initially return a `401 Unauthorized` HTTP error code, with a `WWW-Authenticate` header describing parameters such as the authentication realm, supported authentication modes, and other method-specific parameters.

  * Upon receiving a `401` code, the client is expected to prompt the user for login and password, or obtain the data from in-memory cache or another credential store (where according to the RFC, it should be looked up by authentication realm name alone; although for security purposes, it should be bound to the host name as well, preferably also to port and protocol).

  * User's credentials are then encoded and sent back in a new request with an additional `Authorization` header, which the server is expected to examine, and grant access or return an error message as appropriate.

Credentials are also cached for authentication with other subresources on the same site, and are sent out on future requests in an unsolicited manner (globally, or only for a particular path prefix).

Two key authentication schemes are supported by virtually all clients: `basic` and `digest`. The former simply sends user names and passwords in plain (`base64`) text - and hence the data is vulnerable to snooping, unless the process takes place over HTTPS. The latter uses a simple nonce-based challenge-response mechanism that prevents immediate password disclosure. Microsoft further extended the mechanism to include two proprietary `NTLM` and `Negotiate` schemes ([reference](http://www.innovation.ch/personal/ronald/ntlm.html)) that integrate seamlessly with Microsoft Windows domain authentication.

As hinted in the section on [URL syntax](http://code.google.com/p/browsersec/wiki/Part1#Uniform_Resource_Locators), URLs are permitted to have an optional `user[:password]@` segment immediately before the host name, to enable pre-authenticated bookmarks or shared links. In practice, the mechanism would be seldom used for legitimate purposes, but became immensely popular with phishers - who would often construct URLs such as `http://`<font color='#ff0000'><code>www.example-bank.com:something_something@</code></font>`www.evilsite.com/` in hopes of confusing the user. This led to this URL syntax being banned in Microsoft Internet Explorer, and often resulting in security prompts elsewhere.

Because of these limitations and the relative inflexibility of this scheme to begin with, HTTP authentication has been almost completely extinct on the Internet, and replaced with custom solutions built around HTTP cookies (it is still sometimes used for intranet applications or for simple access control for personal resources).

Amusingly, its ghost still haunts modern web applications: HTTP authentication prompts often come up in browsers when viewing trusted pages where a minor authentication-requiring sub-resource, such as `<IMG>`, is included from a rogue site - but these prompts usually do a poor job of clearly explaining who is asking for the credentials. This poses a phishing risk for services, such as blogs or discussion forums, that allow users to embed external content.

| <font color='#3050a0'><b>Test description</b></font> | <font color='#3050a0'><b>MSIE6</b></font> | <font color='#3050a0'><b>MSIE7</b></font> | <font color='#3050a0'><b>MSIE8</b></font> | <font color='#3050a0'><b>FF2</b></font> | <font color='#3050a0'><b>FF3</b></font> | <font color='#3050a0'><b>Safari</b></font> | <font color='#3050a0'><b>Opera</b></font> | <font color='#3050a0'><b>Chrome</b></font> | <font color='#3050a0'><b>Android</b></font> |
|:-----------------------------------------------------|:------------------------------------------|:------------------------------------------|:------------------------------------------|:----------------------------------------|:----------------------------------------|:-------------------------------------------|:------------------------------------------|:-------------------------------------------|:--------------------------------------------|
| <font color='#3050a0'>Is HTTP authentication supported?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#ff0000'>NO</font>             |
| <font color='#3050a0'>Does link-embedded authentication work?</font> | NO                                        | NO                                        | NO                                        | prompt                                  | prompt                                  | YES                                        | prompt                                    | YES                                        | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Is authentication data bound to realms?</font> | NO                                        | NO                                        | NO                                        | NO                                      | NO                                      | <font color='#30ff30'>YES</font>           | NO                                        | <font color='#30ff30'>YES</font>           | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Is authentication bound to host name?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Is authentication bound to protocol or port?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Are cached credentials sent out on all future requests?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | <font color='#30ff30'>subdir only</font>   | <font color='#30ff30'>subdir only</font>  | <font color='#30ff30'>subdir only</font>   | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Do password prompts come up on <code>&lt;IMG&gt;</code>?</font> | <font color='#ff0000'>YES</font>          | <font color='#ff0000'>YES</font>          | <font color='#ff0000'>YES</font>          | <font color='#ff0000'>YES</font>        | <font color='#ff0000'>YES</font>        | <font color='#ff0000'>YES</font>           | <font color='#ff0000'>YES</font>          | <font color='#ff0000'>YES</font>           | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Do password prompts come up on <code>&lt;EMBED&gt;</code> / <code>&lt;APPLET&gt;</code>?</font> | NO                                        | NO                                        | NO                                        | <font color='#ff0000'>YES</font>        | <font color='#ff0000'>YES</font>        | NO                                         | <font color='#ff0000'>YES</font>          | <font color='#ff0000'>YES</font>           | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Do password prompts come up on <code>&lt;SCRIPT&gt;</code> and stylesheets?</font>  | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#909090'>n/a</font>            |
| <font color='#3050a0'>Do password prompts come up on <code>&lt;IFRAME&gt;</code>?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#909090'>n/a</font>            |

<b></b>

## Name look-ahead and content prefetching ##

Several browsers are toying with the idea of prefetching certain information that, to their best knowledge, is likely to be needed in the immediate future. For example, Firefox offers an option to perform a background prefetch of certain links specified by page authors ([reference](https://developer.mozilla.org/En/Link_prefetching_FAQ)); and Chrome is willing to carry out look-ahead DNS lookups on links on a page ([reference](http://dev.chromium.org/developers/design-documents/dns-prefetching#TOC-DNS-Prefetch-Control)). The idea here is that if the user indeed takes the action anticipated by browser developers or page owners, he or she would appreciate the reduced latency achieved through these predictive, background operations.

On the flip side, prefetching has two important caveats: one is that it is generally rather wasteful - as users are very unlikely to follow _all_ prefetched links, and quite often, would not follow any of them; the other is that in certain applications, such look-aheads may reveal privacy-sensitive information to third parties.

One particularly interesting risk scenario is the context of a web mail interface: unless prefetching is properly disabled or limited, the sender of a mail containing a HTTP link might have the ability to detect that the recipient had read his message, even if the link itself is not followed.

## Password managers ##

As a part of a broader form auto-completion mechanism, most browsers offer users the ability to save their passwords on client side, and auto-complete them whenever previously seen authentication forms appear again.

Unfortunately, as noted in the previous section, the only standard mechanism for identifying and scoping user passwords in HTTP traffic had largely fallen into disuse; the new form- and cookie-based alternatives are custom-built for every application, are share almost nothing in common with each other. As a result, most browsers face challenges trying heuristically detect when a legitimate authentication attempt takes place and succeeds, or trying to decide how to define the realm for stored data.

In particular, it might be easier than intuitively expected for certain types of user content hosted (or injected) under the same host name as a trusted login interface to spoof such forms and intercept auto-filled data; letting user-controlled forms and login interfaces mix within a single same-origin context appears to be not necessarily a good idea for time being.

Several of the relevant password manager behaviors are shown below:

| <font color='#3050a0'><b>Test description</b></font> | <font color='#3050a0'><b>MSIE6</b></font> | <font color='#3050a0'><b>MSIE7</b></font> | <font color='#3050a0'><b>MSIE8</b></font> | <font color='#3050a0'><b>FF2</b></font> | <font color='#3050a0'><b>FF3</b></font> | <font color='#3050a0'><b>Safari</b></font> | <font color='#3050a0'><b>Opera</b></font> | <font color='#3050a0'><b>Chrome</b></font> | <font color='#3050a0'><b>Android</b></font> |
|:-----------------------------------------------------|:------------------------------------------|:------------------------------------------|:------------------------------------------|:----------------------------------------|:----------------------------------------|:-------------------------------------------|:------------------------------------------|:-------------------------------------------|:--------------------------------------------|
| <font color='#3050a0'>Password manager operation model</font> | <font color='#30ff30'>needs user name</font> | <font color='#30ff30'>needs user name</font> | <font color='#30ff30'>needs user name</font> | auto-fills on load                      | auto-fills on load                      | auto-fills on load                         | <font color='#30ff30'>needs UI action</font> | auto-fills on load                         | auto-fills on load                          |
| <font color='#3050a0'>Are stored passwords restricted to a full URL path?</font> | <font color='#30ff30'>YES</font>          | <font color='#30ff30'>YES</font>          | <font color='#30ff30'>YES</font>          | NO                                      | NO                                      | NO                                         | NO                                        | NO                                         | NO                                          |
| <font color='#3050a0'>Are stored <code>https</code> passwords restricted to SSL only?</font> | YES                                       | YES                                       | YES                                       | YES                                     | YES                                     | YES                                        | YES                                       | YES                                        | <font color='#ff0000'>NO</font>             |

<b></b>

Robert Chapin offers some additional research into [password manager scoping habits](http://www.info-svc.com/news/2008/12-12/), exploring some non-security considerations as well.

## Microsoft Internet Explorer zone model ##

All current versions of Internet Explorer utilize an interesting concept of [security zones](http://support.microsoft.com/kb/174360), not shared with any other browser on the market. The idea behind the approach is to compartmentalize resources into various categories, depending on the degree of trust given - and then control almost all security settings for these groups separately. The following zones are defined:

  * **My computer:** a container for all local `file:///` resources, with the exception of documents annotated with mark-of-the-web tags. The user has no control over what gets included into this zone, although an `urlmon.dll` API is provided to give certain protocol handlers the ability to define additional mappings ([reference](http://msdn.microsoft.com/en-us/library/ms537133(VS.85).aspx)).

  * **Local intranet:** a container for all sites determined by simple configurable heuristics to belong to the local network (non-FQDN host names, proxy server exception list, content accessed over SMB). The user may list additional host names to include in this zone.

  * **Trusted sites:** an empty container for trusted pages. This group enjoys certain elevated privileges, particularly to run ActiveX controls, install desktop elements, or programatically access the clipboard. The user is expected to populate the zone with trusted sites that require these permissions to operate.

  * **Restricted sites:** an empty container for untrusted pages. This group has many of the rudimentary permissions taken away, such as the ability to initiate file downloads or use `<META HTTP-EQUIV="Refresh" ...>`. The user is expected to populate the zone with sites he would rather approach with caution (presumably ahead of the first planned visit).

  * **Internet:** a default container for all sites on the Internet not included in any of the remaining categories.

This design makes it possible to, for example, fine-tune the permissions of `file:///` resources without impacting Internet sites, or to forbid navigation from "Internet" to "Local intranet" - and from this perspective, appears to offer a major security benefit. On the flip side:

  * Quite a few important security settings are excluded from the zone model, somewhat diminishing its value; for example, cookie permissions or SSL behavior is controlled elsewhere.

  * The number of settings currently offered in this model is remarkably high, and many of them appear to be too finely grained, or have vague or confusing descriptions with security impact not clearly explained (_"Script controls marked as safe for scripting"_, _"Navigate sub-frames across different domains"_), increasing the risk of hard-to-spot configuration errors.

  * The model fails to account for the impact of cross-site scripting flaws in trusted sites. Since users add pages such as Windows Update, their banks, and other legacy sites that may not always work properly in the "Internet" zone to "Trusted sites", giving them a lot of unnecessary permissions as a side effect, the impact of cross-site scripting flaws on these pages may result in the attacker suddenly gaining the ability to carry out dangerous actions normally not available to Internet content.

  * The complexity of the model and the permissive settings of some zones resulted in a [large number of security problems](http://www.securityfocus.com/news/8998) that could easily be avoided by employing a simpler approach instead.

## Microsoft Internet Explorer frame restrictions ##

Another interesting, little-known security feature unique to Microsoft Internet Explorer is the concept of [SECURITY=RESTRICTED frames](http://msdn.microsoft.com/en-us/library/ms534622(VS.85).aspx). The idea behind the mechanism is that some services may have a legitimate use for limiting the ability of data displayed within `<IFRAME>` containers to disrupt the top-level document or abuse same-origin policies - and so, with the `SECURITY=RESTRICTED` attribute, the content may be placed in the "Restricted sites" zone, and - in theory - deprived of the ability to run disruptive scripts or access cookies.

No support outside of Microsoft Internet Explorer makes this feature useless as a security defense for most intents and purposes; but the mechanism needs to be considered for its potential negative security impact, such as the ability to prevent frame busting code from operating properly and making [UI redress attacks](http://code.google.com/p/browsersec/wiki/Part2#Arbitrary_page_mashups_(UI_redressing)) a bit easier. No support for HTTP cookies within a restricted container would normally limit the impact, but flaws in the design are known.

Due to its minimal use, the mechanism likely received very little security scrutiny otherwise.

## HTML5 sandboxed frames ##

A better-developed descendant of the `SECURITY=RESTRICTED` is the current HTML5 proposal for [sandboxed frames](http://www.w3.org/TR/html5/text-level-semantics.html#attr-iframe-sandbox). The design appears to be considerably more robust and offers a better granularity for restricting the behavior of framed sites: embedding domains will be able to disallow scripting altogether; allow Java<b></b>Script but make all same-origin policy checks fail;  or allow scripting, but prevent navigating the top-level document to prevent framebusting.

Of all these features, the ability to place content in a same-origin policy sandbox is the most tricky one. Perhaps most importantly, the attacker could simply open the normally framed document directly (this is prevented by using `text/html-sandboxed` as a MIME type; but [content sniffing logic](http://code.google.com/p/browsersec/wiki/Part2#Survey_of_content_sniffing_behaviors) will render this trick unsafe in certain browsers). The other significant problem is that mechanisms such as local storage and workers (see next chapter), form autocomplete, and so forth, need to be specifically accounted forth and denied access to - something that will likely prove challenging in the long run.

## HTML5 storage, cache, and worker experiments ##

Firefox 2 embraced the idea of [globalStorage](https://developer.mozilla.org/En/DOM/Storage), a somewhat haphazard mechanism that permitted data to be stored persistently for offline use in a local database shared across all sites (and until Firefox 2, with no strong security controls on scoping). The concept of `globalStorage` originated with early HTML5 drafts, although there, got eventually ditched in favor of a more tightly controlled [sessionStorage](http://dev.w3.org/html5/webstorage/#the-sessionstorage-attribute) and [localStorage](http://dev.w3.org/html5/webstorage/#dom-localstorage) APIs, supported by Firefox 3.6, Internet Explorer 8, Opera, and WebKit-based browsers.

Another idea of [SQL database](http://www.w3.org/TR/2009/WD-html5-20090212/structured-client-side-storage.html#sql) support made rounds, but originally failed to gain any support.

_WARNING: The implementation of `sessionStorage` and `localStorage` in Internet Explorer 8 dangerously deviates from the standard, treating storages managed by HTTP and HTTPS content as same-origin. The `sessionStorage` implementation in Firefox 3.6 also permits properties set by HTTP pages to be read by HTTPS content, but not vice versa - which can be problematic._

Further along the lines of offline storage mechanisms, Firefox 3 introduced, and Firefox 3.1 revised in a largely incompatible manner, the concept of [cache manifests](https://developer.mozilla.org/en/Offline_resources_in_Firefox) that permit sites to opt for having certain resources persistently stored in a site-specific cache, and then retrieved from there instead of being looked up on the net. The feature is again borrowed from the ever-shifting HTML5 specification (and now also available in WebKit browsers).

[Web workers](http://www.whatwg.org/specs/web-workers/current-work/) are another HTML5 design along these lines, starting to ship with WebKit browsers. The idea is to permit asynchronous, background Java<b></b>Script execution in a separate process; dedicated workers would be tied to their opener, and terminated when the page closes; shared workers are not bound to their origin, and free to use `postMessage` API to interact with third-party domains; and persistent workers may be allowed to launch and execute across browser sessions with no additional dependencies of any sort. There are still some kinks in the design documents at this point, however.

## Microsoft Internet Explorer XSS filtering ##

An experimental [reflected HTML injection filter](http://blogs.technet.com/swi/archive/2008/08/19/ie-8-xss-filter-architecture-implementation.aspx) is available in Microsoft Internet Explorer 8. The filter attempts to disable any Javascript on the page that seems to be copied from query parameters. The complexity of escaping rules, jiffy handling of certain character sets, and quirky rules for parsing HTML documents, all make it likely that the mechanism would never achieve a 100% coverage - a property that Microsoft [pragmatically acknowledges](http://blogs.msdn.com/dross/archive/2008/07/03/ie8-xss-filter-design-philosophy-in-depth.aspx) in their design proposals. As such, the mechanism would likely serve to complement, rather than replace, proper server-side development practices.

An important downside of the design is that it aims to selectively disable script snippets that appear in URL query parameters, instead of displaying an interstitial or other security prompt, and permitting the page to display in full, or not display at all. This behavior may potentially enable attackers to carefully take out [frame-busting code](http://code.google.com/p/browsersec/wiki/Part2#Arbitrary_page_mashups_(UI_redressing)) or other security-critical components of targeted applications, while leaving the rest of the functionality intact.

The filter also features a same-origin exception that inhibits filtering when site-supplied links are followed. In many complex web applications, this may be used to bypass the mechanism altogether.

## Script restriction frameworks ##

Several experimental browser-side or client-side designs aim to provide control over same-origin permissions for scripts, or over when scripts may execute on pages to begin with. Although such features would not necessarily prevent all the potential negative effects of rendering attacker-controlled HTML in trusted domains, a well-executed solution could indeed avert most of the high-impact vulnerabilities.

There are two primary classes of solutions proposed:

  * **Comprehensive script security frameworks:** these approaches strive to offer the ability for site owner to approve or reject arbitrary actions executed by scripts with a great deal of flexibility. Examples include Microsoft [Mutation-Event Transforms](https://research.microsoft.com/users/yxie/papers/hotOS07.pdf) for browser-side checks (not clear if this is being pursued in any browser), or Google [Caja](http://code.google.com/p/google-caja/) and Microsoft [Web Sandbox](http://websandbox.livelabs.com/Default.aspx) for a solution that requires no browser-side tweaks.

  * **Selective script disable / enable functions:** these comparatively simpler solutions attempt to give web developers the tools to disable scripting in certain critical areas of the document, such as around user input or a block of intentionally non-sanitized HTML. One example such a proposal are Trevor Jim's [Browser-Enforced Embedded Policies](http://www.research.att.com/~trevor/papers/beep-tr.pdf), or the [toStaticHTML() API](http://msdn.microsoft.com/en-us/library/cc848922(VS.85).aspx) in Microsoft Internet Explorer 8.

## Secure JSON parsing ##

A traditional method for modern web applications to parse same-origin, serialized text Java<b></b>Script responses returned by servers in response to `XMLHttpRequest` calls is to pass them to `eval()` - which turns the string into a native object. Unfortunately, calls to `eval()` have a number of side effects - most notably, any code smuggled inside the string would execute in the recipient script security context, putting an additional burden on the server or the client to sanitize the code - and leading to many security bugs.

Several implementations of secure JSON parsers were proposed earlier, such as the (unsafe!) implementation suggested [RFC4627](http://www.ietf.org/rfc/rfc4627.txt), but they generally combine varying levels of insecurity with very significant performance penalties.

In response to this, several browsers rolled out [JSON.parse()](http://blog.mozilla.com/webdev/2009/02/12/native-json-in-firefox-31/), a native non-executing JSON parser that could be used as a drop-in replacement for `eval()`. The interface does not address the much greater risk of loading cross-domain JavaScript information via `<SCRIPT SRC=...>` or similar approaches, and the cross-browser support is at this point very limited, shipping in Chrome and Firefox 3.5.

## Origin headers ##

Adam Barth, Collin Jackson, and other researchers propose the introduction of reliable [Origin headers](http://people.mozilla.org/~bsterne/content-security-policy/origin-header-proposal.html) as an answer to the risk of cross-site request forgery - giving sites the ability to reliably and easily decide whether a particular request [permitted across domains](http://code.google.com/p/browsersec/wiki/Part2#Navigation_and_content_inclusion_across_domains) comes from a trusted party or not.

In theory, an existing `Referer` HTTP header could be used for the same purpose, but many users install tweaks to suppress it for privacy reasons; the header may also be dropped on certain page transitions, and was not generally designed to be reliable. `Origin` proposals attempt to limit the risk of the header being disabled by limiting the amount of data being sent across domains (host name, but no full URL; though arguably, it may still easily disclose undesirable information); and limiting the types of requests on which the header is provided (although this may undermine some of the security benefits associated with the solution).

## Mozilla content security policies ##

Brandon Sterne of Mozilla proposes [content security policies](http://people.mozilla.org/~bsterne/content-security-policy/index.html) ([specification](https://wiki.mozilla.org/Security/CSP/Spec)), a mechanism that would permit site owners to define a list of valid target domains for `<IMG>`, `<EMBED>`, `<APPLET>`, `<SCRIPT>`, `<IFRAME>`, and similar resources appearing in their content; and to disallow inline code from being used directly on a page.

The security benefit of these features is limited primarily to cross-site scripting prevention and webmaster policy enforcement; the proposal originally also incorporated the ability to define a list of valid origins for incoming requests as a method to mitigate cross-site request forgery attacks - but this part of the design got dropped in favor of more flexible and simpler `Origin` headers.

# Open browser engineering issues #

Other than the general design of HTTP, HTML, and related mechanisms discussed previously, a handful of browser engineering decisions tends to contribute to a disproportional number of day-to-day security issues. Understanding these properties is sometimes important for properly assessing the likelihood and maximum impact of security breaches, and hence determining the safety of user data. Some of the pivotal, open-ended issues include:

  * **Relatively unsafe core programming languages:** C++ is used for a majority of code in Internet Explorer, Firefox, Safari, Opera, and Chrome; C is used in certain high-performance or low-level areas, such as image manipulation libraries. The choice of C and C++ means that browsers are regularly plagued by memory management and integer overflow problems, despite considerable ongoing audit efforts.

  * **No security compartmentalization:** once control of the process is seized due to common implementation flaws, most browsers provide essentially unconstrained access to the user context they are running in. This means that browser bugs - historically, very common - easily lead to total system integrity loss.<p /><i>There are three exceptions: Chrome uses a <a href='http://dev.chromium.org/developers/design-documents/sandbox'>security sandbox</a> to contain the renderer (which includes Web<b></b>Kit and V8, hence constituting one of the largest and most complicated bodies of code in the project). Access to system functions and browser-managed data is heavily constrained, and individual tabs are generally run in separate processes, with some exceptions for new tabs spawned in response to document navigation. Android, on the other hand, runs the browser in a dedicated user account, and then restricts the ability for this context to execute any undesirable actions in the system. Likewise, Internet Explorer 7, when running on Windows Vista, is running with reduced privileges as a whole (<a href='http://msdn.microsoft.com/en-us/library/bb250462.aspx'>reference</a>).</i>.

  * **Web technologies are used in browser chrome:** Java<b></b>Script, HTML, and XML are all used to a varying degree to implement some browser internals and various diagnostic and error pages in most browsers. This choice contributes to an elevated risk of HTML injection flaws that permit web content to gain elevated chrome privileges, which - depending on the browser - may carry the permission to read or write files, access arbitrary sites on the Internet, or alter browser settings. The problem is particularly pronounced for Firefox, which implements much of its user interface in this manner.

  * **Inconsistent and overly complex security UIs:** a vast majority of browsers employ highly inconsistent UI elements and security messaging, including several styles of modal prompts, interstitials, icons, color codes, and messages that pop up either on the bottom or on the top of the document window. Usability studies consistently show that at least some of these features are easily mis-identified, misunderstood, or trivial to spoof (this is particularly the case for interstitials and notification bars that are not anchored in browser UI). Although a gradual improvement may be observed in certain aspects, further coordinated work in this area seems to be necessary; [timing attack issues](http://lcamtuf.blogspot.com/2010/08/on-designing-uis-for-non-robots.html) are a particularly interesting problem to deal with.

  * **Ad hoc plugin security models:** every plugin enforces its own variant of the same-origin policy and HTTP stack security checks, often with subtle variations from the browser implementation, and with a relatively poor track record. There is no compelling reason why these checks should not be managed centrally in browser code, other than the absence of well-documented APIs. This situation leads to frequent plugin security model issues, and make web application development more challenging than needs to be.

  * **Inconsistent and haphazard data storage practices:** browsers use a mix of random storage methods to keep temporary files, downloads, configuration data, and sensitive records such as passwords, browsing history, saved cookies, or cache entries. These methods include system registry, database container files, drop-off directories, text-based configs (CSV, INI, tab-delimited, XML), and proprietary binary files. The data may be stored in user home directories, system-wide temporary directories, or global program installation folders. Controlling the permissions on all these resources and manipulating them securely is relatively difficult, contributing to many problems, particularly in multi-user systems, or when multiple browsers are used by the same user.

  * **General susceptibility to denial-of-service attacks:** as discussed in the section on Java<b></b>Script execution restrictions, browsers are generally vulnerable to resource exhaustion or UI blocking attacks that may crash or render the browser - or even the underlying operating system - unresponsive or difficult to interact with. In some cases, this may be even abused to extort certain UI actions from less experienced users. Even with Java<b></b>Script out of the picture, tasks such as parsing XML or HTML documents, or rendering certain types of images or videos, are often sufficiently computationally expensive to facilitate such attacks.

  * **Incompatibility with untrusted networks:** browsers will persistently cache resources retrieved over untrusted networks, such as public wifi - and will grant such content full access to previously cached pages or credentials. This problem makes it virtually impossible to use public wireless hotspots safely.

_([Go back to the beginning](http://code.google.com/p/browsersec/wiki/Main))_