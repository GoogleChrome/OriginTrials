# Shipping a feature for use in an origin trial

_For the full context on origin trials, please see the [explainer](explainer.md)._

Here, we describe what is involved in shipping a new browser feature for experimentation in an origin trial. Most importantly, origin trials are integrated into the [launch process for new web platform features](http://www.chromium.org/blink#launch-process). You should be following that overall process (maybe you ended up here from that page).

Contents:
- [How do Origin Trials work in Chrome?](#how-do-origin-trials-work-in-chrome)
- [Is your feature ready to ship?](#is-your-feature-ready-to-ship)
- [Ready to ship - what is the process?](#what-is-the-actual-process-for-shipping)
- [How to integrate your feature with the framework?](#how-to-integrate-your-feature-with-the-framework)
- [Roadmap](#roadmap)

## How do Origin Trials work in Chrome?

The framework will enable features at runtime, on a per-execution-context basis (practically, this will be per-document or per-worker). Features are disabled by default, and only be enabled if a properly signed token, scoped to the origin that it is being presented on, and scoped to the specific feature name, is present in either:

- an HTTP Origin-Trial header in the server response,
- an HTML \<META\> tag in the document's head, or
- (for Dedicated Workers only) the HTTP response or document head of the parent document.

The logic for enabling includes a check of your [runtime feature flag](http://dev.chromium.org/blink/runtime-enabled-features) (even if the origin trials framework isn't being used). This means you can easily test your feature locally, even without any trial tokens.

Origin Trials are being enabled in documents (for both inline and external scripts), and in shared and dedicated workers. (Note that for shared workers, HTTP headers are the only way to enable trials. Dedicated workers will also inherit any trials enabled by their parent document).

If an experiment gets out of hand (*way* too popular to be an experiment anymore, for instance),  we’ll be able to turn it off remotely, for all origins. SImilarly, if there turns out to be major problems with the implementation of the framework itself, we’ll be able to turn it off completely, and disable all trials. (Hopefully we don’t have to do that, but we're still in the early stages of origin trials, and we’re being careful.)

## Is your feature ready to ship?

Before shipping as trial, your feature needs to be ready for both web developers and users. Your feature must satisfy the following:
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

## What is the actual process for shipping?

Shipping as an origin trial requires the following:
- Let the origin trials team know about your feature
  - We can help you through this process, and make sure origin trials will meet your needs
- Make sure your feature is ready to ship ([see above](#is-your-feature-ready-to-ship))
 - Specifically, get Chrome launch approval!
- Integrate with the origin trials framework ([see below](#how-to-integrate-your-feature-with-the-framework))
- Send an [Intent to Experiment](http://www.chromium.org/blink#launch-process)
- Ship the feature in Chrome (i.e. all code landed prior to beta)
- Publicize the availability of the feature as an origin trial
  - Typically, this would be publishing a prepared blog post
  - The origin trials team will add your feature to the [sign up form](https://bit.ly/OriginTrialSignup), and to the list of [available trials](available-trials.md).


Note that these steps are not meant to be sequential. For example, you can
certainly start integrating your feature with origin trials prior to getting
various launch approvals.

## How to integrate your feature with the framework?
To expose your feature via the origin trials framework, you’ll need to configure [RuntimeEnabledFeatures.in](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/platform/RuntimeEnabledFeatures.in).  This is explained in the file, but you use `origin_trial_feature_name` to associate your runtime feature flag with a name for your origin trial.  The name can be the same as your runtime feature flag, or different.  Eventually, this configured name will be used in the Origin Trials developer console (still under development). You can have both `status=experimental` and `origin_trial_feature_name` if you want your feature to be enabled either by using the `--enable-experimental-web-platform-features` flag **or** the origin trial:

```
MyFeature status=experimental, origin_trial_feature_name=MyFeature
```

Once configured, there are two mechanisms to gate access to your feature behind an origin trial. You can use either mechanism, or both, as appropriate to your feature implementation.

1. A native C++ method that you can call in Blink code at runtime to expose your feature: `bool OriginTrials::myFeatureEnabled()`
2. An IDL attribute [\[OriginTrialEnabled\]](https://chromium.googlesource.com/chromium/src/+/master/third_party/WebKit/Source/bindings/IDLExtendedAttributes.md#OriginTrialEnabled-i_m_a_c) that you can use to automatically expose and hide JavaScript methods/attributes/objects. This attribute works very similar to \[RuntimeEnabled\].
```
[OriginTrialEnabled=MyFeature]
partial interface Navigator {
     readonly attribute MyFeatureManager myFeature;
}
```

If `OriginTrialEnabled` is used with IDL bindings, you may need to manually install the appropriate methods in the V8 bindings code. See [V8Binding.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/bindings/core/v8/V8Binding.cpp)'s `installOriginTrials` and [V8BindingForModules.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/bindings/modules/v8/V8BindingForModules.cpp)'s `installOriginTrialsForModules`. Eventually, the V8 bindings code will be generated automatically (See [crbug.com/615060](https://bugs.chromium.org/p/chromium/issues/detail?id=615060)).

**NOTE:** Your feature implementation must not persist the result of the enabled check. Your code should simply call `OriginTrials::myFeatureEnabled()` as often as necessary to gate access to your feature.


#### Limitations
What you can't do, because of the nature of these Origin Trials, is know at either browser or renderer startup time whether your feature is going to be used in the current page/context. This means that if you require lots of expensive processing to begin (say you index the user's hard drive, or scan an entire city for interesting weather patterns,) that you will have to either do it on browser startup for *all* users, just in case it's used, or do it on first access. (If you go with first access, then only people trying the experiment will notice the delay, and hopefully only the first time they use it.). We are investigating providing a method like `OriginTrials::myFeatureShouldInitialize()` that will hint if you should do startup initialization.  For example, this could include checks for trials that have been revoked (or throttled) due to usage, if the entire EF has been disabled, etc.  The method would be conservative and assume initialization is required, but it could avoid expensive startup in some known scenarios.

Similarly, if you need to know in the browser process whether a feature should be enabled, then you will have to either have the renderer inform it at runtime, or else just assume that it's always enabled, and gate access to the feature from the renderer.

### Testing
If you want to test your code's interactions with the framework, you'll need to generate some tokens of your own. To generate your own tokens, use [/tools/origin_trials/generate_token.py](https://code.google.com/p/chromium/codesearch#chromium/src/tools/origin_trials/generate_token.py).
You can generate signed tokens for localhost, or for 127.0.0.1, or for any origin that you need to help you test. For example:

```
tools/origin_trials/generate_token.py --key-file=tools/origin_trials/eftest.key http://localhost:8000 MyFeature
```

The `eftest.key` file is the private key for the test keypair used by Origin Trials unit tests (tokens generated with this key will **not** work in the browser by default; see the [Developer Guide](developer-guide.md) for instructions on creating real tokens). To use a test token with the browser, run Chrome with the command-line flag:

```
--origin-trial-public-key=dRCs+TocuKkocNKa0AtZ4awrt9XKH2SQCI6o4FY6BNA=
```

This is the public key associated with `eftest.key`. If it doesn't work, see [trial_token_unittest.cc](https://cs.chromium.org/chromium/src/content/common/origin_trials/trial_token_unittest.cc).

## Roadmap
All of this may change, as we respond to your feedback about the framework itself. Please let us know how it works, and what's missing!

What we're planning in the near future:
- [Auto-generate V8 bindings code to install trials](https://bugs.chromium.org/p/chromium/issues/detail?id=615060)
- [Revoking Tokens](https://bugs.chromium.org/p/chromium/issues/detail?id=582042)
- [Supporting origin trials in Dev Tools](https://bugs.chromium.org/p/chromium/issues/detail?id=607555)
- Origin trials in compositor workers

What we're considering (no guarantees!) for later:
- [iOS support](https://bugs.chromium.org/p/chromium/issues/detail?id=582056)
- [Extra data attached to tokens](https://bugs.chromium.org/p/chromium/issues/detail?id=582060)
- Scoping tokens to a path

To follow the most up-to-date progress and plans, filter in crbug.com for “[component:Internals>OriginTrials](https://bugs.chromium.org/p/chromium/issues/list?q=component:Internals%3EOriginTrials)”.
