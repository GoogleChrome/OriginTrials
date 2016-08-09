# Shipping a feature for use in an origin trial

For the full context on origin trials, please see the [explainer](explainer.md). Here, we describe what is involved in shipping a new browser feature for experimentation in an origin trial.

The process and functionality below is targetting the _Chrome M52_ milestone.

## What is provided by the Origin Trials framework in Chrome?

The framework will enable features at runtime, on a per-execution-context basis (practically, this will be per-document or per-worker). Features are disabled by default, and only be enabled if a properly signed token, scoped to the origin that it is being presented on, and scoped to the specific feature name, is present in either:

- an HTTP Origin-Trial header in the server response,
- an HTML <META> tag in the document's head, or
- (for Dedicated Workers only) the HTTP response or document head of the parent document. 

The logic for enabling includes a check of your [runtime feature flag](http://dev.chromium.org/blink/runtime-enabled-features) (even if the origin trials framework isn't being used). This means you can easily test your feature locally, even without any trial tokens.

Origin Trials are being enabled in documents (for both inline and external scripts), and in shared and dedicated workers. (Note that for shared workers, HTTP headers are the only way to enable trials. Dedicated workers will also inherit any trials enabled by their parent document).

If an experiment gets out of hand (*way* too popular to be an experiment anymore, for instance),  we’ll be able to turn it off remotely, for all origins. SImilarly, if there turns out to be major problems with the implementation of the framework itself, we’ll be able to turn it off completely, and disable all trials. (Hopefully we don’t have to do that, but we're still in the early stages of origin trials, and we’re being careful.)

## What does this mean for you as a feature author?

First, origin trials are integrated into the [launch process for new web platform features](http://www.chromium.org/blink#launch-process). You should be following that overall process (maybe you ended up here from that page).

To ship as an origin trial, your feature must meet the following eligibility criteria:
- Be approved by the internal Chrome launch review process
  - Users may be exposed to your feature without opting in, so the appropriate measures must be taken for privacy, security, etc.
- Implement a kill switch that allows your feature to be disabled via Finch
  - This can use the existing functionality in PermissionContextBase or base::FeatureList, or be a feature-specific implementation.
- Record usage of the feature using [UseCounter](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/frame/UseCounter.h)
  - The feature must have a corresponding entry in the enum [UseCounter::Feature](https://code.google.com/p/chromium/codesearch#chromium/src/third_party/WebKit/Source/core/frame/UseCounter.h&q=%22enum%20Feature%22&sq=package:chromium&type=cs&l=65).
  - For any JavaScript-exposed API, usage can be recorded easily via one of the [\[Measure\]](https://chromium.googlesource.com/chromium/src/+/master/third_party/WebKit/Source/bindings/IDLExtendedAttributes.md#Measure-m_a_c) or [\[MeasureAs\]](https://chromium.googlesource.com/chromium/src/+/master/third_party/WebKit/Source/bindings/IDLExtendedAttributes.md#MeasureAs-m_a_c) IDL attributes.
  - If not exposed via IDL, the appropriate `UseCounter::count*()` method can be used directly from your feature implementation.

### Integration with the framework
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

If `OriginTrialEnabled` is used with IDL bindings, you may need to manually install the appropriate methods in the V8 bindings code. See [V8Binding.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/bindings/core/v8/V8Binding.cpp)'s `installOriginTrials` and [V8BindingForModules.cpp](https://cs.chromium.org/chromium/src/third_party/WebKit/Source/bindings/modules/v8/V8BindingForModules.cpp)'s `installOriginTrialsForModules` [[crbug.com/615060](https://bugs.chromium.org/p/chromium/issues/detail?id=615060)].

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

Stretch goals (for Chrome M52):
- [Revoking Tokens](https://code.google.com/p/chromium/issues/detail?id=582042)
- Revoking entire features

What we're not going to have (for Chrome M52):
- [iOS support](https://code.google.com/p/chromium/issues/detail?id=582056)
- [Extra data attached to tokens](https://code.google.com/p/chromium/issues/detail?id=582060)
- Scoping tokens to a path
- Origin trials in compositor workers
- Persistent origin trials in cached service worker scripts

To follow the most up-to-date progress and plans, filter in crbug.com for “[component:Internals>OriginTrials](https://bugs.chromium.org/p/chromium/issues/list?q=component:Internals%3EOriginTrials)”.
