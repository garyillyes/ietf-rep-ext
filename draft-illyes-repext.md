---
title: "Robots Exclusion Protocol Extension for URL Level Control"
abbrev: "REPext for URL level"
category: info

docname: draft-illyes-repext-latest
submissiontype: independent  # also: "independent", "editorial", "IAB", or "
number:
date:
v: 4
keyword:
 - robots.txt

author:
 -
    fullname: "Gary Illyes"
    organization: Google LLC.
    street: "Brandschenkestrasse 110"
    city: "Zürich"
    code: "8002"
    country: "Switzerland"
    email: "garyillyes@google.com"
normative:
  HTTP-CACHING: rfc9111
  HTTP-SEMANTICS: rfc9110
  ROBOTSTXT: rfc9309
  STRUCTURED-FIELD-VALUES: rfc8941
  TLS: rfc8446
  WEBLINKING: rfc8288
  X-DEPRECATION: rfc6648
informative:
  KiB:
    target: https://simple.wikipedia.org/wiki/Kibibyte
    title: KibiByte
    date: 2022-10-14
  HTML-META:
    target: https://html.spec.whatwg.org/multipage/semantics.html#the-meta-element
    title: HTML Meta Element
    date: 2026-04-14




--- abstract

This document extends RFC9309 by specifying additional URL level controls
through an HTTP response header and, foir historical reasons, through HTML meta
tags originally developed in 1996. Additionally it moves the HTTP response
header out of the experimental header space (i.e., "X-") and defines the
combinability of multiple headers, which was previously not possible.


--- middle

# Introduction

While the Robots Exclusion Protocol {{ROBOTSTXT}} enables service owners to
control how, if at all, automated clients known as crawlers may access the URLs
on their services as defined by {{WEBLINKING}}, the protocol doesn't provide
controls on how the data returned by their service may be used upon allowed
access.

Originally developed in 1996 and widely adopted since, the use-case control is
left to URL level controls implemented in the response headers, or in case of
HTML in the form of a meta tag as defined by {{HTML-META}}. This document
specifies these control tags, and in case of the response header field, brings
it to standards compliance with {{HTTP-SEMANTICS}}.

Application developers are requested to honor these tags. The tags are not a
form of access authorization however.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification uses the following terms from {{STRUCTURED-FIELD-VALUES}}:
List, String, Parameter.

# Specification

## Robots control

The URL level crawler controls are a key-value pair that can be specified two
ways:

* an HTTP response header structured field as specified by
{{STRUCTURED-FIELD-VALUES}}.
* for hiostorical reasons, in case of HTML, one or more meta tags as defined by
the {{HTML-META}} specification.

### HTTP Response Header

The robots-tag field is a List as defined in {{STRUCTURED-FIELD-VALUES}}. Each
member of the List is an Item representing a product token. Rules applicable to
a product token are defined as Parameters of that Item. For historical reasons,
implementors SHOULD also support the experimental field name `x-robots-tag`.

The product token is either a specific string as defined in
Section 2.2.1 of {{ROBOTSTXT}} or the global identifier `*`. If a product token
appears multiple times in the field, the Parameters from all instances MUST be
merged and deduplicated.

For example, the following response header field specifies `noindex` and
`nosnippet` rules for all accessors, however specifies no rules for the product
token `ExampleBot`:


~~~~~~~~
Robots-Tag: *;noindex;nosnippet, ExampleBot;
~~~~~~~~


The structured field in the examples is deserialized into the following objects:


~~~~~~~~
"*" = [
       ["noindex", true],
       ["nosnippet", true]
      ],
"ExampleBot" = []
~~~~~~~~

Implementors SHOULD impose a parsing limit on the field value to protect their
systems. The parsing limit MUST be at least 8 kibibytes [KiB].

### HTML Meta Element

For historical reasons the `robots-tag` header values may be specified by
HTML service owners as an HTML meta tag. In case of the meta tag, the name
attribute is used to specify the product token, and the content attribute to
specify the comma separated robots-tag rules.

As with the header, the product token may be a global token, `robots`, which
signifies that the rules apply to all requestors, or a specific product token
applicable to a single requestor. For example:

~~~~~~~~
<meta name="robots" content="noindex">
<meta name="examplebot" content="nosnippet">
~~~~~~~~

Multiple robots meta elements may appear in a single HTML document. Requestors
must obey the sum of negative rules specific to their product token and the
global product token.

### Robots controls rules

The possible values of the rules are:

* `noindex` - instructs the parser to not store the served data in its publicly
accessible index.
* `nosnippet` - instructs the parser to not reproduce any stored data as an
excerpt snippet.

The values are case insensitive.

Implementors may support other rules as specified in Section 2.2.4 of
{{ROBOTSTXT}}.

### Caching of values

Implementors SHOULD link the validity of robots-tag rules to the freshness of
the associated resource. Rules extracted from an HTTP response header or an HTML
`meta` tag remain in effect until the resource is recrawled or the cached
representation expires.

Implementors SHOULD determine the freshness of the rules using standard HTTP
cache directives, such as `Cache-Control` or `Expires`, as defined in
{{HTTP-SEMANTICS}}. In the absence of explicit cache directives, implementors
MAY use heuristics to determine the refresh interval, typically matching the
scheduled recrawl frequency of the resource.

If a resource is not recrawled due to a lack of perceived change, the last known
`robots-tag` rules MUST continue to be honored.

# Security Considerations

The `robots-tag` is not a substitute for valid content security measures. To
control access to the URL paths where the `robots-tag` appears, service owners
SHOULD employ a valid security measure such as HTTP Authentication as defined in
{{HTTP-SEMANTICS}}.

The content of the `robots-tag` header field is not secure, private or
integrity-guaranteed, and due caution should be exercised when using it. Use of
Transport Layer Security ({{TLS}}) with HTTP ({{HTTP-SEMANTICS}}) is currently
the only end-to-end way to provide such protection.

In case of a `robots-tag` specified in a HTML meta element, implementors should
consider only the `meta` elements specified in the head element of the HTML
document, which is generally only accessible to the service owner.

To protect against memory overflow attacks, implementers should enforce a limit
on how much data they will parse; see section N for the lower limit.

# IANA Considerations

## HTTP Field Name Registration

IANA is requested to register the following HTTP field name in the
"Hypertext Transfer Protocol (HTTP) Field Name Registry" according to the
procedures defined in Section 18.4 of {{HTTP-SEMANTICS}}:

Field Name: Robots-Tag

Template: None

Status: permanent

Reference: [This document]

Comments: This field name supersedes the experimental X-Robots-Tag field.


## Deprecation of `X-Robots-Tag`

The `X-Robots-Tag` field name was used prior to the standardization of the
`Robots-Tag` field. New implementations MUST NOT use the `X-` prefix for this
field, adhering to the principles in {{X-DEPRECATION}}. While parsers SHOULD
continue to support `X-Robots-Tag` for backward compatibility with legacy
systems, it is formally deprecated by this document.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
