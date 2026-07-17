# TSN Private View Container

## A Device-Authorized Pixel Rendering Boundary for User-Owned Protocol Data

**Research organization:** TrustLink Labs  
**Protocol:** Transfer Settlement Network (TSN)  
**Status:** Implemented design with documented deployment gaps  
**Reference implementation:** `@trustlink/tsn-sdk/private-view`

## Abstract

Applications commonly receive sensitive protocol data as JSON, framework state, or DOM text. The application can then log, serialize, index, replay, or leak it. TSN Private View changes this boundary: the application declares which protocol-owned field should occupy a location, while the TSN SDK performs authorization, resolution, local decryption, and rendering.

The renderer is a Lit custom element with a closed Shadow DOM. Authorized plaintext is drawn onto an HTML canvas as pixels and is never intentionally inserted into a DOM text node, attribute, slot, accessibility label, hidden input, or React state. Selection is disabled. Copying is an explicit SDK operation that freshly resolves the authorized value and writes directly to the clipboard.

This construction reduces exposure to ordinary DOM inspection, application serialization, framework capture, and casual selection. It does not make a compromised authorized execution environment trustworthy and is not cryptographic protection against malicious code already running with the user's browser privileges.

## 1. Problem statement

TSN integrations need to show user-owned fields without making the integrating platform the data owner. Examples include settlement authority, recovery authority, private transaction references, confidential receipts, and PRU routing material.

The conventional path creates multiple observation points:

```text
Protocol API -> platform backend -> platform JSON -> framework state -> DOM text
```

TSN instead targets a composed SDK rendering boundary:

```text
TSN encrypted data
        -> owner + device authorization
        -> TSN SDK resolver
        -> Lit reactive custom element
        -> closed Shadow DOM
        -> responsive canvas pixels
        -> explicit SDK Copy operation
```

The host application receives identifiers and status, not the revealed field as an ordinary integration value.

## 2. Security objective

The Private View Container is designed so that:

1. an unauthorized device cannot request decryption successfully;
2. authorization is bound to the TIN owner and registered device credential;
3. the integrating application does not receive the revealed field through the component API;
4. revealed text is absent from ordinary DOM text and HTML serialization;
5. copy is deliberate, user-initiated, and SDK mediated;
6. revocation, expiry, replacement, and disconnection return the view to a locked state.

The authorization protocol—not Lit, canvas, or CSS—decides whether decryption is permitted. Lit lifecycle control, canvas, closed Shadow DOM, and selection controls are defense-in-depth after authorization succeeds.

## 3. Why Lit is part of the protection architecture

Lit is not merely a convenience wrapper around canvas. It supplies the standards-based component boundary that makes the renderer reusable, reactive, isolated, and disposable.

### Reactive updates

The TIN, requested field, authorization state, loading state, and copy state are reactive component properties. When relevant inputs change, Lit schedules an update and invokes the component lifecycle. The SDK invalidates the previous resolution, clears the current bitmap, resolves the newly authorized field, renders the canvas, and redraws only after Lit has committed the new element tree. Integrators do not manually inject or replace plaintext DOM.

### Lifecycle-governed setup and teardown

Lit connects the custom element to the browser lifecycle. On connection, the component can begin authorized resolution. On disconnection, TSN increments its resolution generation, clears the canvas bitmap and dimensions, drops component state, and prevents a late asynchronous response from drawing into a removed view. This deterministic cleanup reduces the lifetime of private presentation state. JavaScript and browser garbage collection do not provide a mathematical guarantee that every historical byte is immediately overwritten, so the paper does not claim perfect memory erasure.

### Closed Shadow DOM encapsulation

Lit creates the render root and TSN configures it as closed. The canvas, fallback, and copy control are internal to the custom element. Ordinary host-page selectors cannot traverse that tree, and global CSS selectors cannot restyle its internal nodes. Inherited CSS properties and explicitly exported CSS parts or variables remain intentional integration surfaces; closed Shadow DOM must not be described as protection against malicious same-origin JavaScript.

### Portable custom-element contract

Lit packages the complete rendering lifecycle behind `<tsn-private-value>`. React, Vue, plain HTML, wallet extensions, and future TrustLink Labs applications can mount the same audited SDK element without owning its internal rendering code. This is essential to the TSN guarantee: platforms integrate a protocol component instead of rebuilding a private-data component.

### Responsive presentation

Lit ensures the canvas exists before the SDK obtains its 2D context. The renderer measures the custom element's actual available width, accounts for the Copy control, uses device-pixel-ratio-aware bitmap dimensions, and draws within that measured surface. CSS makes the component follow its host container. A layout change must trigger a safe redraw or a new authorized resolution; it must never move plaintext into ordinary DOM merely to gain responsive behavior.

## 4. Authorization composition

The intended private-session proof combines:

- owner-wallet authorization proving control of the TIN authority;
- a fresh, domain-separated challenge with expiry and replay protection;
- a registered device signing key proving device possession;
- a device encryption-key fingerprint selecting that device's envelope;
- request binding to the TIN, field, action, audience, and session.

The service must reject missing, expired, replayed, cross-audience, cross-TIN, cross-field, or incorrectly signed requests. A wallet signature must not become a permanent bearer token. Device private keys should remain non-exportable where the platform permits it.

## 5. Rendering architecture

```html
<tsn-private-value
  tin="1000000008"
  field="settlementWallet"
  fallback="Privately verified"
></tsn-private-value>
```

