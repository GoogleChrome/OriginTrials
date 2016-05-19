# Motivation and Explainer

This document describes why we feel it is important to enable browser vendors and standards designers to experiment on the web, and how we believe it is possible to do safely.

Note: If you’re a developer interested in how to use origin trials, see [this guide](developer-guide.md).

## The problem
It is well known that **iteration and fast feedback cycles are key to designing high quality software**. The process of development is one of designing interfaces and abstractions, attempting to use them or have your friends and colleagues use them, realizing they’re leaky or broken, fixing those problems, and then repeating the cycle.

This works great in an environment where you can cycle through the process many times, but **this process is very challenging in some cases**, especially when:
- You are not the target user of the interface or abstraction
- The target user is very different from you
- The target users aren’t willing or able to try it out in actual usage
- Changing the interface or abstraction is difficult or costly, once shipped

Interestingly, the web is approximately a worst case example of all of the above:
- Browser engineers are designing abstractions and interfaces for web developers, not for browser engineers
- Browser engineers often have very different backgrounds, knowledge and experiences than web developers
- Web developers generally aren’t compelled to try features until they can use them to build a site and have their users interact with it
- Making breaking changes to web APIs is extremely challenging; even before shipping, changes are difficult to make due to the need to drive consensus between individuals across companies

The resulting situation is one in which **features shipping on the web take a long time to design, and are then frozen into standards well before they have been significantly road-tested by real web developers**.

