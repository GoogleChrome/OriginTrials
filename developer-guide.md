# Origin Trials Guide for Web Developers

When an API is available as an origin trial, you are able to register to have it enabled for all users on your origin for a fixed period of time. The purpose is to try it out and give feedback on usability, practicality, and effectiveness of the API to the web standards community. Note that when the trial finishes we will contact you with a request to provide this feedback.

Once your origin has opted into a trial of an experimental feature you can then build demos and prototypes that your friends and beta testing users can try for the duration of the trial without them needing to flip special flags in Chrome.

## How do I enable an experimental feature on my origin?

You can opt any page on your origin into the trial of an experimental feature by [requesting a token for your origin](http://bit.ly/OriginTrialSignup). After filling out the form, we'll send you an email with your token. Note that these tokens are currently generated manually so there may be up to 24 hours delay before you receive the token. Until then we encourage you to start developing with the feature by flipping the relevant flag in about:flags.

When you receive the email, it will include the token, which is specific to your origin and requested feature. There are two ways to provide this token on any pages in your origin:

- Add an `origin-trial` \<meta\> tag to the head of any page. For example this may look something like:
```
<meta http-equiv="origin-trial" content="**insert your token as provided in the the email here**">
```
- If you can configure your server, you can also provide the token on pages using an `Origin-Trial` HTTP header. The resulting response header should look something like:
```
Origin-Trial: **token as provided in the email**
```

If you have trouble configuring pages with your token, or need other help, please contact us at origin-trials-support@google.com.

## What is the thinking behind origin trials?
An exploration of the motivations and reasoning behind origin trials is provided in [the explainer](explainer.md). The TL;DR is that we strongly value the feedback of real web developers (that means you!) during the process of designing and standardizing new APIs. We believe origin trials provide a good way of encouraging that feedback, while being extremely careful that the experiments aren’t used by sites in production-critical roles or as if they’re finalized features.

## What experimental features are currently available?
The [experimental feature tracker](available-trials.md) lists all of the currently available features and their experiment timelines.

## FAQ

1. How can I find out about new experiments when they become available?
  - When you register for an origin trial token you will be automatically added to a mailing list. We'll use this list to send high level updates about the origin trials system, including announcing new features.
  - Additionally, we will be posting updates to [developers.google.com/web/updates/](http://developers.google.com/web/updates/) about each new feature that becomes available as an origin trial.
2. Will all of these experiments ship eventually?
  - These are only experiments and there is a good chance that some of them will never ship as standardized APIs on the web. These experimental features are essentially very similar to Chrome flags: an exciting glimpse into one possible future that you can play around with today, and provide feedback for.
3. What happens if a large site such as a Google service starts depending on an experimental feature?
  - Origin trials have a built-in safeguard that automatically disables an experimental feature globally if its usage exceeds 0.03% of all Chrome page loads. This is to keep usage limited to developers experimenting and below Chrome’s threshold whereby features used on less than 0.03% of all page loads (as measured by [Chrome Status](https://www.chromestatus.com/metrics/feature/popularity)) may be deprecated. 
4. Isn’t this just vendor prefixing all over again?
  - This topic has been explored in depth in Alex Russell’s Medium post [Doing Science on the Web](https://medium.com/@slightlylate/doing-science-on-the-web-af26d9be2faa#.94pf1lwmp). A couple of key differences include:
    - These features automatically stop working before they become too broadly adopted.
    - Developers cannot simply copy-paste sample code using an experimental feature, as they must provide a unique trial token obtained via the experimental feature registration signup process (and accept that the feature is going to shortly stop working).
5. Does this change impact how we think about security or privacy on the web?
  - No, these experimental features have all been held to the same high privacy and security standards as any Chrome platform feature.
6. Is there any restriction on which websites can sign up to use experimental features?
  - Origin trials are available to any website served over HTTPS. Note that there is no policy against specific large sites opting into origin trials, but the system is designed to prevent large populations of the web depending on experimental features. To achieve that, origin trials have a built-in safeguard that automatically limit it globally if its usage exceeds 0.03% of all Chrome page loads. This means that experimental features aren’t suitable for use on large production sites such as the Google home page.
7. Is there any review process for signing up a website to access an experimental features?
  - No. We do not review domain content before generating a token. There is some latency in receiving a token once signing up but that is only because the process for generating tokens is currently manual. We are now investigating building an on-demand token generation system.
8. Is there a way to only enable an origin trial for some of my users or only some pages on my site?
  - Yes, origin trials are enabled on a per-page basis.
9. Will these experiments work in Opera or other web browsers?
  - Not today, but if this model proves to work well then it’s possible that other web browsers may build their own origin trials system. The experimental feature implementations likely won’t be compatible between browsers due to the nature of them being experimental.
