# Web Install

This repository incubates the **Web Install** specification. It defines two
ways for a website to ask the user agent to install a web application:

- the promise-based `navigator.install()` API; and
- the declarative `<install>` HTML element.

Both entry points use one shared model for resolving a target manifest,
validating application identity, obtaining user consent, installing the
application, protecting installed-state privacy, and reporting outcomes.

## Documents

- [Specification source](./index.html)
- [`<install>` element explainer](./explainer-manifest-url.md)
- [`navigator.install()` explainer](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/WebInstall/explainer.md)
- [Rendered draft](https://wicg.github.io/install-element/)

The specification is an integrated ReSpec Community Group Draft. Inline issues
identify behavior that still needs interoperability, privacy, or ownership
decisions.

## Participate

- [Issue tracker](https://github.com/WICG/install-element/issues)
- [Chromium status: `navigator.install()`](https://chromestatus.com/feature/5183481574850560)
- [Chromium status: `<install>`](https://chromestatus.com/feature/5152834368700416)

## Editors

- [Lia Hiscock](https://github.com/LiaHiscock), Microsoft
- [Lu Huang](https://github.com/LuHuangMSFT), Microsoft

## Related specifications

- [Web Application Manifest](https://www.w3.org/TR/appmanifest/)
- [Manifest Incubations](https://wicg.github.io/manifest-incubations/)
- [WHATWG HTML](https://html.spec.whatwg.org/)
- [Capability Elements](https://wicg.github.io/PEPC/geolocation-element.html)
