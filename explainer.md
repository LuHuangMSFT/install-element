# Web Install

## Authors

- [Lia Hiscock](https://github.com/LiaHiscock), Microsoft
- [Mike West](https://github.com/mikewest), Google
- Diego González, Microsoft

## Participate

- [Issue tracker](https://github.com/WICG/install-element/issues)
- [Specification](./index.html)

## Summary

Web Install gives websites a standard, user-mediated way to offer installation
of a web application. It has two complementary entry points:

- `navigator.install()` lets a site use its own installation UI and observe a
  privacy-limited promise outcome.
- `<install>` provides a declarative, user-agent-rendered control whose
  validated activation is a stronger signal of user intent.

Both entry points use the same target-resolution, manifest-processing,
identity, consent, installation, and privacy model. The
[specification](./index.html) is the normative source for that shared behavior.

## User problem

Web application installation affordances vary across browsers and platforms.
They can be hidden in browser menus, difficult to explain consistently, and
limited to the application associated with the current page. Sites cannot use
one standard mechanism to offer installation from an application suite,
discovery page, or independent catalog.

Web Install lets the user remain in context while the browser presents
trustworthy confirmation UI that distinguishes the requesting site from the
application being installed.

## Goals

- Offer installation of the current document's web application.
- Offer installation of an application identified by its manifest URL.
- Support same-origin suites and cross-origin discovery/catalog use cases.
- Require user activation and browser-controlled confirmation.
- Avoid directly exposing whether a target application is already installed.
- Provide a zero-script declarative entry point with ordinary HTML fallback.
- Keep the two entry points aligned through one normative processing model.

## Non-goals

- Silent or unattended installation.
- Installing native apps, extensions, or content without a web app manifest.
- Defining one exact OS representation of an installed web application.
- Enumerating installed applications.
- Requiring offline capability or a service worker as part of installation.

## Entry points

### `navigator.install()`

Install the web application associated with the current document:

```js
try {
  await navigator.install();
} catch (error) {
  if (error.name === "AbortError") {
    // The operation did not complete.
  } else if (error.name === "DataError") {
    // The target manifest or identity data was invalid.
  }
}
```

Install an application identified by a manifest URL:

```js
await navigator.install({
  manifest: "https://music.example/manifest.webmanifest",
  manifestId: "https://music.example/app"
});
```

`manifestId` is an optional assertion about the processed manifest identity.
When it is omitted, the target manifest is expected to declare an explicit
`id`.

### `<install>`

Install the current document's application:

```html
<install>
  <a href="/install-help">How to install this app</a>
</install>
```

Install an application identified by a manifest URL:

```html
<install
  manifest="https://music.example/manifest.webmanifest"
  manifestid="https://music.example/app"
  oninstallresult="handleInstallResult(event)">
  <a href="https://music.example/">Open Music Example</a>
</install>
```

A supporting browser replaces the fallback children with a native,
user-agent-rendered control. An unsupported browser treats `<install>` as an
unknown element and renders the fallback children normally.

The element reports a coarse asynchronous result:

```js
function handleInstallResult(event) {
  switch (event.result) {
    case "success":
      break;
    case "invalid_data":
      // Correct the manifest or identity information before retrying.
      break;
    case "aborted":
      // The operation did not complete.
      break;
  }
}
```

## Shared processing model

```text
entry point
    |
    +-- current document --> use its processed linked manifest
    |
    `-- explicit target --> fetch manifest without credentials
                              |
                              v
                   process and validate manifest
                              |
                              v
                    validate app identity
                              |
                              v
                   permission and confirmation
                              |
                              v
                 install or mediate existing app
                              |
                              v
                   privacy-limited public result
```

The explicit-target form fetches the manifest directly. It does not load or
execute the target application's document before the user decides whether to
install.

## User consent

Every install attempt requires a user action and browser-controlled
confirmation.

For the imperative explicit-target form, the current draft also defines a
`web-app-installation` permission associated with the requesting origin. For
the declarative form, the browser validates that `<install>` is visible,
unobscured, stably rendered, and activated by the user. The validated click can
serve as the additional intent signal without a separate permission prompt.

The exact relationship between a stored permission denial and a trusted
`<install>` activation remains an open design question in the specification.

## Identity and target authority

The browser processes the target's Web App Manifest and compares its computed
identity with `manifestId` when an assertion is supplied. If no assertion is
supplied, an explicit manifest `id` is required by the current design.

The initial specification requires the manifest URL and processed `start_url`
to be same-origin. This reduces spoofing risk but excludes manifests hosted on
a different CDN origin. Supporting those deployments requires an authority
mechanism that does not make application distribution depend on a privileged
catalog or allowlist.

## Cross-origin distribution and user agency

Cross-origin installation is the main capability that current-document
installation does not provide, but it also creates the proposal's most
important user-agency and privacy questions.

The design does not give target applications an installer allowlist or require
catalog registration. Any site can identify a public target manifest, while the
browser remains responsible for attributing the request, showing trustworthy
target identity, and obtaining the user's decision. Installing one origin does
not grant it permissions or storage belonging to the requesting origin.

Standards feedback has raised concerns about centralization, misleading
catalogs, diminished user control, and cross-origin outcome signals. The
normative draft records those concerns beside the relevant Fetch, consent,
identity, and result algorithms rather than treating cross-origin behavior as
settled.

## Outcomes and privacy

The caller must not learn whether an arbitrary application was already
installed. An already-installed application can be handled through
user-mediated browser UI, such as an offer to launch it, but the public success
result does not distinguish installation from that existing-app action.

`navigator.install()` uses promises and DOM exceptions. `<install>` uses
`InstallResultEvent`. Both intentionally expose coarse outcomes. The exact
boundary between useful developer diagnostics and cross-origin information
leakage is still under discussion.

The element's dimensions, label, styling, DOM state, and accessibility tree
must not vary based on installed state.

## Security and abuse resistance

- Only a fully active top-level document can initiate installation.
- User activation is required.
- Sandboxed documents, fenced frames, and disallowed Permissions Policy
  contexts are blocked.
- Explicit manifest fetches omit credentials.
- Browser UI attributes the requesting origin and the target application.
- Target metadata is untrusted and cannot replace trustworthy origin display.
- `<install>` uses capability-element visibility, occlusion, styling, and
  cooldown protections against clickjacking.
- Browsers can apply additional throttling, reputation, and platform policy.

## Specification dependencies

The integrated specification:

- reuses Web App Manifest concepts and processing;
- references Manifest Incubations only if it needs an incubating concept;
- extends WHATWG HTML with the element and Navigator method;
- reuses Capability Elements activation protections; and
- integrates with Fetch, Permissions, Permissions Policy, Secure Contexts, and
  Launch Handler where applicable.

Any host-spec change is written as an explicit added step, replacement, or
parameterization in the normative draft.

## Major open questions

- What Fetch mode allows credentialless cross-origin manifest processing
  without creating an unintended distribution gate?
- Should permission be checked before or after the target manifest fetch?
- Which manifest fields are minimum cross-browser installation requirements?
- Should an explicit target always require installation permission, including
  a same-origin sibling app?
- Which public outcome distinctions are safe and useful?
- What exactly counts as success for an already-installed application?
- Should same-origin frames or a new sandbox token be supported?
- Where should the reusable Capability Elements core behavior be maintained?
- Should the interfaces be exposed only in secure contexts?

These questions are marked beside the affected normative text in the
[specification](./index.html).
