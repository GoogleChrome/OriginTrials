# Motivation and Explainer

This document describes why we feel it is important to enable browser vendors and standards designers to experiment on the web, and how we believe it is possible to do safely.

_Note: If you’re a developer interested in how to use origin trials, see [this guide](developer-guide.md). New origin trials are announced on [developers.google.com/web/updates](https://developers.google.com/web/updates/), and a list of active trials are [posted here](https://developers.chrome.com/origintrials/#/trials/active)._

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

The risks don’t stop there though. Assuming we did ship a feature experimentally, how can we prevent one large company from adopting it and relying on it at scale, making deprecation or changes to the feature much more difficult? How can we ensure experimental features maintain our high privacy and security standards?

## A sketch of a solution
When reflecting on the problem we outlined earlier, it’s clear that we can’t change the fact that web browser engineers are fundamentally different people to web developers. For that reason, we focused our design efforts on coming up with a safe way to encourage web developers to try new features and provide us feedback while maintaining our ability to modify designs, ensure successful experiments result in eventual interoperability and ensure unsuccessful experiments are not baked into production websites.

We believe these constraints are solvable with ‘origin trials’. **In summary, with origin trials**:
- Developers are able to register for an experimental feature to be enabled on their origin for a fixed period of time measured in months. In exchange, they provide us their email address and agree to give feedback once the experiment ends.
- Usage of these experiments is constrained to remain below Chrome’s deprecation threshold (< 0.5% of all Chrome page loads) by a system which automatically disables the experiment on all origins if this threshold is exceeded.

This design is expanded in detail later, but first consider how this satisfies the constraints we outlined earlier:
- **Encouraging developers to try features and provide feedback:** This system allows us to effectively communicate with these developers for the first time, and we believe the ability to use new features to build demos and prototypes that can be shared with friends, Twitter and beta testers will compel developers to try out these features. Production sites could run limited experiments to gauge the impact of a feature and provide that data as part of the feedback to the feature teams.
- **Maintaining our ability to change our implementation:** The time-limited nature of an origin trial means that these features will self destruct after a few months whatever happens. This gives us clear cover for making breaking changes. Additionally, no large site can rely on these features for their full production traffic thanks to the usage constraint, so there is minimal risk in causing it to become a defacto-standard prematurely.
- **Ensuring successful experiments result in eventual interoperability:** Developers will be very aware that the trial they signed up for will end soon, and that their site must therefore rely on proper feature detection (and degrade gracefully without the feature). Assuming the feature is a success we can then communicate any breaking changes to these developers, who can then make the changes and have the feature work with the finalized syntax.
- **Ensuring unsuccessful experiments are not baked into production websites:** These experimental APIs will all stop working within a few months and developers will be very aware of this when they sign up.

And when considering those additional risks we were worried about:
- **Preventing premature production use from forcing us to keep an experimental feature available:** The automatic usage limiting minimizes the ability for major properties to rely on a feature before it is shipped, and the time-limited design means that inaction results in the feature reverting.
- **Maintaining our high privacy and security standards:** Features launched as origin trials are only experimental in terms of their syntax and semantics. Otherwise these are features going out to actual end users, and as such will go through Chrome’s standard launch process the same as any other feature to ensure a high bar for privacy and security is maintained.

This design also has the additional benefit that developers cannot simply copy paste code using experimental APIs from Stack Overflow as was possible with prefixes.

To summarize this document so far: in order to design good features on the web, the web developer feedback needs to be the most important voice in the standards process. We have been very careful in designing origin trials and are tentatively declaring we believe it is safe way to make that possible.

## High-level design of the solution

Earlier, we outlined two key aspects of the origin trials solution. First, using registration to establish a communication channel with developers. Second, using automated usage limits to prevent burn-in of experimental features and maintain flexibility in implementation. Here, we describe the high-level design of the solution. For more details on the design, see [Origin Trials Framework Design Outline](https://docs.google.com/document/d/1qVP2CK1lbfmtIJRIm6nwuEFFhGhYbtThLQPo3CSTtmg/).

The high-level process for experimental features is:
- An experimental feature is implemented in Chrome.
  - This follows the established [launch process](http://www.chromium.org/blink#launch-process) for web platform features, with the addition of an "Intent to Experiment" and other steps for experiments.
  - Feature authors (i.e. browser engineers) can refer to [this guide](https://dev.chromium.org/blink/origin-trials/running-an-origin-trial) for more details.
- The feature author publishes an origin trial (i.e. experiment) for this feature. The trial must include an end date, to limit the duration.
  - Currently, this is a manual process where a feature is made available for registrations
  - We expect to automate this process, as part of a self-service developer console
- Web developers register their origin to participate in the trial, and receive a trial token.
  - Registrations will be collected via the [developer console](https://developers.chrome.com/origintrials/).
  - More details are available in the [developer guide](developer-guide.md).
- Web developers change their origin to embed the trial token.
  - The token must be embedded in their origin (via \<meta\> tag or HTTP header)
  - Again, details are available in the [developer guide](developer-guide.md).
- Users visit the registered origin, and the Origin Trials Framework exposes the new feature, but only if a valid trial token is found.
  - In Chrome, the framework will extract any token found in the page. The token is checked for origin, feature and expiry date. Only if all the token data is valid, the framework will then allow the feature to be exposed via JavaScript.
- The end date of the trial passes, all issued trial tokens expire, and the framework automatically prevents further access to the feature.
- Feedback on the feature is requested from all registered developers
  - Feedback may also be solicited earlier, during the trial.

### Trial tokens

After considering other mechanisms, we settled on using trial tokens to provide limited access to experimental features. We wanted a mechanism that provided a few important capabilities:
- Features are disabled/off by default, but can be enabled/turned on (almost) immediately after a developer registers their origin
- Features can be enabled with minimal performance impact, and without internet connectivity for every page load
- Developers can control which individual pages have access to a feature, and/or which users have access to a feature

Trial tokens are self-contained, verifiable blobs of text, which can safely be embedded in web pages. Each token is issued for a specific combination of origin and feature. It is not possible for a developer to use one token to enable multiple experimental features on their origin. As well, a token issued for a given origin and feature cannot be used to enable that same feature on a different origin.

### Monitoring and limiting usage

As above, we've defined a cap that usage of a single experimental feature cannot exceed 0.5% of all Chrome page loads

To enforce these limits, usage for each feature with an active origin trial will be monitored regularly. The primary source of data will be anonymous usage statistics collected by Chrome (i.e. [UseCounter](https://cs.chromium.org/chromium/src/third_party/blink/renderer/core/frame/use_counter.h)). With these usage statistics, we can determine if the limit is exceeded for each feature.

When the usage limits are exceeded for a feature, the feature will be disabled for future use. As features are enabled by self-contained trial tokens, a separate mechanism is needed to revoke/override the access provided by otherwise valid tokens. In general we intend to use:
 - Feature-implemented Finch switches, that we require them to implement, or
 - If the API is permissioned, the global permissions kill switch which effectively denies permission to access the new API

There are further steps we can take if there is some unforseen issue with the above kill switches, for example:
 - Remotely replace the token signing key on clients (effectively invalidates all tokens signed with the previous
    key).
 - Remotely disable the entire Origin Trials framework (disables all features exposed via trials).

All the above mechanisms make use of the existing infrastructure in Chrome to push out information to installed browsers. For the feature-implemented switches, this makes use of the [field trials](https://cs.chromium.org/chromium/src/base/metrics/field_trial.h) infrastructure. For the origin trials revocation lists/signing key replacement, this makes use of the infrastructure for component updates.

Different mechanisms will be used to disable a feature, depending on how the
usage limits were exceeded. Initially, the feature-implemented switch will most
commonly be used to disable the specific overused feature.

### Collecting developer feedback
One of the main goals of origin trials is to enable browser vendors to collect developer feedback to help iterate on new features. With the registration process, we will be able to establish a communication channel with developers. We hope that developers will participate in the community around each experimental feature. Feedback is always valuable, so we'll encourage developers to provide informal/adhoc feedback during the trial, or join the mailing list/GitHub repo/etc. for each feature.

## FAQ

*Why does this project exist?*
  - Sometimes we, the web standards community, spend years designing an API that turns out to be fundamentally flawed and then have to go back to the drawing board for a few more years before we can ship the right thing. We believe letting developers safely try experiments in production early will allow us to realize these mistakes earlier, increasing our ability to learn and ship the right things sooner.

*Will all of these experiments ship eventually?*
  - Remember that these are only experiments. There is a good chance that some of them will never ship as standardized APIs on the web. These experimental features are essentially very similar to Chrome flags: an exciting glimpse into one possible future that you can play around with today, and provide feedback for.

*What happens if a large site such as a Google service starts depending on an experimental API?*
  - Origin trials have a built-in safeguard that automatically disables an experimental API globally if its usage exceeds 0.5% of all Chrome page loads. This is to keep usage limited to developers experimenting and below Chrome’s threshold whereby features used on less than 0.5% of all page loads (as measured by Chrome Status) may be deprecated.

*Isn’t this just vendor prefixing all over again?*
  - This topic has been explored in great depth in Alex Russell’s Medium post [Doing Science on the Web](https://medium.com/@slightlylate/doing-science-on-the-web-af26d9be2faa). A couple of key differences include the fact that these features automatically stop working before they become too broadly adopted and that developers cannot simply copy-paste code using an experimental API from the web since they would need to go through the experimental API signup process and accept that the feature is going to shortly stop working.

*Does this change impact how we think about security or privacy on the web?*
  - No, these experimental APIs will all be approved as safe by Chrome’s privacy and security teams.

*Is there any restriction on which websites can sign up to use experimental APIs?*
  - No, any website can sign up to use an experimental API. The only thing to note is that an experiment will be automatically shut off for all domains if it becomes used on more than 0.5% of all Chrome page loads, which means, unsurprisingly, experimental APIs aren’t suitable for full-production use on very large production sites such as the Google home page. Limited experimental use on very large production sites is encouraged, just not a full launch until the feature itself is no longer experimental.

*Is there any review process for signing up a website to access an experimental API?*
  - No. We do not review domain content before generating a token.

*Can one large website prevent anyone from accessing an origin trial (either maliciously or inadvertently)?*
  - We automatically disable any experiment that becomes used too broadly to prevent features from being used in production such that we wouldn't feel OK about breaking them. This does make it possible for a site to maliciously or inadvertently disable an experiment although in that case no developers will be depending on the feature in production since it is experimental so the harm is mitigated.

*What is the structure of trial tokens?*
  - The properties and structure of trials tokens are discussed in detail in the [design document](https://docs.google.com/document/d/1qVP2CK1lbfmtIJRIm6nwuEFFhGhYbtThLQPo3CSTtmg/edit#bookmark=id.jtr9rupl4osm).

*Why shouldn't browsers just use A/B experimentation techniques to test out web platform features*
  - It's common in software to rely on A/B experiments to compare one design to another.  This isn't appropriate when experimenting with platform API surface area because it would make it extremely difficult for developers to reason about the behavior they're seeing from the platform (making the platform much less predictable).  For example, when a user reports an issue, developers try to match the exact browser version and OS when attempting to reproduce the problem.  When there's some other variable they don't know about (eg. rarely a user will have a special browser flag enabled), it inevitably causes a ton of frustration and confusion.
