# Origin Trials
Origin trials are an approach to enable safe experimentation with web platform features.

Briefly, the web needs new features, and iteration yields the best designs and implementations for those features. However, previous efforts have seen experiments prematurely become de-facto standards, with browser vendors scrambling to implement the features, and web developers coming to rely on these features. These experimental features became burned-in, and resistant to change (or removal), even though better implementations were identified/available.

One of the root causes was that experimental features were available too widely, and thus usage grew unchecked as a result. Ideally, it should be easier to expose and iterate on new features, but reliably limit the experimental population. With a test population of developers committed to providing feedback, and limits in user base size and experiment duration, iteration can happen faster, but without the risk of burn-in.

Please see the [explainer](explainer.md) to learn more about the problem, and why origin trials is a good solution.

## Signing up for an origin trial
Developers can use [the developer console](https://developers.chrome.com/origintrials/) to request a token to access a feature currently available as an Origin Trial.

## Contents
In addition to describing the problem, and solution, you'll find information for implementing features as experiments, participating in experiments, and details about how it works in Chrome.

* [Motivation/Explainer](explainer.md) - Detailed discussion of the problem, the history and why we think origin trials are a good solution
* [Web Developer Guide](developer-guide.md) - For web developers looking to participate in a trial
* [Feature Author Guide](https://dev.chromium.org/blink/origin-trials/running-an-origin-trial) - For browser engineers looking to experiment with their feature