The host supplies only privacy-safe identifiers and fallback language. After authorization, the SDK keeps plaintext in a short-lived local variable, draws it with Canvas 2D, and releases its string reference. The displayed result remains pixels in the canvas bitmap.

The component also applies `user-select: none` and `-webkit-user-select: none`. This removes normal selection behavior; it is a user-interface control, not access control.

## 6. Copy protocol

Because canvas pixels are not selectable text, the SDK provides an explicit Copy button:

1. require a user click;
2. freshly resolve the value through the authorized SDK resolver;
3. write directly with `navigator.clipboard.writeText()`;
4. create no temporary input, DOM text, application callback, or plaintext event;
5. discard the local reference after the operation.

Clipboard access requires a secure context and is subject to browser permission and user-activation rules. After copying, the clipboard is outside the TSN boundary and represents a deliberate user disclosure.

## 7. Platform conformance rules

A conforming integration must not:

- request decrypted fields through its application backend;
- pass plaintext as an attribute, property, child, or slot;
- recreate the renderer in application-owned React code;
- capture resolver output in logs, analytics, errors, caches, or global state;
- run DOM/session-replay capture around private views;
- replace owner/device authorization with platform authentication;
- expose an API returning private values to arbitrary application code.

Platforms may position and theme the SDK element through its documented presentation contract. Presentation must not change authorization or reveal plaintext.

## 8. Threat analysis

### Unauthorized requester

Without owner and device proof, decryption fails before rendering.

### DOM inspection and saved HTML

The host contains only the custom element and privacy-safe attributes. The revealed value is not DOM text or an attribute, so ordinary serialization does not contain it.

### Framework capture

The field is not returned as a React property or stored in application state, reducing accidental exposure through framework tooling.

### Selection and casual copying

Canvas has no selectable text and the element disables selection. Copy requires the SDK control.

### Malicious script in the authorized page

Arbitrary code with equivalent browser privileges may instrument the resolver, Canvas, Clipboard, `attachShadow()`, or runtime/network primitives. It may read pixels if it obtains the canvas reference. This design cannot make hostile same-origin execution safe.

### Extensions, capture, and OS compromise

Privileged extensions, screenshots, screen readers, remote-control software, malware, graphics capture, and physical cameras may observe what the authorized user sees.

### Compromised authorities

If the required wallet and device authorities are compromised, an attacker may satisfy the policy. Revocation, recovery, wallet security, and device security remain necessary.

## 9. Claims and non-claims

Supported claims:

- correctly implemented authorization blocks unauthorized devices;
- integrations need no plaintext-returning component API;
- revealed text is absent from intended DOM and serialization paths;
- closed Shadow DOM reduces ordinary application access;
- canvas and selection controls reduce casual extraction;
- copy is explicit and SDK mediated.

Prohibited claims:

- “canvas encryption”;
- “impossible to inspect”;
- protection from all extensions, malware, screenshots, or compromised browsers;
- absolute platform blindness while the platform can execute arbitrary hostile code in the authorized page;
- production completeness while TSN authorization services remain application-hosted.

## 10. Implementation status

Implemented:

- TSN SDK-owned Lit element and closed Shadow DOM;
- automatic authorized resolution;
- canvas pixel rendering;
- privacy-safe fallback and ARIA description;
- selection disabled;
- fresh-resolution Copy control;
- bitmap cleanup and outstanding-resolution invalidation.

Remaining:

- move challenge, device-registry, and private-session adapters to a TSN-owned service boundary;
- remove legacy plaintext authority fields from application APIs;
- complete CSP, dependency-integrity, revocation, session-replay exclusion, and adversarial browser tests;
- design an opt-in accessible reveal mode with an explicit disclosure model;
- publish third-party integration conformance tests.

## 11. Conformance tests

- `element.shadowRoot` is `null`.
- Host and internal DOM contain no private plaintext node or attribute.
- Default HTML serialization contains no revealed value.
- Canvas pixels appear only after valid authorization.
- Unauthorized, expired, revoked, wrong-audience, and replayed requests remain locked.
- Selection cannot reveal the value.
- Copy freshly resolves and creates no temporary plaintext DOM.
- Disconnect clears canvas dimensions and invalidates pending work.
- Logs, analytics, errors, and application state contain no revealed value.

## 12. Repository and publication model

TrustLink Pay is the reference integration workspace; it does not own TSN research or protocol architecture. Canonical publications belong in the independent `Trustlink-Research` repository. TSN SDK, TIN SDK, Gateway, Veil, and other protocol deliverables should likewise have independent TrustLink Labs repositories and release histories.

The integrated workspace may consume them as versioned packages or Git submodules. It may keep a synchronized documentation copy for local development, but the research repository remains canonical. Independent projects should not be flattened into ignored folders with no recorded revision: the parent must record an exact package version, commit, or submodule revision for reproducibility.

Canonical library: <https://github.com/Trustlink-Labs/Trustlink-Research>

## References

- [Lit Shadow DOM](https://lit.dev/docs/components/shadow-dom/)
- [Lit lifecycle](https://lit.dev/docs/components/lifecycle/)
- [MDN: `Element.attachShadow()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/attachShadow)
- [HTML Standard: canvas](https://html.spec.whatwg.org/multipage/canvas.html)
- [MDN: Canvas `getImageData()`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/getImageData)
- [MDN: Clipboard `writeText()`](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard/writeText)
- [MDN: CSS `user-select`](https://developer.mozilla.org/en-US/docs/Web/CSS/user-select)
- [MDN: Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
