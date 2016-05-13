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
