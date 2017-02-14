# Running an origin trial for a feature

_For the full context on origin trials, please see the [explainer](explainer.md)._

Here, we describe what is involved in running an origin trials experiment for a new browser feature. Most importantly, origin trials are integrated into the [launch process for new web platform features](http://www.chromium.org/blink#launch-process). You should be following that overall process (maybe you ended up here from that page).

Contents:
- [Should you run an Origin Trial?](#should-you-run-an-origin-trial)
- [How do Origin Trials work in Chrome?](#how-do-origin-trials-work-in-chrome)
- [Is your feature ready to be an origin trial?](#is-your-feature-ready-to-be-an-origin-trial)
- [What is the process to run an origin trial?](#what-is-the-actual-process-to-run-an-origin-trial)
- [How to integrate your feature with the framework?](#how-to-integrate-your-feature-with-the-framework)
- [Roadmap](#roadmap)

## Should you run an Origin Trial?

Origin Trials are intended to be used to ensure we design the best possible features by getting feedback from developers before the standard is finalized. They may also be used to prove developer interest in a feature proposal that is otherwise undesired due to an expected lack of interest.

*If you're planning to run an Origin Trial please first schedule a meeting with the Origin Trials core team to quickly talk over your feature and the reason for running the trial.* To set up this meeting you can email owencm@chromium.org or chasej@chromium.org. Google employees can alternatively schedule a meeting directly with origin-trials-core@google.com.

## How do Origin Trials work in Chrome?

The framework will enable features at runtime, on a per-execution-context basis (practically, this will be per-document or per-worker). Features are disabled by default, and only be enabled if a properly signed token, scoped to the origin that it is being presented on, and scoped to the specific feature name, is present in either:

- an HTTP Origin-Trial header in the server response,
- an HTML \<META\> tag in the document's head, or
- (for Dedicated Workers only) the HTTP response or document head of the parent document.

The logic for enabling includes a check of your [runtime feature flag](http://dev.chromium.org/blink/runtime-enabled-features) (even if the origin trials framework isn't being used). This means you can easily test your feature locally, even without any trial tokens.

Origin Trials are being enabled in documents (for both inline and external scripts), and in service, shared, and dedicated workers. (Note that for service workers and shared workers, HTTP headers are the only way to enable trials. Dedicated workers will also inherit any trials enabled by their parent document).

If an experiment gets out of hand (*way* too popular to be an experiment anymore, for instance),  we’ll be able to turn it off remotely, for all origins. SImilarly, if there turns out to be major problems with the implementation of the framework itself, we’ll be able to turn it off completely, and disable all trials. (Hopefully we don’t have to do that, but we're still in the early stages of origin trials, and we’re being careful.)

## Is your feature ready to be an origin trial?

Before running an origin trial experiment, your feature needs to be ready for both web developers and users. Your feature must satisfy the following:
- Be approved by the internal Chrome launch review process
  - Users may be exposed to your feature without opting in, so the appropriate measures must be taken for privacy, security, etc.
- Have a way to remotely disable the feature
  - Origin trials provides infrastructure to disable a feature (or a specific origin), but this only applies to the exposure as an origin trial. That means, any interface(s) controlled by the trial will be disabled, but it will still be possible to enable the feature via its runtime flag. As well, all of the token validation/revocation happens in the renderer.
  - If the previous point is not sufficient for disabling the feature, you should implement a kill switch that allows your feature to be disabled remotely via Finch
  - This can use the existing functionality in PermissionContextBase or base::FeatureList, or be a feature-specific implementation.
- Have UMA metrics to track feature usage
  - Typically you will record usage with [UseCounter](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/frame/UseCounter.h)
  - The feature must have a corresponding entry in the enum [UseCounter::Feature](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/frame/UseCounter.h&q=%22enum%20Feature%22&sq=package:chromium&type=cs&l=65).
  - For any JavaScript-exposed API, usage can be recorded easily via one of the [\[Measure\]](https://chromium.googlesource.com/chromium/src/+/master/third_party/WebKit/Source/bindings/IDLExtendedAttributes.md#Measure-m_a_c) or [\[MeasureAs\]](https://chromium.googlesource.com/chromium/src/+/master/third_party/WebKit/Source/bindings/IDLExtendedAttributes.md#MeasureAs-m_a_c) IDL attributes.
  - If not exposed via IDL, the appropriate `UseCounter::count*()` method can be used directly from your feature implementation.
  - Your feature may use a different UMA metric to track usage. For example, if it's not feasible to integrate with UseCounter, usage is best tracked outside a renderer, etc.
- Have an established community for discussion of the feature
  - At a minimum, this should be a WICG group, Github repo, etc. Anywhere developers can find discussion or log issues about your feature
  - The origin trials system will facilitate collecting feedback from web developers. However, the goal is to have web developers participate in the existing community around the feature
- Prepared a blog post/article/landing page introducing the feature
  - There needs to be single link that will provide details about the feature
  - Should make it clear how developers provide feedback/log issues for your feature
  - This could be the README.md in your Github repo, or any other page of your choice
  - Should include details about availability via origin trials

## What is the timeline for running a trial and collecting feedback?
Please see our [overview of the timeline for running a trial and collecting feedback](https://docs.google.com/spreadsheets/d/1QVuhf96PZdnrfUQP4rRMhptkQPJe2K1EmJPuT6iS0a4/edit#gid=0). Contact experimentation-dev@chromium.org with any questions.

## What is the actual process to run an origin trial?

Running an origin trial requires the following:
- Let the origin trials team know about your feature
  - We can help you through this process, and make sure origin trials will meet your needs
- Make sure your feature is ready to run an origin trial experiment ([see above](#is-your-feature-ready-to-be-an-origin-trial))
 - Specifically, get Chrome launch approval!
- Integrate with the origin trials framework ([see below](#how-to-integrate-your-feature-with-the-framework))
- Send an [Intent to Experiment](http://www.chromium.org/blink#launch-process)
- Land the feature in Chrome prior to beta
- Publicize the availability of the feature as an origin trial
  - Typically, this would be publishing a prepared blog post
  - The origin trials team will add your feature to the [sign up form](https://bit.ly/OriginTrialSignup), and to the list of [available trials](available-trials.md).
  - [See below](#adding-your-feature-to-the-sign-up-form) for more details

Note that these steps are not meant to be sequential. For example, you can
certainly start integrating your feature with origin trials prior to getting
various launch approvals.

### Adding your feature to the sign up form

Before your feature can be added to the sign up form, the landing page must be available to web developers ([see above](#is-your-feature-ready-to-run-an-origin-trial)). The origin trials team needs some documentation for web developers that sign up (which has happened within minutes of a feature being added to the form!).

In some cases, this may lead to a chicken-and-egg problem. For example, you may not want to publish a blog post until the feature is added the form. If the blog post has detailed information on joining the origin trial, it doesn't make sense to publish and have web developers unable to see your feature on the sign up form. Conversely, if the feature is added to the form first, web developers can (and have in the past) seen the change and signed up before the docs are ready.


## How to integrate your feature with the framework?
The integration instructions now live in the Chromium source repo:
https://chromium.googlesource.com/chromium/src/+/master/docs/origin_trials_integration.md

## Roadmap
All of this may change, as we respond to your feedback about the framework itself. Please let us know how it works, and what's missing!

What we're planning in the near future:
- [Auto-generate V8 bindings code to install trials](https://bugs.chromium.org/p/chromium/issues/detail?id=615060)
- [Revoking Tokens](https://bugs.chromium.org/p/chromium/issues/detail?id=582042)
- [Supporting origin trials in Dev Tools](https://bugs.chromium.org/p/chromium/issues/detail?id=607555)
- Origin trials in Animation Worklets (was compositor workers)

What we're considering (no guarantees!) for later:
- [iOS support](https://bugs.chromium.org/p/chromium/issues/detail?id=582056)
- [Extra data attached to tokens](https://bugs.chromium.org/p/chromium/issues/detail?id=582060)
- Scoping tokens to a path

To follow the most up-to-date progress and plans, filter in crbug.com for “[component:Internals>OriginTrials](https://bugs.chromium.org/p/chromium/issues/list?q=component:Internals%3EOriginTrials)”.
