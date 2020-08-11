# Answers to [Security and Privacy Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/)

This has been created by copy-and-pasting from https://w3ctag.github.io/security-questionnaire/, as requested in the TAG review instructions.

1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

    No information is exposed to Web sites or other parties as part of this feature.

2. Is this specification exposing the minimum amount of information necessary to power the feature?

    Yes.

3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?

    No personal information is collected, used, or exposed via this feature.

4. How does this specification deal with sensitive information?

    No sensitive information is collected, used, or exposed via this feature.

5. Does this specification introduce new state for an origin that persists across browsing sessions?

    No.

6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

    None.

7. Does this specification allow an origin access to sensors on a user’s device?

    No.

8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

    None.

9. Does this specification enable new script execution/loading mechanisms?

    No.

10. Does this specification allow an origin to access other devices?

    No.

11. Does this specification allow an origin some measure of control over a user agent’s native UI?

    Yes. It allows an origin to request app content from an installed web app from that origin be displayed in an app window. The user still has complete control over the choice of UI. The change of UI has no effect on security or privacy controls. Analysis: app-like UI for web apps is an existing, validated feature in many user agents.

12. What temporary identifiers might this specification create or expose to the web?

    No identifiers are created or exposed to the web.

13. How does this specification distinguish between behavior in first-party and third-party contexts?

    This feature cannot be used by third-party resources. The only access is through the first-party controlled web app manifest.

14. How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?

    This feature has no effect in Private Browsing or "incognito" modes because PWA features cannot be used in those modes.

15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?

    Yes. See these links:
    * https://github.com/WICG/pwa-url-handler/blob/master/explainer.md#security-considerations
    * https://github.com/WICG/pwa-url-handler/blob/master/explainer.md#privacy-considerations

16. Does this specification allow downgrading default security characteristics?

    No.