This has resulted in a number of unfortunate situations such as us spending years designing a [declarative disaster](http://alistapart.com/article/application-cache-is-a-douchebag) when we should have been iterating to design a [programmable proxy](http://www.html5rocks.com/en/tutorials/service-worker/introduction/).

To summarize, we believe it would be hugely beneficial to the web if we had a safe way to enable browser vendors to iteratively develop features using feedback from web developers.

## The risks and past attempts to solve the problem
This problem is difficult to solve. Browser vendors previously attempted to solve it with [vendor prefixes](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix), which allowed browser vendors to ship features prior to standardization. The theory was that this would allow developers to provide early feedback and give vendors the ability to change APIs before standardizing them. The intention was that the ugly appearance of prefixes would make it obvious that developers shouldn’t depend on the features in production.

Unfortunately, what actually happened was that developers didn’t consider these experimental but rather used prefixed features in production sites that have survived for years and years. As a result, the web is now full of sites that in the worst case only support some features in the browsers that happened to have a prefixed form at the time, and in the best case ship huge CSS files full of redundant vendor prefixes.

This experience makes it clear that **any system to enable experimentation should also converge to eventual interoperability for successful features**.

Conversely, for unsuccessful features we should ensure that any system to enable experimentation prevents designs we later regret from being baked into the web.

The risks don’t stop there though. Assuming we did ship a feature experimentally, how can we prevent one large company from adopting it and then force us into a game of high stakes deprecation chicken? How can we ensure experimental features maintain our high privacy and security standards?

## A sketch of a solution
When reflecting on the problem we outlined earlier, it’s clear that we can’t change the fact that web browser engineers are fundamentally different people to web developers. For that reason, we focused our design efforts on coming up with a safe way to encourage web developers to try new features and provide us feedback while maintaining our ability to modify designs, ensure successful experiments result in eventual interoperability and ensure unsuccessful experiments are not baked into production websites.

We believe these constraints are solvable with ‘origin trials’. **In summary, with origin trials**:
- Developers are able to register for an experimental feature to be enabled on their origin for a fixed period of time measured in months. In exchange, they provide us their email address and agree to give feedback once the experiment ends.
- Usage of these experiments is constrained to remain below Chrome’s deprecation threshold (< 0.03% of all Chrome page loads) by a system which automatically disables the experiment on all origins if this threshold is exceeded.

This design is expanded in detail later, but first consider how this satisfies the constraints we outlined earlier:
- **Encouraging developers to try features and provide feedback:** This system allows us to effectively communicate with these developers for the first time, and we believe the ability to use new features to build demos and prototypes that can be shared with friends, Twitter and beta testers will compel developers to try out these features.
- **Maintaining our ability to change our implementation:** The time-limited nature of an origin trial means that these features will self destruct after a few months whatever happens. This gives us clear cover for making breaking changes. Additionally, no large site can use these features in production thanks to the usage constraint, so we cannot be pressured into extending origin trials by powerful parties.
- **Ensuring successful experiments result in eventual interoperability:** Developers will be very aware that the trial they signed up for will end soon, and that their site must therefore rely on proper feature detection (and degrade gracefully without the feature). Assuming the feature is a success we can then communicate any breaking changes to these developers, who can then make the changes and have the feature work with the finalized syntax.
- **Ensuring unsuccessful experiments are not baked into production websites:** These experimental APIs will all stop working within a few months and developers will be very aware of this when they sign up.

And when considering those additional risks we were worried about:
- **Preventing one large company from forcing us to keep an experimental feature available:** The automatic usage limiting minimizes the ability for major properties to attempt to force our hand, and the time-limited design means that inaction results in the desirable outcome.
- **Maintaining our high privacy and security standards:** Features launched as origin trials are only experimental in terms of their syntax and semantics. Otherwise these are features going out to actual end users, and as such will go through Chrome’s standard launch process the same as any other feature to ensure a high bar for privacy and security is maintained.

This design also has the additional benefit that developers cannot simply copy paste code using experimental APIs from Stack Overflow as was possible with prefixes.

To summarize this document so far: in order to design good features on the web, the web developer feedback needs to be the most important voice in the standards process. We have been very careful in designing origin trials and are tentatively declaring we believe it is safe way to make that possible.

## High-level design of the solution

Earlier, we outlined two key aspects of the origin trials solution. First, using registration to establish a communication channel with developers. Second, using automated usage limits to prevent burn-in of experimental features and maintain flexibility in implementation. Here, we describe the high-level design of the solution. For more details on the design, see [Origin Trials Framework Design Outline](https://docs.google.com/document/d/1qVP2CK1lbfmtIJRIm6nwuEFFhGhYbtThLQPo3CSTtmg/).

The high-level process for experimental features is:
- An experimental feature is implemented in Chrome.
  - This follows the established [launch process](http://www.chromium.org/blink#launch-process) for web platform features, with the addition of an "Intent to Experiment" and other steps for experiments.
  - Feature authors (i.e. browser engineers) can refer to [this guide](ship-as-trial.md) for more details.
- The feature author publishes an origin trial (i.e. experiment) for this feature. The trial must include an end date, to limit the duration.
  - Currently, this is a manual process where a feature is made available for registrations
  - We expect to automate this process, as part of a self-service developer console
- Web developers register their origin to participate in the trial, and receive a trial token.
  - Registrations will be collected via a Google form.
  - More details are available in the [developer guide](developer-guide.md).
- Web developers change their origin to embed the trial token.
  - The token must be embedded in their origin (via \<meta\> tag or HTTP header)
  - Again, details are available in the [developer guide](developer-guide.md).
- Users visit the registered origin, and the Origin Trials Framework exposes the new feature, but only if a valid trial token is found.
  - In Chrome, the framework will extract any token found in the page. The token is checked for origin, feature and expiry date. Only if all the token data is valid, the framework will then allow the feature to be exposed via JavaScript.
- The end date of the trial passes, all issued trial tokens expire, and the framework automatically prevents further access to the feature.
- Feedback on the feature is requested from all registered developers
  - Feedback may also be solicited earlier, during the trial.
  - Developers must provide feedback, or will not be able to register for new origin trials.

### Trial tokens

After considering other mechanisms, we settled on using trial tokens to provide limited access to experimental features. We wanted a mechanism that provided a few important capabilities:
- Features are disabled/off by default, but can be enabled/turned on (almost) immediately after a developer registers their origin
- Features can be enabled with minimal performance impact, and without internet connectivity for every page load
- Developers can control which individual pages have access to a feature, and/or which users have access to a feature

Trial tokens are self-contained, verifiable blobs of text, which can safely be embedded in web pages. Each token is issued for a specific combination of origin and feature. It is not possible for a developer to use one token to enable multiple experimental features on their origin. As well, a token issued for a given origin and feature cannot be used to enable that same feature on a different origin.

### Monitoring and limiting usage

As above, we've defined a cap that usage of a single experimental feature cannot exceed 0.03% of all Chrome page loads (i.e. same as Chrome’s deprecation threshold). In addition, usage of a single feature, by a single origin, cannot exceed 0.01% of all Chrome page loads.

To enforce these limits, usage for each feature with an active origin trial will be monitored regularly. The primary source of data will be anonymous usage statistics collected by Chrome (i.e. [UseCounter](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/frame/UseCounter.h)). With these usage statistics, we can determine if the limit is exceeded for each feature. Depending the usage pattern for features, it may also be feasible to detect when features are approaching, but haven't exceeded, the limit.

When the usage limits are exceeded for a feature, the feature will be disabled for future use. As features are enabled by self-contained trial tokens, a separate mechanism is needed to revoke/override the access provided by otherwise valid tokens. There are a number of potential mechanisms to turn off features:
- Feature-implemented switches for disabling a specific feature.
- Origin trials infrastructure to revoke access for features and/or tokens (partially
  implemented).
- Replace the token signing key (effectively invalidates all tokens signed with the previous
  key).
- Disable the entire Origin Trials framework (disables all features exposed via
  trials).

All the above mechanisms make use of the existing infrastructure in Chrome to push out information to installed browsers. For the feature-implemented switches, this makes use of the [field trials](https://code.google.com/p/chromium/codesearch#chromium/src/base/metrics/field_trial.h) infrastructure. For the origin trials revocation lists/signing key replacement, this makes use of the infrastructure for component updates.

Different mechanisms will be used to disable a feature, depending on how the
usage limits were exceeded. Initially, the feature-implemented switch will most
commonly be used to disable the specific overused feature. 

### Collecting developer feedback
One of the main goals of origin trials is to enable browser vendors to collect developer feedback to help iterate on new features. With the registration process, we will be able to establish a communication channel with developers.

The primary means for collecting feedback will be structured surveys at the end of each trial. Using surveys will allow feature authors to collect both qualitative and quantitative data about various aspects of their feature, such as the effectiveness and ergonomics in addressing a particular developer pain point. Initially, we'll have a separate survey for each feature, collected via a Google form. Over time, we expect to provide a consistent survey approach across features. The collected feedback will be anonymized and aggregated, and we'll make a summary of the feedback publicly available.

Beyond surveys, we hope that developers will participate in the community around each experimental feature. Feedback is always valuable, so we'll encourage developers to provide informal/adhoc feedback during the trial, or join the mailing list/GitHub repo/etc. for each feature. 

## FAQ

1. Can one large website prevent anyone from accessing an origin trial (either maliciously or inadvertently)?
  - We have a system to monitor and limit total usage of each feature. This same system constrains usage for each origin to an even smaller limit (i.e. 0.01% of all page loads). This is to prevent a single origin from disabling the experiment for all other origins which are safely within the usage limits. Thus, only the origin that exceeds the smaller per-site limits would be disabled for further use.

2. What is the structure of trial tokens?
  - The properties and structure of trials tokens are discussed in detail in the [design document](https://docs.google.com/document/d/1qVP2CK1lbfmtIJRIm6nwuEFFhGhYbtThLQPo3CSTtmg/edit#bookmark=id.jtr9rupl4osm).

3. What happens to the developer feedback collected in surveys?
  - All data collected is subject to the [Chrome](https://www.google.com/intl/en/chrome/browser/privacy/) and [Google](http://www.google.com/policies/privacy/) privacy policies.
