![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DK Hostmaster RDAP Service Specification

2017/03/15
Revision: 1.0 ~ *DRAFT*

**PLEASE NOTE THAT THIS SERVICE IS CURRENTLY A PROTOTYPE**

# Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3, 4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
  - [Localization](#localization)
  - [Media types / Format](#media-types--format)
  - [Encoding](#encoding)
  - [Rate Limiting](#rate-limiting)
  - [Security](#security)
  - [Authentication](#authentication)
  - [Event Actors](#event-actors)
  - [Non-supported APIs](#non-supported-apis)
- [Service](#service)
  - [domain](#domain)
    - [API](#api)
    - [Example](#example)
  - [entity](#entity)
  - [Limitations to available entity data](#limitations-to-available-entity-data)
    - [API](#api-1)
    - [Example](#example-1)
  - [name server](#name-server)
    - [API](#api-2)
    - [Example](#example-2)
- [References](#references)
  - [RDAP](#rdap)
  - [vCard and jCard](#vcard-and-jcard)
- [Resources](#resources)
  - [Mailing list](#mailing-list)
  - [Issue Reporting](#issue-reporting)
  - [Additional Information](#additional-information)
- [Appendices](#appendices)
  - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
# Introduction

This document describes the RDAP service offered by DK Hostmaster A/S. It is primarily aimed at a technical audience and the reader is required to have knowledge of the RDAP protocol.

The RDAP service implementation by DK Hostmaster does not implement all features outlined in the related RFC, this document describes primarily the available features and possible limitations to these and lists the unsupported features.

Please note that all provided data are provided under the disclaimer, included in the response in the `notices` section.

<a id="about-this-document"></a>
# About this Document

This specification describes version 1 (1.0.x) of the DK Hostmaster RDAP service implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster RDAP service implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/rdap-service-specification/blob/master/README.md), using the **gitprint** service.

<a id="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
## Document History

- 1.0 2017-03-14 ~ *DRAFT*
  - Initial revision

<a id="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

<a id="features"></a>
# Features

The RDAP service offers the following features.

- Domain name inquiry
- Name server inquiry
- Entity inquiry
- Support for multiple encodings (see: [Encoding](#encoding) below)
- Support for both IPv6 and IPv6

<a id="implementation-limitations"></a>
# Implementation Limitations

This section describes the common limitations of the RDAP service,

For limitations to specific capabilities please see the designated sections.

<a id="localization"></a>
## Localization

In general the service is not localized and all information is provided in English. Address data however represented in the format suitable for use in the DK Hostmaster registry, meaning addresses in Denmark are represented in the local format and addresses based outside Denmark in the international representation if available.

For more information on this please refer to the section on contact creation in the [EPP service specification](https://github.com/DK-Hostmaster/epp-service-specification#create-contact).

<a id="media-types--format"></a>
## Media types / Format

The service supports the following media types:

- `application/json` for JSON
- `application/rdap+json` also for JSON

Not using any of these media-types, results in the HTTP error code `415`, "Unsupported media type".

<a id="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8, being the primary encoding used
- Punycode [RFC:3492][RFC:3492], please see the specific APIs for information on use of this encoding. This is solely used representing IDN domain names and name server names under IDN domain names
- URI encoding [RFC:3986][RFC:3986], this is not a part of the RDAP standard, but it is supported for transparency and client support

<a id="rate-limiting"></a>
## Rate Limiting

Currently no rate limiting is implemented

We reserve the right to adjust/enforce the rate limiting in order ensure quality of service as an operational measure.

<a id="security"></a>
## Security

The service is only available under: TLS 1.2

<a id="authentication"></a>
## Authentication

Authentication is currently unsupported and hereby authorization, so only public available data are presented.

<a id="event-actors"></a>
## Event Actors

Event actors are not currently disclosed.

<a id="non-supported-apis"></a>
## Non-supported APIs

The following RDAP capabilities are unsupported by DK Hostmaster.

- AS query
- IP query
- Domain search
- Entity search
- Name server search

<a id="service"></a>
# Service

The service is available at:

- `https://rdap.dk-hostmaster.dk`

The service offers four APIs, one specialized for each entity type:

- domain (domain name)
- name server (hostname/name server)
- entity (userid)
- help

The service requires that the `Accept` header is specified to be `application/json` or `application/rdap+json`, all of the below examples demonstrates this using the command line utilities `curl` or `httpie`.

If the header is unspecified, not specified correctly or specified to an unsupported format, the service will error with HTTP status code: `415` -"Unsupported media type".

```bash
$ curl https://rdap.dk-hostmaster.dk/entity/DKHM1-DK
"Unsupported Media Type"
```

Correct specification using `curl` should read as follows:

```bash
> curl --header "Accept: application/json" https://rdap.dk-hostmaster.dk/entity/DKHM1-DK
```

And for `httpie`:

```bash
$ http https://rdap.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'
```

<a id="domain"></a>
## domain

This service returns data on a given domain name according to the RDAP specification, please see the limitations on entity data, which might apply to the entities listed as part of the response to a domain query request.

The domain name can be specified in UTF-8 or punycode (ASCII).

The status of the domain response is currently limited to:

- `active`
- `inactive`
- `pending delete`

These might be extended in the future.

<a id="api"></a>
### API

```
https://rdap.dk-hostmaster.dk/domain/{domain name}
```

The service returns `200` if the relevant object is available.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example"></a>
### Example

Using `httpie`

```bash
$ http https://rdap.dk-hostmaster.dk/domain/eksempel.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://rdap.dk-hostmaster.dk/domain/eksempel.dk | jq
```

```json
{
    "entities": [
        {
            "events": [
                {
                    "eventAction": "registration",
                    "eventDate": "2017-03-10 20:06:29"
                },
                {
                    "eventAction": "last changed",
                    "eventActor": "n/a",
                    "eventDate": "2017-03-10 20:06:29"
                }
            ],
            "handle": "DKHM666-DK",
            "links": [
                {
                    "href": "https://rdap.dk-hostmaster.dk/entity/DKHM666-DK",
                    "rel": "self",
                    "type": "application/rdap+json",
                    "value": "https://rdap.dk-hostmaster.dk/entity/DKHM666-DK"
                }
            ],
            "objectClassName": "domain",
            "remarks": [
                {
                    "description": [
                        "The official DK Hostmaster A/S RDAP service specification is available at: https://github.com/DK-Hostmaster/rdap-service-specification"
                    ],
                    "title": "Official Service Specification"
                }
            ],
            "roles": [
                "proxy"
            ],
            "vcardArray": [
                "vcard",
                {
                    "adr": [
                        {
                            "country-name": [
                                "Danmark"
                            ],
                            "extended-address": [
                                ""
                            ],
                            "locality": [
                                "København V"
                            ],
                            "post-office-box": [
                                ""
                            ],
                            "postal-code": [
                                "1560"
                            ],
                            "region": [
                                ""
                            ],
                            "street-address": [
                                "Kalvebod Brygge 45,3"
                            ]
                        }
                    ],
                    "fn": "DK Hostmaster A/S"
                }
            ]
        },
        {
            "events": [
                {
                    "eventAction": "registration",
                    "eventDate": "2017-03-10 20:06:29"
                },
                {
                    "eventAction": "last changed",
                    "eventActor": "n/a",
                    "eventDate": "2017-03-10 20:06:29"
                }
            ],
            "handle": "DKHM1-DK",
            "links": [
                {
                    "href": "https://rdap.dk-hostmaster.dk/entity/DKHM1-DK",
                    "rel": "self",
                    "type": "application/rdap+json",
                    "value": "https://rdap.dk-hostmaster.dk/entity/DKHM1-DK"
                }
            ],
            "objectClassName": "domain",
            "remarks": [
                {
                    "description": [
                        "The official DK Hostmaster A/S RDAP service specification is available at: https://github.com/DK-Hostmaster/rdap-service-specification"
                    ],
                    "title": "Official Service Specification"
                }
            ],
            "roles": [
                "registrant"
            ],
            "vcardArray": [
                "vcard",
                {
                    "adr": [
                        {
                            "country-name": [
                                "Danmark"
                            ],
                            "extended-address": [
                                ""
                            ],
                            "locality": [
                                "København V"
                            ],
                            "post-office-box": [
                                ""
                            ],
                            "postal-code": [
                                "1560"
                            ],
                            "region": [
                                ""
                            ],
                            "street-address": [
                                "Kalvebod Brygge 45,3"
                            ]
                        }
                    ],
                    "fn": "DK Hostmaster A/S"
                }
            ]
        }
    ],
    "events": [
        {
            "eventAction": "registration",
            "eventDate": "2016-09-14"
        },
        {
            "eventAction": "last changed",
            "eventActor": "DKHM1-DK",
            "eventDate": "2017-03-10 20:06:43"
        }
    ],
    "handle": "eksempel.dk",
    "ldhName": "eksempel.dk",
    "links": [
        {
            "href": "https://rdap.dk-hostmaster.dk/domain/eksempel.dk",
            "rel": "self",
            "type": "application/rdap+json",
            "value": "https://rdap.dk-hostmaster.dk/domain/eksempel.dk"
        }
    ],
    "nameservers": [
        {
            "handle": "auth01.ns.dk-hostmaster.dk",
            "ldhName": "auth01.ns.dk-hostmaster.dk",
            "objectClassName": "nameserver"
        },
        {
            "handle": "auth02.ns.dk-hostmaster.dk",
            "ldhName": "auth02.ns.dk-hostmaster.dk",
            "objectClassName": "nameserver"
        }
    ],
    "notices": [
        {
            "description": [
                "The data in the DK Whois database is provided by DK Hostmaster A/S\nfor information purposes only, and to assist persons in obtaining\ninformation about or related to a domain name registration record.\nWe do not guarantee its accuracy. We will reserve the right to remove\naccess for entities abusing the data, without notice. \n\nAny use of this material to target advertising or similar activities\nare explicitly forbidden and will be prosecuted. DK Hostmaster A/S\nrequests to be notified of any such activities or suspicions thereof."
            ],
            "title": "Disclaimer"
        }
    ],
    "objectClassName": "domain",
    "port43": "whois.dk-hostmaster.dk",
    "rdapConformance": [
        "rdap_level_0"
    ],
    "remarks": [
        {
            "description": [
                "The official DK Hostmaster A/S RDAP service specification is available at: https://github.com/DK-Hostmaster/rdap-service-specification"
            ],
            "title": "Official Service Specification"
        }
    ],
    "secureDNS": {
        "delegationSigned": false
    },
    "status": [
        "active"
    ],
    "unicodeName": "eksempel.dk"
}
```

<a id="entity"></a>
## entity

This service returns data on a given handle/user-id.

This service returns data on a given entity according to the RDAP specification, please see the limitations on entity data, which might apply to the entities included in the response to a entity query request.

The handle is identified by a handle/user-id, the handle holds the format:

- ASCII letters and numbers in upper-case
- postfixed with the string `-DK`

An example: DKHM1-DK

The status of the entity response is currently limited to:

- `active`
- `validated`
- `associated`
- `pending delete`

<a id="limitations-to-available-entity-data"></a>
## Limitations to available entity data

As part of the privacy policy, the following data are not necessarily available:

- Name, address and user for users with address protection enabled
- Email for all users
- Phone numbers marked as non-public

<a id="api-1"></a>
### API

```
https://rdap.dk-hostmaster.dk/handle/{user-id}
```

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example-1"></a>
### Example

Using `httpie`

```bash
$ http https://rdap.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl --header "Accept: application/json" https://rdap.dk-hostmaster.dk/handle/DKHM1-DK | jq
```

```json
{
    "events": [
        {
            "eventAction": "registration",
            "eventDate": "2017-03-10 20:06:29"
        },
        {
            "eventAction": "last changed",
            "eventActor": "n/a",
            "eventDate": "2017-03-10 20:06:29"
        }
    ],
    "handle": "DKHM1-DK",
    "links": [
        {
            "href": "https://rdap.dk-hostmaster.dk/entity/DKHM1-DK",
            "rel": "self",
            "type": "application/rdap+json",
            "value": "https://rdap.dk-hostmaster.dk/entity/DKHM1-DK"
        }
    ],
    "notices": [
        {
            "description": [
                "The data in the DK Whois database is provided by DK Hostmaster A/S\nfor information purposes only, and to assist persons in obtaining\ninformation about or related to a domain name registration record.\nWe do not guarantee its accuracy. We will reserve the right to remove\naccess for entities abusing the data, without notice. \n\nAny use of this material to target advertising or similar activities\nare explicitly forbidden and will be prosecuted. DK Hostmaster A/S\nrequests to be notified of any such activities or suspicions thereof."
            ],
            "title": "Disclaimer"
        }
    ],
    "objectClassName": "entity",
    "port43": "whois.dk-hostmaster.dk",
    "rdapConformance": [
        "rdap_level_0"
    ],
    "remarks": [
        {
            "description": [
                "The official DK Hostmaster A/S RDAP service specification is available at: https://github.com/DK-Hostmaster/rdap-service-specification"
            ],
            "title": "Official Service Specification"
        }
    ],
    "status": [
        "validated"
    ],
    "vcardArray": [
        "vcard",
        {
            "adr": [
                {
                    "country-name": [
                        "Danmark"
                    ],
                    "extended-address": [
                        ""
                    ],
                    "locality": [
                        "København V"
                    ],
                    "post-office-box": [
                        ""
                    ],
                    "postal-code": [
                        "1560"
                    ],
                    "region": [
                        ""
                    ],
                    "street-address": [
                        "Kalvebod Brygge 45,3"
                    ]
                }
            ],
            "fn": "DK Hostmaster A/S"
        }
    ]
}
```

<a id="name-server"></a>
## name server

This service returns data on a given .

This service returns data on a given host name/name server according to the RDAP specification.

The name server name can be specified in UTF-8 or punycode (ASCII).

<a id="api-2"></a>
### API

```
https://rdap.dk-hostmaster.dk/nameserver/{hostname}
```

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a id="example-2"></a>
### Example

Using `httpie`

```bash
$ http https://rdap.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk Accept:'application/json'
```

Using `curl` and `jq`

```bash
$ curl https://rdap.dk-hostmaster.dk/host/auth01.ns.dk-hostmaster.dk | jq
```

```json
{
    "events": [
        {
            "eventAction": "registration",
            "eventDate": "2013-01-30 10:33:00"
        },
        {
            "eventAction": "last changed",
            "eventActor": "n/a",
            "eventDate": "2013-01-30 10:33:00"
        }
    ],
    "handle": "auth01.ns.dk-hostmaster.dk",
    "ipAddresses": {
        "v4": [
            "193.163.102.11"
        ],
        "v6": [
            "2a01:630:0:40::11"
        ]
    },
    "ldhName": "auth01.ns.dk-hostmaster.dk",
    "links": [
        {
            "href": "https://rdap.dk-hostmaster.dk/nameserver/auth01.ns.dk-hostmaster.dk",
            "rel": "self",
            "type": "application/rdap+json",
            "value": "https://rdap.dk-hostmaster.dk/nameserver/auth01.ns.dk-hostmaster.dk"
        }
    ],
    "notices": [
        {
            "description": [
                "The data in the DK Whois database is provided by DK Hostmaster A/S\nfor information purposes only, and to assist persons in obtaining\ninformation about or related to a domain name registration record.\nWe do not guarantee its accuracy. We will reserve the right to remove\naccess for entities abusing the data, without notice. \n\nAny use of this material to target advertising or similar activities\nare explicitly forbidden and will be prosecuted. DK Hostmaster A/S\nrequests to be notified of any such activities or suspicions thereof."
            ],
            "title": "Disclaimer"
        }
    ],
    "objectClassName": "nameserver",
    "port43": "whois.dk-hostmaster.dk",
    "rdapConformance": [
        "rdap_level_0"
    ],
    "remarks": [
        {
            "description": [
                "The official DK Hostmaster A/S RDAP service specification is available at: https://github.com/DK-Hostmaster/rdap-service-specification"
            ],
            "title": "Official Service Specification"
        }
    ],
    "status": [
        "active"
    ],
    "unicodeName": "auth01.ns.dk-hostmaster.dk"
}
```

<a id="references"></a>
# References

- [Basic RDAP Demo Client](https://github.com/DK-Hostmaster/rdap-demo-client-mojolicious)
- [Command line RDAP client](https://github.com/registrobr/rdap-client) implementation in Go

<a id="rdap"></a>
## RDAP

- [RFC:7480 HTTP Usage in the Registration Data Access Protocol (RDAP)](https://tools.ietf.org/html/rfc7480)
- [RFC:7482 Registration Data Access Protocol (RDAP) Query Format](https://tools.ietf.org/html/rfc7482)
- [RFC:7483 JSON Responses for the Registration Data Access Protocol (RDAP)](https://tools.ietf.org/html/rfc7483)
- [RFC:7484 Finding the Authoritative Registration Data (RDAP) Service](https://tools.ietf.org/html/rfc7484)

<a id="vcard-and-jcard"></a>
## vCard and jCard

- [RFC:6350 vCard Format Specification](https://tools.ietf.org/html/rfc6350)
- [RFC:7095 jCard: The JSON Format for vCard](https://tools.ietf.org/html/rfc7095)

<a id="resources"></a>
# Resources

Resources for DK Hostmaster RDAP support can be found below.

<a id="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster RDAP service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

- `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a id="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the RDAP implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

- `info@dk-hostmaster.dk`

<a id="additional-information"></a>
## Additional Information

The DK Hostmaster website service page

- `https://www.dk-hostmaster.dk/en/rdap`

<a id="appendices"></a>
# Appendices

<a id="http-status-codes"></a>
## HTTP Status Codes

The services hold their own table of return codes, this is just a curated list to collect all possible return codes, please refer to the specific services also.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |
| `500` | Internal server error |

[RFC:7480]: https://tools.ietf.org/html/rfc7480
[RFC:7483]: https://tools.ietf.org/html/rfc7483
[RFC:7484]: https://tools.ietf.org/html/rfc7484

[RFC:5890]: https://tools.ietf.org/html/rfc5890
[RFC:6350]: https://tools.ietf.org/html/rfc6350
[RFC:7095]: https://tools.ietf.org/html/rfc7095
