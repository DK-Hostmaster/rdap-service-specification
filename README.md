# DK Hostmaster RDAP Service Specification

2017/03/15
Revision: 1.0 ~ *DRAFT*

**PLEASE NOTE THAT THIS SERVICE IS CURRENTLY A PROTOTYPE**

# Table of Contents

<!-- MarkdownTOC depth=3 -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
    - [License](#license)
    - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
    - [Localization](#localization)
    - [Media-type / Format](#media-type--format)
    - [Encoding](#encoding)
    - [Rate Limiting](#rate-limiting)
    - [Security](#security)
    - [Authentication](#authentication)
    - [Non-supported APIs](#non-supported-apis)
- [Service](#service)
    - [domain](#domain)
        - [API](#api)
        - [Example](#example)
    - [entity](#entity)
        - [API](#api-1)
        - [Example](#example-1)
    - [nameserver](#nameserver)
        - [API](#api-2)
        - [Example](#example-2)
- [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)
- [Appendices](#appendices)
    - [HTTP Status Codes](#http-status-codes)

<!-- /MarkdownTOC -->

<a name="introduction"></a>
# Introduction

This document describes the RDAP service offered by DK Hostmaster A/S. It is primarily aimed at a technical audience and the reader is required to have knowledge of the RDAP protocol.

<a name="about-this-document"></a>
# About this Document

This specification describes version 1 (1.0.x) of the DK Hostmaster RDAP service implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster RDAP service implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/rdap-service-specification/blob/master/README.md), using the gitprint service.

<a name="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a name="document-history"></a>
## Document History

* 1.0 2017-03-14 ~ *DRAFT*
  * Initial revision

<a name="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

<a name="features"></a>
# Features

The RDAP service offers the following features.

- Domain name inquiry
- Nameserver inquiry
- Entity inquiry
- Support for multiple encodings (see: [Encoding](#encoding) below)
- Support for both IPv6 and IPv6

<a name="implementation-limitations"></a>
# Implementation Limitations

<a name="localization"></a>
## Localization

In general the service is not localized and all information is provided in English, address data however represented in a format suitable for use in the DK Hostmaster registry, meaning addresses in Denmark are represented in the local format and addresses based outside Denmark in the international representation if available.

For more information on this please refer to the section on contact creation in the [EPP service specification](https://github.com/DK-Hostmaster/epp-service-specification#create-contact).

<a name="media-type--format"></a>
## Media-type / Format

The service supports the following media-types: 

- `application/json` for JSON
- `application/rdap+json` also for JSON

<a name="encoding"></a>
## Encoding

The service supports the following encodings:

- UTF-8, being the primary encoding used
- Punycode [RFC:3492][RFC:3492], please see the specific APIs for information on use of this encoding. This is solely used representing IDN domain names and nameserver names under IDN domain names
- URI encoding [RFC:3986][RFC:3986], this is not a part of the RDAP standard, but it is supported for transparency and client support

<a name="rate-limiting"></a>
## Rate Limiting

Currently no rate limiting is implemented

We reserve the right to adjust the rate limit in order ensure quality of service. 

<a name="security"></a>
## Security

The service is only available under **TLS 1.2**

<a name="authentication"></a>
## Authentication

Authentication is currently unsupported and hereby authorization, so only public available data are presented.

<a name="non-supported-apis"></a>
## Non-supported APIs

The following RDAP capabilities are unsupported by DK Hostmaster.

- AS query
- IP query
- Domain search
- Entity search
- Nameserver search

<a name="service"></a>
# Service

The service is available at:

- `https://rdap.dk-hostmaster.dk`

The service offers four APIs, one specialized for each entity type:

- domain (domainname)
- nameserver (hostname/nameserver)
- entity (userid)
- help

The service requires that the `Accept` header is specified to be `application/json` or `application/rdap+json`, all of the below examples demonstrates this using the commandline utilities `curl` or `httpie`.

If the header is unspecified, not specified correctly or specified to an unsupported format, the service will error with HTTP status code: `415`

```bash
$ curl https://rdap.dk-hostmaster.dk/entity/DKHM1-DK 
"Unsupported Media Type"
```

Correct specification using `curl` should be as follows:

```bash
> curl --header "Accept: application/json" https://rdap.dk-hostmaster.dk/entity/DKHM1-DK
```

And for `httpie`:

```bash
$ http https://rdap.dk-hostmaster.dk/handle/DKHM1-DK Accept:'application/json'
```

<a name="domain"></a>
## domain

This service returns data on a given domain name.

<a name="api"></a>
### API

    https://rdap.dk-hostmaster.dk/domain/{domainname}

The service returns `200` if the relevant object is available.

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="example"></a>
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

<a name="entity"></a>
## entity

This service returns data on a given handle/user-id.

<a name="api-1"></a>
### API

    https://rdap.dk-hostmaster.dk/handle/{userid}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="example-1"></a>
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

<a name="nameserver"></a>
## nameserver

This service returns data on a given hostname/nameserver.

<a name="api-2"></a>
### API

    https://rdap.dk-hostmaster.dk/host/{hostname}

| Return Code  | Description |
| ------------ | ------------ |
| `200` | OK |
| `400` | Bad request |
| `404` | Object not found |
| `415` | Unsupported media type |

<a name="example-2"></a>
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

<a name="resources"></a>
# Resources

Resources for DK Hostmaster RDAP support can be found below.

<a name="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster RDAP service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a name="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the RDAP implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `info@dk-hostmaster.dk`

<a name="additional-information"></a>
## Additional Information

The DK Hostmaster website service page

  * `https://www.dk-hostmaster.dk/en/rdap`

<a name="appendices"></a>
# Appendices

<a name="http-status-codes"></a>
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



