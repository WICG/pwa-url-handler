# PWAs as URL Handlers (Archived)

Authors: [Lu Huang](https://github.com/LuHuangMSFT) &lt;luhua@microsoft.com&gt;

Input from: [Mike Jackson](mailto:mjackson@microsoft.com), [Mandy Chen](mailto:mandy.chen@microsoft.com), [Howard Wolosky](mailto:howard.wolosky@microsoft.com), [Matt Giuca](mailto:mgiuca@google.com)

## Obsolete

This API proposal is obsolete in favor of [`scope_extensions`](https://github.com/WICG/manifest-incubations/blob/gh-pages/scope_extensions-explainer.md) and link capturing.
Context: https://docs.google.com/document/d/1w9qHqVJmZfO07kbiRMd9lDQMW15DeK5o-p-rZyL7twk

## Status of this Document

This document is intended as a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.

* This document status: **Active**
* Expected venue: [W3C Web Applications Working Group](https://www.w3.org/2019/webapps/)
* Current version: this document

## Introduction

Developers can create a more engaging experience if Progressive Web Apps (PWAs) are able to register as handlers for https uniform resource identifiers (URLs). This document proposes a scheme for a PWA to register as a URL handler and be launched when associated URLs are activated. PWA developers and end users are the customers of this solution.

Today, native applications on many operating systems (Windows, Android, iOS, MacOS) can be associated with http(s) URLs. They can request to be launched as URL handlers when associated URLs are activated. For example, a user could click on a link to a news story from an e-mail. An associated native app for viewing news stories would automatically be launched to handle the activation of the link. Web developers would be able to build more compelling PWA experiences with stronger user engagement if PWAs could request to be URL handlers through their web app manifests.

PWAs may have different levels of URL handling ability depending on the capabilities of the host OS. Whenever URL activations launch a conforming browser, that browser should have the ability to launch a registered PWA to render the requested content. In the best case, it would be possible for URL activations anywhere in the user's system to launch PWAs because they are registered as URL handlers with the OS. We are proposing changes below that could help accomplish this.

## Goals

1. Enable PWA developers to opt-in to URL handling features using the web app manifest.
2. Enable the default browser to support URL activation by launching a PWA.
3. Where possible, enable registration of PWAs as URL handlers with the operating system.
4. Provide a better user experience by allowing users to explicitly choose an installed PWA in app pickers and disambiguation dialogs.
5. Protect content owners by only allowing associated PWAs to act as URL handlers for their content.
6. Keep the user in control of choosing the best experience for them, whether that is in the browser, in a native app, or in a PWA.

## Non-Goals

* Custom URL protocol registration and handling. A separate explainer for that can be found [here](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/master/URLProtocolHandler/explainer.md).

* Launching PWAs due to in-browser link navigation. Capturing from navigation needs to address additional concerns around security and privacy. It is excluded here to reduce complexity. This explainer focuses on scenarios where the URL activation comes from outside the browser. Nothing written here should prevent URL handling from working on navigations in the future.

## Use Case

There is one use case that we wish to address in this explainer: URL activation in native applications.

The user clicks on a Spotify link in their native e-mail application, (eg., [https://open.spotify.com/album/7FA9xfqPrBaja1sEv15DU2](https://open.spotify.com/album/7FA9xfqPrBaja1sEv15DU2) in the Outlook app), which launches their default browser through the OS. Because the user already has the Spotify PWA installed and registered as a URL handler with their default browser, the URL activation launches the PWA instead of a new tab.

## Proposed Solution

1. Modify the web app manifest specification to include an `url_handlers` member.
    * Allows PWAs to handle URLs from multiple different origins.
    * Allows PWA developers to opt-in to URL handling in the same way across different platforms.

2. Specifying a `web-app-origin-association` file format for validating out-of-scope URL associations.
    * Provides a web standard option for association with web apps.
    * Protects content owners by letting them define which PWAs are allowed to associate with their sites.
    * Gives content owners control over which URLs are allowed to be handled by each associated PWA.

3. If a browser is launched to handle a URL activation, it should look for matching, installed PWAs to handle that URL.

4. If multiple PWAs match a given URL, browsers should display a disambiguation dialog to allow users to choose one or continue in a browser window.

5. On OSes with adequate support, conforming browsers should register PWAs as URL handlers at the OS level instead.
    * Allows any native app, including browsers, to launch participating PWAs using OS URL launch.

**Changes 1-4** allow PWAs to handle URL activations at the browser level without needing to integrate with the OS. As long as a URL activation (clicking on a link) launches a conforming browser, the browser has the ability to launch the matching PWA. A PWA will not be launched through URL activations from the OS if it is not known to the default browser.

To allow PWAs to handle URLs that are outside of their own scope, it is necessary to introduce a mechanism for the owner of those URLs to opt-in to URL handling by PWAs. **Change 2** introduces the concept of a `web-app-origin-association` file that will serve this purpose. This file is similar to the [Apple App Site Association File](https://developer.apple.com/documentation/safariservices/supporting_associated_domains_in_your_app#3001215), the [`assetlinks.json`](https://developer.android.com/training/app-links/verify-site-associations) file in Android, and the [`windows-app-web-link`](https://docs.microsoft.com/en-us/windows/uwp/launch-resume/web-to-app-linking#associate-your-app-and-website-with-a-json-file) file in Windows. What differs is that the `web-app-origin-association` file does not reference PWAs using a platform-specific app id but by their web app manifest URL.

**Change 5** will also allow PWAs to handle URL activations at the OS level on supporting platforms. If the PWA is able to register as a URL handler with the OS, it could be launched whenever a URL activation is handled by the OS, regardless of the default browser setting. Most native applications rely on the OS for URL activation. In terms of user experience, the OS could now prompt the user to choose between the PWA and the default browser using the system app disambiguation dialog. The user is able to make an explicit choice to select the PWA and configure their default setting from there. (Implementation note: because most OSes do not know of or treat PWAs as first-class applications, it may not be possible to register them directly with the OS as URL handlers. Supporting changes will need to be made in PWA implementation and/or in the OS to enable this. A notable exception is Chrome's implementation of PWAs on Android using `WebAPK` . Because `WebAPK` s are recognized by the Android OS, Chrome PWAs are able to fully integrate with OS features like the app picker.)

### Manifest Changes

| Field       | Required / Optional | Description                                   | Type     | Default |
| :---------- | :------------------ | :-------------------------------------------- | :------- | :------ |
| `url_handlers` | Optional            | Origins of URLs that the app wishes to handle | object[] | `[]`    |

We propose adding a new _optional_ member `url_handlers` to the manifest object of `object[]` type. Each object in `url_handlers` contains a `origin` string, which is a pattern for matching origins. These patterns are allowed to have a wildcard (*) prefix in order to include multiple sub-domains. URLs that match these origins could be handled by this web app.

Each `url_handlers` object is a request from the PWA to handle URLs from a specific origin or origins. The browser should validate with each origin that the app is recognized and if so retrieve the patterns for allowed URLs. On an OS that allows for deeper integration, the browser should also register URL handling requests with the OS and keep them in sync with the app.

Example web app manifest at `https://contoso.com/manifest.json` :

``` json
{
    "name": "Contoso Business App",
    "display": "standalone",
    "icons": [
        {
            "src": "images/icons-144.png",
            "type": "image/png",
            "sizes": "144x144"
        }
    ],
    "capture_links": "existing_client_event",
    "url_handlers" : [
        {
            "origin": "https://contoso.com"
        },
        {
            "origin": "https://conto.so"
        },
        {
            "origin": "https://*.contoso.com"
        }
    ]
}
```

Example web app manifest at `https://partnerapp.com/manifest.json`

``` json
{
    "name": "Contoso Business App",
    "display": "standalone",
    "icons": [
        {
            "src": "images/icons-144.png",
            "type": "image/png",
            "sizes": "144x144"
        }
    ],
    "capture_links": "existing_client_event",
    "url_handlers": [
        {
            "origin": "https://contoso.com"
        },
        {
            "origin": "https://conto.so"
        },
        {
            "origin": "https://*.contoso.com"
        }
    ]
}
```

(`capture_links` from the [Declarative Link Capturing](https://github.com/WICG/sw-launch/blob/master/declarative_link_capturing.md) proposal added to examples for comparison.)

A PWA matches a URL for URL handling if the URL matches one of the origin strings in `url_handlers` and the browser is able to validate that the origin agrees to let this app handle such a URL.

`url_handlers` can contain an origin that encompasses requesting PWA's scope and also other unrelated origins. Not restricting URLs to the same scope or domain as the requesting PWA allows the developer to use different domain names for the same content but handle them with the same PWA. See [this section](#web-app-to-origin-association) for how `url_handlers` requests can be validated with origins. Navigation redirection is not a good alternative with respect to offline scenarios.

#### Wildcard Matching

The wildcard character `*` can be used to match one or more characters.

A wildcard prefix can be used in `url_handlers` origin strings to match for different subdomains. The prefix must be `*.` for this usage. The scheme is still assumed to be https when using a wildcard prefix.

For eg. `*.contoso.com` matches `tenant.contoso.com` and `www.tenant.contoso.com` but not `contoso.com` . There may be other ways of specifying a group of related origins such as [First Party Sets](https://github.com/krgovind/first-party-sets). This feature would not be necessary if there was a way to specify a multi-origin app scope with a similar matching pattern.

### web app to origin association

Browsers must validate a handshake between a PWA and an origin to successfully register URL handlers. Origins can declare associations with specific web apps to complete this handshake. Web apps can be identified by their manifest URL currently before a [unique identifier](https://github.com/w3c/manifest/issues/586) is standardized. An origin should be allowed to specify URL patterns to fine-tune URL paths for URL handling.

We propose a platform-independent association json file format that origins could use for the handshake.

#### web-app-origin-association file

Example 1: web-app-origin-association file at both `www.contoso.com/.well-known/web-app-origin-association` and `https://conto.so/.well-known/web-app-origin-association` :

``` json
{
    "web_apps": [
        {
            "manifest": "https://contoso.com/manifest.json",
            "details": {
                "paths": [
                    "/*"
                ],
                "exclude_paths": [
                    "/blog",
                    "/about"
                ]
            }
        },
        {
            "manifest": "https://partnerapp.com/manifest.json",
            "details": {
                "paths": [
                    "/public/data/*"
                ]
            }
        }
    ]
}
```

Example 2: web-app-origin-association file at `https://tenant.contoso.com/.well-known/web-app-origin-association` :

``` json
{
    "web_apps": [
        {
            "manifest": "https://contoso.com/manifest.json",
            "details": {
                "paths": [
                    "/*"
                ],
                "exclude_paths": [
                    "/only/for/partnerapp/*"
                ]
            }
        },
        {
            "manifest": "https://partnerapp.com/manifest.json",
            "details": {
                "paths": [
                    "/*"
                ]
            }
        }
    ]
}
```

Example 1 shows that the origins `https://contoso.com` and `https://conto.so` can both use the same file to associate with the PWA that has a web app manifest at `www.contoso.com/manifest.json` . In practice, the file at `contoso.so` could be a redirect.

Example 2 shows that the origin `https://tenant.contoso.com` allows the PWAs with web app manifests at `https://contoso.com/manifest.json` and `https://partnerapp.com/manifest.json` to handle a subset of its URLs.

This file must contain valid JSON. The top-level structure is an object, with a member named `web_apps`. `web_apps` is an array of objects and each object represents an entry for a unique web app. Each object contains:

| Field         | Required / Optional | Description                                              | Type   | Default |
| :------------ | :------------------ | :------------------------------------------------------- | :----- | :------ |
| `manifest`    | Required            | URL string of the web app manifest of the associated PWA | string | N/A     |
| `details`     | Optional            | Contains arrays of URL patterns                          | object | N/A     |

Each `details` object contains:
| Field           | Required / Optional | Description                      | Type     | Default |
| :-------------- | :------------------ | :------------------------------- | :------- | :------ |
| `paths`         | Optional            | Array of allowed path strings    | string[] | `[]`    |
| `exclude_paths` | Optional            | Array of disallowed path strings | string[] | `[]`    |

#### File Location

To make use of the web-app-origin-association file, we suggest that association files be placed in a `.well-known` directory within the root path of the origin. In order to match an origin with a `*.` prefix, we suggest that the corresponding association file be placed relative to the root path of the domain. Eg. an origin `*.contoso.com` could have a `web-app-origin-association` file at `contoso.com/.well-known/web-app-origin-association`.

Alternatively, we suggest browsers locate it using a `<link rel="web-app-origin-association" href="/web-app-origin-association">` element in the header section of the main document at the origin's root path.

#### Failure to Associate

If necessary association validation cannot be completed successfully during PWA installation for any reason, the browser must not register the app as an active URL handler for the affected URLs. The browser should try to complete installation with as many valid URL handling registrations as possible.

#### Periodic Revalidation

An origin could modify its associations with PWAs at any time. Conforming browsers must regularly attempt to revalidate the associations of installed web apps. If a URL handler registration fails to revalidate because the association data has changed or is no longer available, the browser must disable or remove registrations that are no longer valid.

#### Shortened URLs

Web applications often provide users with shortened URLs for convenience. If developers would like register these URLs for URL handling, they need access to the short URL origin to place a validation file. This may not be possible with third party URL shortening services that the developer does not control.

### Browser Changes

To support basic, browser-level registration of URL handlers, browsers should make the following changes:

1. Validate and register the `url_handlers` data from PWA manifests during PWA installation and periodically revalidate installed apps.

2. Perform adequate validation to address security and privacy concerns. They may do so using a web-app-origin-association file or another method of their choosing.

3. When starting with a URL parameter, determine if there are any matching PWA URL handlers.

   * If there is a match, launch the PWA and load the URL in a new standalone window instead of the browser window.

   * If the launch URL is not within the app scope, browsers can delegate to a document event handler. If there is an existing app window with a loaded document, browsers can try to find a suitable event handler there. If not found, the browser could fall back to opening a new app window, waiting for a document to load, then looking for a suitable event handler.  

   * If there is a manifest member that specifies a behavior for launching web apps from link activations (for eg. `capture_links` in [Declarative Link Capture](https://github.com/WICG/sw-launch/blob/master/declarative_link_capturing.md)), app launch should try to follow that launch behavior specification.

   * If there is more than one match, display the choices and collect the user's input with a disambiguation dialog.

4. Keep URL handling registrations in sync with the PWA's lifecycle (i.e. during manifest update, uninstall, etc.)

#### User preferences

If URL handlers are registered and launched at the browser level, the browser should allow the user to enable/disable URL handlers, set default launch behavior, etc through browser settings. If URL handlers are registered and launched at the OS level, the browser should direct the user to the OS settings.

#### Handling multiple registrations

The association file is able to contain association objects for multiple PWA handlers. Multiple PWAs are able to request to handle the same URL. In cases where there are multiple registered handlers available, the browser or OS should present options to the user and allow the user to decide which handler to use.

### Operating System Changes

To provide OS level registration of URL handlers, browsers' PWA implementations need to integrate more deeply with the OS application platform. This is to allow the OS to recognize PWAs as apps that can be launched and can therefore serve as URL handlers. There may be different approaches to accomplishing this with different OS platforms. OS changes may be necessary to enable this integration. In OS or browser versions where this integration cannot be implemented, the behavior should default to URL handling by the browser.

## Security Considerations

URL handlers, if improperly implemented, can be used to hijack traffic for websites. This is why the app association mechanism is an important part of the proposed scheme. Associations must be validated when PWAs are installed, and again periodically to evaluate any new changes in the association file and web app manifest.

If a browser registers apps with the OS as URL handlers, the OS must trust that browser to validate the PWA-to-site association or implement suitable validation itself. If the OS delegates the validation to browsers, it must be clear which browsers are compliant with the validation requirements.

If an associated site is overtaken by a malicious actor, it is possible for users to be exposed to malicious content through the PWA handling those URLs. To mitigate this risk, the browser may want to suppress the PWA launch or get user confirmation using a security mechanism which detects risky URLs.

Conforming browsers may want to limit the maximum processed entries of `url_handlers` to N and the numbers of allowed and disallowed paths (in `web-app-origin-association` or equivalent) each to M. This will limit the amount of work the manifest parser does and further limit the risk of URL hijacking.

URL handler registrations should only be performed for installed PWAs as users expect installed applications to be more deeply integrated with the OS. Furthermore, conforming browsers should not activate PWAs as URL handlers for any URL without an explicit user confirmation.

## Privacy Considerations

Third-party websites would not be able to leverage this API to detect installed web apps by observing navigations to first-party URL links if there is no difference in in-browser navigation behavior.

Although not an immediate privacy concern, fingerprinting risks should be thoroughly examined when designing web APIs that activate URL handlers instead of performing regular in-browser navigation. Further mitigation such as obfuscation of launch timing may be necessary.

Native applications can already use OS APIs to enumerate installed applications on the user's system. For example, native applications in Windows can use the [FindAppUriHandlersAsync](https://docs.microsoft.com/en-us/uwp/api/windows.system.launcher.findappurihandlersasync) API to enumerate URL handlers. If PWAs register as OS level URL handlers in Windows, their presence would be visible to other applications.

## Relation to other proposals

### [Declarative Link Capturing](https://github.com/WICG/sw-launch/blob/master/declarative_link_capturing.md) (DLC)

[Declarative link capturing](https://github.com/WICG/sw-launch/blob/master/declarative_link_capturing.md) would allow a developer to opt-in all app scope URLs to link capturing behavior with simple changes to their web app manifest. If Declarative Link Capturing and URL Handling features are both available, browsers may reduce overlap in functionality by prioritizing DLC behavior over URL handling behavior: if a URL matches the app scope of an installed app with DLC enabled, there is no need to further match against the URL handling registrations of other apps.

DLC aims to provide a choice of different app launch behaviors. To avoid overlap in the proposals, URL Handling should use a default, non-configurable launch behavior and support the standardization of a manifest member like `capture_links`.

### [Service Worker Scope Pattern Matching](https://github.com/wanderview/service-worker-scope-pattern-matching/blob/master/explainer.md) (SWSPM)

This proposal uses a wildcard and pattern matching syntax that is compatible with the manifest syntax designed for scope pattern matching. If there are multiple manifest members that use URL pattern matching, URL Handling should continue to use a compatible syntax for developers' ease of use.

## OS Specific Implementation Notes

### Windows

* Starting from the Windows 10 Anniversary update, the ["Apps for Websites"](https://docs.microsoft.com/en-us/windows/uwp/launch-resume/web-to-app-linking) capability allows the registration of apps to handle URLs from associated websites.

### Android

* Chrome installs a PWA on Android by generating and installing a [WebAPK](https://developers.google.com/web/fundamentals/integration/webapks).
* When a Chrome PWA is installed on Android, it can [register a set of intent filters](https://developers.google.com/web/fundamentals/integration/webapks?hl=ro#android_intent_filters) for all URLs within the scope of the app.
* This means that Chrome PWAs already handle associated URLs on Android at the OS level using intent filters. This is also known as Deep Linking.
* Additionally, Android apps are also able to register to be URL handlers by using Android App Links.
* App Links require server side verification of the relationship to the app while Deep Links do not.
* Deep Linking will show an app picker but App Linking will take the user directly to the app.

Other browsers (e.g., Edge) on Android are able to add PWAs to the home screen but are not able to register intent filters. They likely need to also be able to use a WebAPK implementation or similar to enable URL handling.

### iOS, MacOS

* iOS allows the association of apps to websites using [Universal Links](https://developer.apple.com/ios/universal-links/).
* Newer versions of MacOS also support Universal Links.
* Safari implements a subset of PWA features.
* Safari is currently the only browser able to install a PWA as an iOS app.
* Universal links/link capturing is not available to iOS PWAs.

## Open Questions

* In order to validate an origin with sub-domain wildcard prefix ( `*.contoso.com` ), is it reasonable to locate the validation file relative to the domain? This breaks from the assumption that the domain and its sub-domains are separate origins and may have further security implications.
