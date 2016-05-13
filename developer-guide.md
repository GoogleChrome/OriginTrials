# Developer Guide

When an API is available as an origin trial, you are able to register to have it enabled for all users on your origin for a fixed period of time. The purpose is to try it out and give feedback on usability, practicality, and effectiveness of the API to the web standards community. Note that when the trial finishes we will contact you with a request to provide this feedback.

Once your origin has opted into a trial of an experimental feature you can then build demos and prototypes that your friends and beta testing users can try for the duration of the trial without them needing to flip special flags in Chrome.

## Useful links
- _\[Coming soon\]_ Form to register for an origin trial token (read more about these below)

## What is the thinking behind origin trials?
An exploration of the motivations and reasoning behind origin trials is provided in [the explainer](explainer.md). The TL;DR is that we strongly value the feedback of real web developers (that means you!) during the process of designing and standardizing new APIs. We believe origin trials provide a good way of encouraging that feedback, while being extremely careful that the experiments aren’t used by sites in production-critical roles or as if they’re finalized features.

## What experimental features are currently available?
The [experimental feature tracker](available-trials) lists all of the currently available features and their experiment timelines.

## How do I enable an experimental feature on my origin?
You can opt any page on your origin into the trial of an experimental feature by adding an origin-trial <meta> tag to the head of that page. For example this may look something like:
```
<meta http-equiv="origin-trial" content="**insert your site’s token here**">
```

We’re currently working on building a console where you can generate these tokens in an automated and self-service way, but for now you can get tokens by filling out the Google sign-up form (linked above), and specifying your origin and the feature you’d like to enable. After registering, we'll send you an email with your token. Note that these tokens are currently generated manually so there may be a delay.
