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
  RFC2817:
  WEBLINKING: RFC8288
  STRUCTURED-FIELD-VALUES: RFC8941
  HTTP-SEMANTICS: RFC9110
  ROBOTSTXT: rfc9309
informative:
  KiB:
    target: https://simple.wikipedia.org/wiki/Kibibyte
    title: KibiByte
    date: 2022-10-14


--- abstract

This document extends RFC9309 by specifying additional URL level controls
through application level header and HTML meta tags originally developed in
1996. Additionally it moves the response header out of the experimental header
space (i.e. "X-") and defines the combinability of multiple headers, which was
previously not possible.


--- middle

# Introduction

While the Robots Exclusion Protocol {{ROBOTSTXT}}enables service owners to
control how, if at all, automated clients known as crawlers may access the URLs
on their services as defined by {{WEBLINKING}}, the protocol doesn't provide
controls on how the data returned by their service may be used upon allowed
access.

Originally developed in 1996 and widely adopted since, the use-case control is
left to URL level controls implemented in the response headers, or in case of
HTML in the form of a meta tag. This document specifies these control tags,
and in case of the response header field, brings it to standards compliance with
{{HTTP-SEMANTICS}}.

Application developers are requested to honor these tags. The tags are not a
form of access authorization however.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This specification uses the following terms from {{STRUCTURED-FIELD-VALUES}}:
Dictionary, List, String, Parameter.

# Specification

## Robots control

The URL level crawler controls are a key-value pair that can be specified two
ways:

* an application level response header structured field as specified by
{{STRUCTURED-FIELD-VALUES}}.
* in case of HTML, one or more meta tags as defined by the HTML specification.

### Application Layer Response Header

The application level response header field `robots-tag` is a structured field
whose value is a dictionary containing list of rules applicable to either all
accessors or specifically named ones. For historical reasons, implementors
should also support the experimental field name, `x-robots-tag`.

The value of the `robots-tag` field is a dictionary containing lists of rules.
The rules are specific to a single product token as defined by {{ROBOTSTXT}} or
a global identifier — "*". The product token is the first element of each list.

Duplicate product tokens must be merged and the rules deduplicated.

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

### HTML meta element

For historical reasons the `robots-tag` header values may be specified by
HTMLservice owners as an HTML meta tag. In case of the meta tag, the name
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

* `noindex` - instructs the parser to not store the served data in its publicly accessible index.
* `nosnippet` - instructs the parser to not reproduce any stored data as an excerpt snippet.

The values are case insensitive. Values unspecified by this document MAY be
ignored, depending on the consumer's requirements.

Implementors may support other rules as specified in Section 2.2.4 of
{{ROBOTSTXT}}.

### Caching of values

The rules specified for a specific product token MUST be obeyed until the rules
have changed. Implementors MAY use standard cache control as defined in
{{HTTP-SEMANTICS}} for caching `robots-tag` rules. Implementors SHOULD refresh
their caches within a reasonable time frame.

# Security Considerations

The `robots-tag` is not a substitute for valid content security measures. To
control access to the URL paths in a `robots.txt` file, users of the protocol
should employ a valid security measure relevant to the application layer on
which the robots.txt file is served — for example, in the case of HTTP,
HTTP Authentication as defined in {{HTTP-SEMANTICS}}.

The content of the `robots-tag` header field is not secure, private or
integrity-guaranteed, and due caution should be exercised when using it. Use of
Transport Layer Security (TLS) with HTTP ({{HTTP-SEMANTICS}}) is currently the
only end-to-end way to provide such protection.

In case of a `robots-tag` specified in a HTML meta element, implementors should
consider only the `meta` elements specified in the head element of the HTML
document, which is generally only accessible to the service owner.

To protect against memory overflow attacks, implementers should enforce a limit
on how much data they will parse; see section N for the lower limit.

# IANA Considerations

```
TODO(illyes):
https://www.rfc-editor.org/rfc/rfc9110.html#name-field-name-registry
```


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
