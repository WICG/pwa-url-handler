# Web App Link Handling Manifest Options

Authors: [Diego Gonzalez](https://github.com/diekus), [Lu Huang](https://github.com/luhuangmsft)

## Status of this Document
This document is a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.
* This document status: **Withdrawn**
* Expected venue: [W3C Web Incubator Community Group](https://wicg.io/)
* **Current version: this document**
    
## Introduction

When clicking on a link, the default behavior is that the browser will open and navigate to the specified URL. However, if a compatible application is installed, a user might prefer to have that application launch and "open" said link. This is the case for native apps that range from media consumption to productivity, where if installed, will open when clicking on a link with related content. This is generally the preferred way to interact with the referenced content. 

To achieve this, an installed application might want to register itself to handle links and create this seamless flow. From a UX perspective, it is desirable to give users choice on how they prefer the link to be opened, and it is important since it creates the cohesive and integrated experience mentioned previously.

The `handle_links` manifest entry aims to allow registration of installed web applications as link handlers, to grant them the possibility of opening links like their native counterparts.

## Goals

* Allow web apps to handle links that are within their [scope](https://www.w3.org/TR/appmanifest/#understanding-scope).

## Non-Goals

* Customize app launch behavior (see [launch_handler](https://github.com/WICG/sw-launch/blob/main/launch_handler.md) for this).
* Extending the scope of the links that can be handled by the installed web app (see [scope extensions](https://github.com/WICG/manifest-incubations/blob/gh-pages/scope_extensions-explainer.md) for this).

## Use Cases

The generic use case is as follows: A user clicks on a link that refers to content within scope a web app they have installed on their machine. The link opens in the web app instead of the browser. This is behavior that currently occurs on commerce, streaming and productivity apps. 

For example, if a user gets a link to edit a presentation they were shared, that presentation can open in the installed web app where they can proceed to work on it. 

In a similar vein, a user can click on a link of a music streaming service (www.streamify.com/player?trackid=demotrack) to open the referenced track in the installed PWA. 

## Proposed Solution

### `handle_links` manifest member

We propose the addition of the `handle_links` member to the web app manifest that specifies the default link handling for the installed web app. The shape of this member is as follows:

```
"handle_links": "auto" | "preferred" | "not-preferred"
```

* `preferred`: the user agent should open in-scope links within the installed application.
* `not-preferred`: the user agent should not open links within the installed application.
* `auto`: The user agent should select the appropriate behavior for the platform.

These options indicate an app's link handling preference. They should be seen as suggestions from an app to the user agent. 
* `auto`: Default value if `handle_links` is not found in the manifest. The user agent may choose between `preferred` and `not-preferred`.
* `preferred`: The user agent should handle links using matching app clients and may promote link handling behavior. 
* `not-preferred`: The user agent should not handle links using matching app clients and may not promote link handling behavior.

## Privacy and Security Considerations

### Privacy

No considerable privacy concerns are expected, but we welcome community feedback.

### Security

The developer options described in this explainer works on app manifest scope and depends on the scope boundary for security. No additional security concerns are expected, but we welcome community feedback.

## Related Proposals

### URL Handlers
The `handle_links` proposal is intended to be a part replacement to the [PWA as URL Handlers proposal](https://github.com/WICG/pwa-url-handler/blob/main/explainer.md). The functionality of `url_handlers` is now divided between `scope_extensions` and `handle_links`. 

### `scope_extensions`

The [`scope_extensions`](https://github.com/WICG/manifest-incubations/blob/gh-pages/scope_extensions-explainer.md) proposal allows web apps to extend their scope to other origins. When used in conjunction with `handle_links` the web app can open links from the new extended scope.

### `launch_handler`
 
The [`launch_handler`](https://github.com/WICG/sw-launch/blob/main/launch_handler.md) proposal enables web apps to customize their launch behavior across all types of app launch triggers. This can work with `handle_links` by specifying how/where to open/route the links. 
