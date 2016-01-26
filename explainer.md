# Why an Experimental Framework?
[Doing Science On The Web](https://infrequently.org/2015/08/doing-science-on-the-web/) summarizes the past problems in bringing new features to the web platform. Briefly, the web needs new features, and iteration yields the best designs and implementations for those features. However, previous efforts have seen experiments prematurely become de-facto standards, with browser vendors scrambling to implement the features, and web developers coming to rely on these features. These experimental features became burned-in, and resistant to change (or removal), even though better implementations were identified/available.

One of the root causes was that experimental features were available too widely, and thus usage grew unchecked as a result. Ideally, it should be easier to expose and iterate on new features, but reliably limit the experimental population. With a test population of developers committed to providing feedback, and limits in user base size and experiment duration, iteration can happen faster, but without the risk of burn-in.

# What does the Experimental Framework do?
The design of the framework is a work in progress, for the latest see [the public design document](https://docs.google.com/document/d/1qVP2CK1lbfmtIJRIm6nwuEFFhGhYbtThLQPo3CSTtmg/edit?usp=sharing).

Briefly, a feature experiment will follow this process:
- A new feature is implemented in a browser (i.e. Chrome).
- The feature author publishes a time-limited trial (i.e. experiment) for this feature.
- Web developers register their origin to participate in the trial, and receive a trial token.
- Web developers change their origin to embed the trial token.
- Users visit the origin, and the Experimental Framework exposes the new feature, but only if a trial token is found.

# What is the API for the Experimental Framework?
Web developers do not code directly to the Experimental Framework.  The only interaction with the framework is to provide trial tokens to enable access to experimental features.

Web developers embed the provided trial token(s) into their web pages using a meta tag:

```html
<head>
  <meta name="origin-trials" content="contents of token 1">
  <meta name="origin-trials" content="contents of token 2">
  ...
```

How does it work?
- The contents of the trial token are opaque, meaning that they must not be edited, encoded, etc. in any way.  The value of a token should simply be copied into the content attribute of the named meta tag.
- Multiple trial tokens are provided by adding one meta tag per token, using the same name.
- The Experimental Framework in the browser examines all the meta tags matching the `origin-trials` name, until it finds a valid trial token to enable a specific experimental feature.
- Invalid trial tokens are simply ignored - the Experimental Framework does not report errors if it is unable to interpret the contents of the meta tag.
  - This allows different token formats to be used across browsers, while web developers use the same meta tag to embed multiple tokens.

## How do web developers check if an experiment is enabled?
The Experimental Framework does not expose an API to check if a trial or experimental feature is enabled.  Instead, web developers can use common feature detection (and fallback) approaches.  For example:
```js
if (navigator.coolExperimentalFeature) {
  navigator.coolExperimentalFeature...
} else {
  // Fallback code
}
