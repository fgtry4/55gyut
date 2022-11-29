---
# Every document starts with a front matter in YAML enclosed by triple dashes.
# See https://jekyllrb.com/docs/front-matter/ to learn more about this concept.
caip: <to be assigned>
title: Web extension provider communication
author: Mark Stacey (@Gudahtt)
discussions-to: <URL>
status: Draft
type: Standard
created: 2022-11-28
---

<!--You can leave these HTML comments in your merged EIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new EIPs. Note that an EIP number will be assigned by an editor. When opening a pull request to submit your EIP, please use an abbreviated title in the filename, `eip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the CAIP.-->
This CAIP establishes a standard way for JSON-RPC providers to communicate with web extensions.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
This CAIP establishes how providers can communicate with web extensions. Two overall strategies are described, one for `externally_connectable` web extensions, and one for `contentscript` web extensions.

## Motivation
<!--The motivation is critical for CAIP. It should clearly explain why the state of the art is inadequate to address the problem that the CAIP solves. CAIP submissions without sufficient motivation may be rejected outright.-->
Web extensions today will typically inject a JavaScript provider API into websites as a global variable. For example, in the Ethereum ecosystem this provider API is standardized in EIP-1193, and is conventionally injected as `window.ethereum`.

This injected API strategy has some advantages:
* For websites developers, using a global variable is simple and requires no effort on their part to setup.
* It allows websites to support any web extension following this standard with no additional effort.

However, the injected API strategy has many disadvantages:
* It depends upon the web extension having read and write access to every website, which is a scary permission that web extension authors might otherwise be able to avoid asking for.
* It slows down every website by injecting additional code to be parsed and executed. It even slows down the initial page load in most cases, because many web extensions inject the provider synchronously to maintain compatibility with websites that expect it to be available immediately.
* Injecting code into a website can break it.
* It provides no way for web extension authors to safely make breaking changes to their provider API, resulting in even more code being injected to support legacy APIs.

An alternative strategy would be for a website to embed its own provider. A provider could be offered as a library, to be embedded by the website author. This strategy can address all disadvantages of the injected provider approach:
* If this strategy became widespread enough, it would allow some web extensions to stop asking for write access to all pages.
* The website author can control when the provider is initialized, and fine-tune performance.
* No code needs to be injected, so websites would no longer be broken by injected code.
* The provider library can be published with breaking changes, allowing changes to the API without needing to embed legacy code in each website.

A provider library can be similarly easy to use for website authors as well, requiring nothing more than a single script tag to import a library and get an equivalent experience to using an the injected provider.

Web extension inter-operability is a challenge for embeded providers though. That is what this proposal means to address. Today there is no way to write a provider such that it is compatible with any web extension. Web extensions differ today in how they communicate with wallets, from the messaging system used to the messaging format. These details often aren't publicly documented or treated as a public-facing API, so they can change without notice, making it risky even to embed support for popular conventions used today.

A standard method for providers to communicate with web extebsions would allow website authors to embed their own providers without losing web extension inter-operability.

## Specification
<!--The technical specification should describe the standard in detail. The specification should be detailed enough to allow competing, interoperable implementations. -->

### Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" written in
uppercase in this document are to be interpreted as described in [RFC
2119](https://www.ietf.org/rfc/rfc2119.txt)

### Summary

Two strategies are described below; one for `externally_connectable` web extensions, and one for `contentscript` web extensions. Each strategy is intended to work with a different web extension permission.

A web extension MUST support at least one of these strategies. A web extension MAY support multiple strategies.

Both strategies use the same message format.

### Message format

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "/caip-x/schemas/provider-request.schema.json",
  "title": "Provider / Web Extension message",
  "description": "A request sent between a provider and a web extension.",
  "type": "object",
  "properties": {
    "type": {
      "description": "The message type, used to identify this as a CAIP-X message",
      "const": "caip-x"
    },
    "data": {
      "description": "A JSON-RPC message",
      "type": "object"
    }
  },
  "required": ["type", "data"]
}
```

### `externally_connectable` web extensions

A web extension can use the `externally_connectable` manifest field to accept messages from websites. This permission can be configured to limit which sites can message the web extension. How this permission is configured is out-of-scope for this proposal; this proposal only concerns sites that the web extension allows messages from.

A web extension can receive messages using `browser.runtime.onMessage`. Incoming messages should be validated according to the


The `target` field MUST be unset for all messages from either the provider or the web extension.

The provider should send messages using `runtime.sendMessage`.

The web extension should recieve messages using `runtime.onMessage`.


### `contentscript` web extensions

The provider should send messages using the format


## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->


## Backwards Compatibility
<!--All CAIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The CAIP must explain how the author proposes to deal with these incompatibilities. CAIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->

## Test Cases
<!--Please add test cases here if applicable.-->

## Links
<!--Links to external resources that help understanding the CAIP better. This can e.g. be links to existing implementations.-->

## Copyright
Copyright and related rights waived via [CC0](../LICENSE).