# Origin Trials Guide for Web Developers

Origin trials allow developers to try out new features and give feedback on usability, practicality, and effectiveness to the web standards community. Your feedback is valuable input into the final decision about the feature design, or even whether we want to proceed with standardizing and enabling the feature by default. When a feature is available as an origin trial, you are able to register to have it enabled for all users on your origin for a fixed period of time. Note that when the trial finishes we will contact you with a request to provide this feedback.

Once your origin has opted into a trial of an experimental feature you can then build demos and prototypes that your friends and beta testing users can try for the duration of the trial without them needing to flip special flags in Chrome.

You can also run limited production experiments to evaluate the effectiveness of a feature at scale. You must be careful to limit the amount of traffic using the feature, as the trial will automatically disable if is used in more than 0.5% of Chrome page loads (across all sites). This is part of the protections in place to prevent the feature from becoming a defacto standard before the experimentation and standards work have completed.

## How do I enable an experimental feature on my origin?

You can opt any page on your origin into the trial of an experimental feature by [requesting a token for your origin](https://developers.chrome.com/origintrials/). After signing up for a trial, we will generate a token for your origin.

There are two ways to provide this token on any pages in your origin:

- Add an `origin-trial` \<meta\> tag to the head of any page. For example this may look something like:

  ```
  <meta http-equiv="origin-trial" content="**insert your token as provided in the developer console**">
  ```

- If you can configure your server, you can also provide the token on pages using an `Origin-Trial` HTTP header. The resulting response header should look something like:

  ```
  Origin-Trial: **token as provided in the developer console**
  ```

**NOTE:** 

- You can provide multiple tokens for a given page. See [Can I provide multiple tokens on a page?](developer-guide.md#15-can-i-provide-multiple-tokens-on-a-page).
- You can provide tokens programmatically, via script. See [Can I provide tokens by running script?](developer-guide.md#16-can-i-provide-tokens-by-running-script).

If you have trouble configuring pages with your token, or need other help, please contact us at origin-trials-support@google.com.

## How can I experiment with the new feature locally?

Each feature that is available as an origin trial can alternatively be enabled on individual machines by flipping the corresponding flag in about:flags. The correct flag depends on the feature, and should be mentioned in the blog post about that specific feature.

You can get started experimenting with the new feature on `localhost` either by flipping the flag locally or requesting an origin trial token for `localhost`.

For automated tests, you can enable the feature flag using a Chrome command-line parameter: ```--enable-blink-features=xxxxx```.

## What is the thinking behind origin trials?
An exploration of the motivations and reasoning behind origin trials is provided in [the explainer](explainer.md). The TL;DR is that we strongly value the feedback of real web developers (that means you!) during the process of designing and standardizing new features. We believe origin trials provide a good way of encouraging that feedback, while being extremely careful that the experiments aren’t used by sites in production-critical roles or as if they’re finalized features.

## What experimental features are currently available?
The [developer console](https://developers.chrome.com/origintrials/#/trials/active) lists all of the currently available features.

## FAQ

### 1. How can I find out about new experiments when they become available?

- When you register for an origin trial token you will be automatically added to a mailing list. We'll use this list to send high level updates about the origin trials system, including announcing new features.
- Additionally, we will be posting updates to [developers.google.com/web/updates/](http://developers.google.com/web/updates/) about each new feature that becomes available as an origin trial.

### 2. Will all of these experiments ship eventually?

These are only experiments and there is a good chance that some of them will never ship as standardized features on the web. These experimental features are essentially very similar to Chrome flags: an exciting glimpse into one possible future that you can play around with today, and provide feedback for.

### 3. What happens if a large site such as a Google service starts depending on an experimental feature?

Origin trials have a built-in safeguard that automatically disables an experimental feature globally if its total usage exceeds 0.5% of all Chrome page loads. This is to keep usage limited to developers experimenting and below Chrome’s threshold whereby features used on less than 0.5% of all page loads (as measured by [Chrome Status](https://www.chromestatus.com/metrics/feature/popularity)) may be deprecated.

For very popular sites it is important to only experiment with a small portion of your traffic when participating in a trial.

### 4. Isn’t this just vendor prefixing all over again?

This topic has been explored in depth in Alex Russell’s Medium post [Doing Science on the Web](https://medium.com/@slightlylate/doing-science-on-the-web-af26d9be2faa#.94pf1lwmp). A couple of key differences include:

- These features automatically stop working before they become too broadly adopted.
- Developers cannot simply copy-paste sample code using an experimental feature, as they must provide a unique trial token obtained via the experimental feature registration signup process (and accept that the feature is going to shortly stop working).

### 5. Does this change impact how we think about security or privacy on the web?

No, these experimental features have all been held to the same high privacy and security standards as any Chrome platform feature.

### 6. Is there any restriction on which websites can sign up to use experimental features?

Origin trials are available to any website served over HTTPS. Note that there is no policy against specific large sites opting into origin trials, but the system is designed to prevent large populations of the web depending on experimental features. To achieve that, origin trials have a built-in safeguard that automatically limits it globally if its usage exceeds 0.5% of all Chrome page loads. This means that experimental features aren’t suitable for full production use on large production sites such as the Google home page, though experiments with limited usage on large sites is encouraged.

### 7. Is there any review process for signing up a website to access an experimental feature?

No. We do not review domain content before generating a token.

### 8. Is there a way to only enable an origin trial for some of my users or only some pages on my site?

Yes, origin trials are enabled on a per-page basis.

### 9. Will these experiments work in Opera or other web browsers?

Not today, but if this model proves to work well then it’s possible that other web browsers may build their own origin trials system. The experimental feature implementations likely won’t be compatible between browsers due to the nature of them being experimental.

### 10. Can I request a token for an origin that I don't own?

Yes, you can technically request a token for an origin that you don't own. However, generating the token won't cause the feature to be enabled on that origin - unless it is served in the pages on that origin (either in the \<head\> or as an HTTP header). Also note that these features have held up to the same high security and privacy standards as any other feature in Chrome.

### 11. Why do tokens expire before the trial ends?

- The trial may have been extended after your last registration. Each token contains the
    expiration date and cannot be updated, so you must renew to generate a new token in this
    case. We'll also send you an email to invite you to renew and continue participating in the
    trial.
- Prior to February 3, 2021, tokens were issued to be short-lived, lasting 6 weeks until they expire. 
    Please see [How do I renew a token that is about to expire/has expired?](#renew) for how to renew your tokens 
    to continue participating in the trial.

<a name="renew"></a>
### 12. How do I renew a token that is about to expire/has expired? 

- You should receive a reminder email to renew the token before it expires. That email includes a 
    link to the registration page in the [developer console](https://developers.chrome.com/origintrials/#/trials/active).
    You can also go directly to the page in the console, by finding the trial in the list on
    [My Registrations](https://developers.chrome.com/origintrials/#/trials/my).
- On the registration page, you may need to provide feedback before you can renew. Use the 
    Feedback button to complete a survey to provide feedback.
- On the registration page, the Renew button will be available when feedback has been provided. 
    Use the Renew button to generate a new token.
- Feedback is required to renew tokens. If you have multiple origins registered for a trial,
    we'll only ask you for once per period, rather than for every origin.

### 13. I have multiple testing/staging domains, or subdomains that are programmatically generated. Do I need to request a token for every subdomain?

- No, we can issue a single token that will match multiple subdomains. These tokens will behave similarly to wildcard matching (like specifying "\*.\<some domain\>"). For example, you can request a token for "example.com", and it will enable the feature on all origins whose suffix matches "example.com", including:
  - a.example.com
  - b.example.com
  - a.b.example.com
  - example.com
- To ensure that an experimental feature is not enabled too broadly, there are some additional checks on requests for subdomain-matching tokens. Specifically, subdomain-matching tokens will not be issued for origins found in the [Public Suffix List](https://publicsuffix.org/). This restriction does not apply to deprecation trials, where subdomain-matching tokens are issued for all origins, regardless if they are found in the Public Suffix List.
- Subdomains do not apply to IP addresses. Tokens issued for IP addresses will only allow exact matching on origin, as before.
- You can request a subdomain token by filling out the appropriate field on the trial signup form.

### 14. Are there different types of trials?

Yes, we've started using the origin trial infrastructure and console to allow developers to temporarily control Chrome's behavior in different ways. For example, to register for a [temporary extension to Web Components V0 deprecation](https://developers.chrome.com/origintrials/#/view_trial/2431943798780067841). We're also planning to have trials to allow developers to opt-in or opt-out of some Chrome interventions (especially those based on heuristics). Some aspects of these trials may be different, such as the expiry period for tokens. However, all types of trials will be limited in duration - this is not meant as a new mechanism for permanent configuration.

### 15. Can I provide multiple tokens on a page?

Yes, you can provide multiple tokens for a given page. Only one valid token is required to enable a trial, any other invalid tokens or non-matching tokens are ignored. You may want to provide multiple tokens if the same page is served to different origins (e.g. example.com, example.ca, etc.). You can add multiple tokens in different ways:
  - Add multiple \<meta\> tags to the page, where each tag contains a single token.
  - Include multiple `Origin-Trial` response headers, where each header contains a single token.
  - Include a single `Origin-Trial` response header, with comma-separated tokens.

### 16. Can I provide tokens by running script?

Yes, you can provide tokens programmatically, from a script running on a given page. A token 
can be provided by creating and injecting a \<meta\> tag into the head of the page, in the same
format as would be provided in HTML markup. For example, using a function like:

```
function addTrialToken(tokenContents) {
  const tokenElement = document.createElement('meta');
  tokenElement.httpEquiv = 'origin-trial';
  tokenElement.content = tokenContents;
  document.head.appendChild(tokenElement);
}
```

Limitations for tokens injected via script include:

- The token must match the origin of the page containing the script (i.e. the *first-party*
origin). For external scripts (e.g. included as `<script src="some origin">`), the token
validation ignores the origin of the script. This does not apply to
[*third-party* tokens](developer-guide.md#18-how-can-i-enable-an-experimental-feature-as-embedded-content-on-different-domains).
- The token can usually be injected later in the page lifecycle (e.g. after loading), but must be
injected before any attempt to use the experimental feature, including feature detection. For
experimental features that are exposed to JavaScript, once the token is injected, you'll be
able to test for API presence as expected.
- There may be experimental features that are not compatible with token injection via script.
For example, features that affect the initial loading of a page or require configuration via HTTP
  response headers.

<a name="iframes"></a>
### 17. Are experimental features enabled in iframes?

Not by default — iframes in a page **do not** inherit the experimental features that are
enabled from their containing page. When the page provides a token to enable an experimental
feature, that only applies to the top-level document. Each iframe is considered a separate
document, and must independently provide a valid token to enable an experimental feature. As
well, each iframe must provide a token that matches its own origin, not the origin of the
containing page. Finally, this applies to all nested iframes.

The behavior is different when the origin trial enables an experimental feature that is
also controlled by
[Permission Policy](https://w3c.github.io/webappsec-permissions-policy/#features).
Some policy-controlled features must be delegated to an iframe from its parent (see
[How is a Policy Specified?](https://github.com/w3c/webappsec-permissions-policy/blob/main/permissions-policy-explainer.md#how-is-a-policy-specified)
in the explainer). For delegation to work, both the parent and the iframe must provide a
valid token to enable the experimental feature.

### 18. How can I enable an experimental feature as embedded content on different domains?

- You can enable an experimental feature as embedded content, as long as you have third-party
script running in the containing page.

- To enable as embedded content, there are two options to request the appropriate tokens(s):
  1. Register for a token for each target origin.
      - Available for all trials.
      - May be feasible if you have a known, short list of the domains that embed your content.
      - Requires you to provide the correct token from your script, which matches the containing
      page.
  2. Register once for a *third-party* token for your own origin.
      - Currently, available on a limited basis for some trials.
      - Requests are subject to verification to prevent misuse.
      - Allows you to use a single token on any embedding page, regardless of its origin.
      - Requires you to provide the token from a script served from your origin. Inline scripts
      are not supported.

- Your third-party script must provide the appropriate token as described in
[Can I provide tokens by running script?](developer-guide.md#16-can-i-provide-tokens-by-running-script).

<a name="usage-limits"></a>
### 19. Are there any usage limits on experimental features? 

Origin trials have a built-in safeguard that automatically disables an experimental feature if
its usage exceeds a small percentage of all Chrome page loads. This is designed to prevent
individual web sites, and large populations of the web, from depending on experimental features.

Deprecation trials are not subject to such limits as they are not introducing a new feature and
therefore there is no risk of depending on experimental features. Similarly, trials where
existing functionality is removed are not constrained as the same risk doesn not apply.

The standard usage limit for a trial is 0.5% of all Chrome page loads, based on the same
statistics as in [Chrome Status](https://www.chromestatus.com/metrics/feature/popularity). The
usage is measured in a way to allow for temporary spikes of higher usage. In rare exceptions, a
trial may be approved for a higher usage limit, only when necessary to allow for more effective
experimentation (e.g. statistical significance requires a larger sample size of page loads).

This means that large production sites can try out experimental features, *but* they must take
care to limit usage to a suitable portion of their traffic.

<a name="usage-restrictions"></a>
### 20. What are the options for usage restrictions on tokens? 

By default, tokens are issued without any usage restriction. This means that when a token is
valid for a page, the experimental feature will always be enabled on that page (assuming it
hasn't been disabled for exceeding [usage limits](#usage-limits)).

For some trials, there will be options for different usage restrictions when registering for a
token. If available, the choices are:

- Standard Limit: This is the default behavior, where the global usage limit is the only
    restriction (described above).
- User Subset: A small percentage of Chrome users will always be excluded from the trial, even
    when a valid token is provided. The exclusion percentage varies for each trial, but is
    typically less than 5%.

<a name="valid-until"></a>
### 21. What does the _Valid Until_ date mean for my tokens?  

- Tokens are guaranteed to be accepted by Chrome up to the _Valid Until_ date. This date is calculated
    based on your feedback, initially set to 6 weeks after the token is created.
- Feedback is required every 6 weeks to extend the _Valid Until_ date in order to keep
    participating in the trial. 
- Before the _Valid Until_ date arrives, we'll send an email to invite you to provide feedback
    and continue participating in the trial.
- Tokens without any feedback in the past 6 weeks will automatically be disabled by a remote
    update process after a grace period.
- We will send a last warning email before disabling the token. This will be your last chance
    to provide feedback and keep the token active. After that your token will no longer be accepted by
    Chrome, and will not enable the experimental feature on any page.
- You can still provide feedback after the token is disabled. However, the token will not be
    re-enabled immediately, even though the _Valid Until_ date is updated. The process to disable
    and re-enable tokens runs periodically, and it takes some time for the remote updates to
    reach the majority of Chrome clients.
- We issue tokens this way for a number of reasons, most importantly:
  - To prevent experimental features from becoming "burned in" to the web platform. With
      shorter-lived tokens, we can ensure that no site can use a feature for more than a month or
      two, without checking in with us.
  - To provide an opportunity to collect feedback on features from web developers. Feedback is
      required to extend the lifespan of the token.
  - To ensure active trial participants can benefit from the trial without the overhead of
      repeatedly deploying new tokens.

<a name="valid-until-feedback"></a>
### 22. How do I keep my tokens active for the entire trial? 

- On the registration page, you can provide feedback for the trial. Use the Feedback button to
    complete a survey to provide feedback.
- Providing feedback will extend the Valid Until date by 6 weeks. You will see the updated
    _Valid Until_ date on the registration page after you submit the feedback. If you have
    multiple origins registered for a trial, we'll only ask you for once per period, rather than
    for every origin.

<a name="verification"></a>
### 23. How are tokens verified?
Tokens are self-contained and verified by Chrome on-device, without any server calls or network access.  

<a name="performance-impact"></a>
### 24. Does token verification have a performance impact?
Verifying a token requires some parsing/decoding of the token, and a crypto call, but the performance impact is unlikely to be observable on any page.
