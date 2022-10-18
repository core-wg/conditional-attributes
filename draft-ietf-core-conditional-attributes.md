---
title: "Conditional Attributes for Constrained RESTful Environments"
abbrev: Conditional Attributes for CoRE
docname: draft-ietf-core-conditional-attributes-latest
date: 2022-10-17
category: info

ipr: trust200902
area: art
workgroup: CoRE Working Group
keyword: [Internet-Draft, CoRE, CoAP, Observe]

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
- ins: M. Koster
  name: Michael Koster
  organization: Dogtiger Labs
  street: 524 H Street
  city: Antioch, CA
  code: 94509
  country: USA
  email: michaeljohnkoster@gmail.com
- ins: A. Soloway
  name: Alan Soloway
  organization: Qualcomm Technologies, Inc.
  street: 5775 Morehouse Drive
  city: San Diego
  code: 92121
  country: USA
  email: asoloway@qti.qualcomm.com
- role: editor
  ins: B. Silverajan
  name: Bilhanan Silverajan
  org: Tampere University
  street: Kalevantie 4
  city: Tampere
  code: 'FI-33100'
  country: Finland
  email: bilhanan.silverajan@tuni.fi

normative:

informative:
  RFC7252: coap
  RFC7641: observe


--- abstract

This specification defines Conditional Notification and Control Attributes that work with CoAP Observe (RFC7641).
 
--- note_Editor_note
 
The git repository for the draft is found at https://github.com/core-wg/conditional-attributes/

--- middle

Introduction        {#introduction}
============

IETF Standards for machine-to-machine communication in constrained environments describe the Constrained Application Protocol (CoAP) {{-coap}}, a RESTful application protocol, as well as a set of related information standards that may be used to represent machine data and machine metadata in REST interfaces. 

This specification defines Conditional Notification and Control Attributes for use with CoAP Observe {{RFC7641}}.

Terminology     {#terminology}
===========

{::boilerplate bcp14}

This specification requires readers to be familiar with all the terms and concepts that are discussed in {{-coap}} and {{RFC7641}}.  This specification makes use of the following additional terminology:

Notification Band:  
: A resource value range that  may be bounded by a minimum and maximum value or may be unbounded having either a minimum or maximum value.

Conditional Attributes        {#binding_attributes}
=============

This specification defines conditional attributes for use with CoRE Observe {{RFC7641}}. Conditional attributes provide fine-grained control of notification and synchronization of resource states. When observing a resource, a CoAP client conveys conditional attributes as metadata using the query component of a CoAP URI. A conditional attribute can be represented as a "name=value" query parameter or simply a "name" without a value. Multiple conditional attributes in a query component are separated with an ampersand "&". A resource marked as Observable in its link description SHOULD support these conditional attributes.
 
Note: In this draft, we assume that there are finite quantization effects in the internal or external updates to the value representing the state of a resource; specifically, that a resource state may be updated at any time with any valid value. We therefore avoid any continuous-time assumptions in the description of the conditional attributes and instead use the phrase "sampled value" to refer to a member of a sequence of values that may be internally observed from the resource state over time.

## Conditional Notification Attributes

Conditional Notification Attributes define the conditions that trigger a notification. Conditional Notification Attributes SHOULD be evaluated on all potential notifications from a resource, whether resulting from an internal server-driven sampling process or from external update requests to the server. 

The set of Conditional Notification Attributes defined here allow a client to control how often a notification is received and how much a representation state should change in order to trigger a notification. One or more Conditional Notification Attributes MAY be included in an Observe request.

Conditional Notification Attributes are defined below:

| Attribute         | Name | Value          |
| --- | --- | --- |
| Greater Than      | c.gt       | xs:decimal      |
| Less Than         | c.lt       | xs:decimal      |
| Change Step       | c.st       | xs:decimal (>0) |
| Notification Band | c.band     | (none)          |
| Edge              | c.edge     | xs:boolean      |
{: #notificationattributes title="Conditional Notification Attributes"}


###Greater Than (c.gt) {#gt}
When present, Greater Than indicates the upper limit value the sampled value SHOULD cross before triggering a notification. A notification is sent whenever the sampled value crosses the specified upper limit value, relative to the last reported value, and the time for "c.pmin" has elapsed since the last notification. The sampled value is sent in the notification. If the value continues to rise, no notifications are generated as a result of "c.gt". If the value drops below the upper limit value then a notification is sent, subject again to the "c.pmin" time. 

The Greater Than parameter can only be supported on resources with a scalar numeric value. 

###Less Than (c.lt) {#lt}
When present, Less Than indicates the lower limit value the resource value SHOULD cross before triggering a notification. A notification is sent whenever the sampled value crosses the specified lower limit value, relative to the last reported value, and the time for "c.pmin" has elapsed since the last notification. The sampled value is sent in the notification. If the value continues to fall no notifications are generated as a result of "c.lt". If the value rises above the lower limit value then a new notification is sent, subject to the "c.pmin" time. 

The Less Than parameter can only be supported on resources with a scalar numeric value. 

###Change Step (c.st) {#st}
When present, Change step indicates how much the value representing a resource state SHOULD change before triggering a notification, compared to the previous resource state. Upon reception of a query including the "c.st" attribute, the current resource state representing the most recently sampled value is reported, and then set as the last reported value (last_rep_v). When a subsequent sampled value or update of the resource state differs from the last reported state by an amount, positive or negative, greater than or equal to st, and the time for "c.pmin" has elapsed since the last notification, a notification is sent and the last reported value is updated to the new resource state sent in the notification. The change step MUST be greater than zero otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

The Change Step parameter can only be supported on resources with a scalar numeric value.

Note: Due to sampling and other constraints, e.g. "c.pmin", the change in resource states received in two sequential notifications may differ by more than "c.st".

###Notification Band (c.band) {#band}

The Notification Band attribute allows a bounded or unbounded (based on a minimum or maximum) value range that may trigger multiple notifications. This enables use cases where different ranges results in differing behaviour. For example, in monitoring the temperature of machinery, whilst the temperature is in the normal operating range, only periodic updates are needed. However as the temperature moves to more abnormal ranges more frequent state updates may be sent to clients.

Without a notification band, a transition across a Less Than (c.lt), or Greater Than (c.gt) limit only generates one notification.  This means that it is not possible to describe a case where multiple notifications are sent so long as the limit is exceeded.

The "c.band" attribute works as a modifier to the behaviour of "c.gt" and "c.lt". Its use is determined only by its presence, and not its value. Therefore, if "c.band" is present in a query, "c.gt", "c.lt" or both, MUST be included.

When "c.band" is present with "c.lt" but without "c.gt", the lower bound for the notification band (notification band minimum) is defined. Notifications occur when the resource value is equal to or above the notification band minimum. No maximum values exist for the band. 

When "c.band" is present with "c.gt" but without "c.lt", the upper bound for the notification band (notification band maximum) is defined. Notifications occur when the resource value is equal to or below the notification band maximum. No minimum values exist for the band.

If "c.band" is specified in which the value of "c.gt" is less than that of "c.lt", in-band notification occurs. That is, notification occurs whenever the resource value is between the "c.gt" and "c.lt" values, including equal to "c.gt" or "c.lt". 

If "c.band" is specified in which the value of "c.gt" is greater than that of "c.lt", out-of-band notification occurs. That is, notification occurs when the resource value not between the "c.gt" and "c.lt" values, excluding equal to "c.gt" and "c.lt".

The Notification Band parameter can only be supported on resources with a scalar numeric value. 

###Edge (c.edge) {#edge}

When present, the Edge attribute indicates interest for receiving notifications of either the falling edge or the rising edge transition of a boolean resource state. When the value of the "c.edge" attribute is 0 (False), the server notifies the client each time a resource state changes from True to False. When the value of the "c.edge" attribute is 1 (True), the server notifies the client each time a resource state changes from False to True. 

The "c.edge" attribute can only be supported on resources with a boolean value.

## Conditional Control Attributes

Conditional Control Attributes define the time intervals between consecutive notifications as well as the cadence of the evaluation of the conditions that trigger a notification. Conditional Control Attributes can be used to configure the internal server-driven sampling process for performing evaluations of the conditions of a resource. One or more Conditional Control Attributes MAY be included in an Observe request.

Conditional Control Attributes are defined below:

| Attribute                    | Name  | Value           |
| --- | --- | --- |
| Minimum Period (s)           | c.pmin       | xs:decimal (>0) |
| Maximum Period (s)           | c.pmax       | xs:decimal (>0) |
| Minimum Evaluation Period (s)| c.epmin      | xs:decimal (>0) |
| Maximum Evaluation Period (s)| c.epmax      | xs:decimal (>0) |
| Confirmable Notification     | c.con        | xs:boolean      |
{: #controlattributes title="Conditional Control Attributes"}


###Minimum Period (c.pmin) {#pmin}

When present, Minimum Period indicates the minimum time, in seconds, between two consecutive notifications (whether or not the resource state has changed). In the absence of this parameter, the minimum period is up to the server. Minimum Period MUST be greater than zero otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

A server MAY update the resource state with the last sampled value that occured during the "c.pmin" interval, after the "c.pmin" interval expires. 

Note: Due to finite quantization effects, the time between notifications may be greater than "c.pmin" even when the sampled value changes within the "c.pmin" interval. "c.pmin" may or may not be used to drive the internal sampling process.

###Maximum Period (c.pmax) {#pmax}

When present, Maximum Period indicates the maximum time, in seconds, between two consecutive notifications (whether or not the resource state has changed). In the absence of this parameter, the maximum period is up to the server. Maximum Period MUST be greater than zero and MUST be greater than or equal to Minimum Period (if present), otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Minimum Evaluation Period (c.epmin) {#epmin}

When present, Minimum Evaluation Period indicates the minimum time, in seconds, the client recommends to the server to wait between two consecutive evaluations of the conditions of a resource since the client has no interest in the server doing more frequent evaluations. When the value of Minimum Evaluation Period expires after the previous evaluation, the server MAY immediately perform a new evaluation. In the absence of this parameter, the minimum evaluation period is not defined and thus not used by the server. The server MAY use "c.pmin", if defined, as a guidance on the desired evaluation cadence. Minimum Evaluation Period MUST be greater than zero, otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Maximum Evaluation Period (c.epmax) {#epmax}

When present, Maximum Evaluation Period indicates the maximum time, in seconds, the server MAY wait between two consecutive evaluations of the conditions of a resource. When the value of Maximum Evaluation Period expires after the previous evaluation, the server MUST immediately perform a new evaluation. In the absence of this parameter, the maximum evaluation period is not defined and thus not used by the server. Maximum Evaluation Period MUST be greater than zero and MUST be greater than Minimum Evaluation Period (if present), otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Confirmable Notification (c.con) {#con}

When present with a value of 1 (True), Confirmable Notification indicates a notification MUST be confirmable, i.e., the server MUST send the notification in a confirmable CoAP message, to request an acknowledgement from the client. When present with a value of 0 (False), Confirmable Notification indicates a notification can be confirmable or non-confirmable, i.e., it can be sent in a confirmable or a non-confirmable CoAP message.

## Server processing of Conditional Attributes

Conditional Notification Attributes and Conditional Control Attributes may be present in the same query. However, they are not defined at multiple prioritization levels. The server sends a notification whenever any of the parameter conditions are met, upon which it updates its last notification value and time to prepare for the next notification. Only one notification occurs when there are multiple conditions being met at the same time. The reference code below illustrates the logic to determine when a notification is to be sent.

~~~~
bool notifiable( Resource * r ) {

  #define EDGE EXISTS(r->edge) 
  #define BAND EXISTS(r->band) 
  #define SCALAR_TYPE ( num_type == r->type )
  #define STRING_TYPE ( str_type == r->type )
  #define BOOLEAN_TYPE ( bool_type == r->type )
  #define PMIN_EX ( r->last_sample_time - r->last_rep_time >= r->pmin )
  #define PMAX_EX ( r->last_sample_time - r->last_rep_time > r->pmax )
  #define LT_EX ( r->v < r->lt ^ r->last_rep_v < r->lt )
  #define GT_EX ( r->v > r->gt ^ r->last_rep_v > r->gt )
  #define ST_EX ( abs( r->v - r->last_rep_v ) >= r->st )
  #define IN_BAND ( ( r->gt <= r->v && r->v <= r->lt ) || \
                    ( r->lt <= r->gt && r->gt <= r->v ) || \
                    ( r->v <= r->lt && r->lt <= r->gt ) )
  #define VB_CHANGE ( r->vb != r->last_rep_vb )
  #define VB_EDGE ( r->vb && r->edge || !r->vb && !r->edge )
  #define VS_CHANGE ( r->vs != r->last_rep_vs )

  return (
    PMIN_EX &&
    ( SCALAR_TYPE ?
      ( ( !BAND && ( GT_EX || LT_EX || ST_EX || PMAX_EX ) ) ||
        ( BAND && IN_BAND && ( ST_EX || PMAX_EX) ) )
    : STRING_TYPE ?
      ( VS_CHANGE || PMAX_EX )
    : BOOLEAN_TYPE ?
      ( ( !EDGE && VB_CHANGE ) || 
        ( EDGE && VB_CHANGE && VB_EDGE ) || 
        PMAX_EX )
    : false )
  );
}
~~~~
{: #figattrint title="Code logic for conditional notification attribute interactions"}


Implementation Considerations   {#Implementation}
=======================

When "c.pmax" and "c.pmin" are equal, the expected behaviour is that notifications will be sent every (c.pmin == c.pmax) seconds. However, these notifications can only be fulfilled by the server on a best effort basis. Because "c.pmin" and "c.pmax" are designed as acceptable tolerance bounds for sending state updates, a query from an interested client containing equal "c.pmin" and "c.pmax" values must not be seen as a hard real-time scheduling contract between the client and the server.

The use of the notification band minimum and maximum allow for a synchronization whenever a change in the resource value occurs. Theoretically this could occur in-line with the server internal sample period or as defined by the "c.epmin" and "c.epmax" values for determining the resource value. Implementors SHOULD consider the resolution needed before updating the resource, e.g. updating the resource when a temperature sensor value changes by 0.001 degree versus 1 degree.

When a server has multiple observations with different measurement cadences as defined by the "c.epmin" and "c.epmax" values, the server MAY evaluate all observations when performing the measurement of any one observation.

This specification defines conditional attributes that can be used with CoAP Observe relationships between CoAP clients and CoAP servers. However, it is recognised that the presence of 1 or more proxies between a client and a server can interfere with clients receiving resource updates, if a proxy does not supply resource representations when the value remains unchanged (eg if "c.pmax" is set, and the server sends multiple updates when the resource state contains the same value). A server SHOULD use the Max-Age option to mitigate this by setting Max-Age to be less than or equal to "c.pmax".

Security Considerations   {#Security}
=======================

The security considerations in Section 11 of {{RFC7252}} apply. Additionally, the security considerations in Section 7 of {{RFC7641}} also apply.

IANA Considerations
===================

This specification requests a new Conditional Attributes registry to ensure attributes map uniquely to parameter names.

Note to IANA: Please replace "RFC XXXX" with the assigned RFC number in the table below.

| Attribute                    | Parameter  | Value           | Reference |
| -------------- | --- | --- | --- |
| Minimum Period (s)           | c.pmin       | xs:decimal (>0) | RFC XXXX |
| Maximum Period (s)           | c.pmax       | xs:decimal (>0) | RFC XXXX |
| Minimum Evaluation Period (s)| c.epmin      | xs:decimal (>0) | RFC XXXX |
| Maximum Evaluation Period (s)| c.epmax      | xs:decimal (>0) | RFC XXXX |
| Confirmable Notification     | c.con        | xs:boolean      | RFC XXXX |
| Greater Than                 | c.gt         | xs:decimal      | RFC XXXX |
| Less Than                    | c.lt         | xs:decimal      | RFC XXXX |
| Change Step                  | c.st         | xs:decimal (>0) | RFC XXXX |
| Notification Band            | c.band       | (none)          | RFC XXXX |
| Edge                         | c.edge       | xs:boolean      | RFC XXXX |


Acknowledgements
================
Hannes Tschofenig and Mert Ocak highlighted syntactical corrections in the usage of pmax and pmin in a query. David Navarro proposed allowing for pmax to be equal to pmin. 

Contributors
============

    Christian Groves
    Australia
    email: cngroves.std@gmail.com

    Zach Shelby
    ARM
    Vuokatti
    FINLAND
    phone: +358 40 7796297
    email: zach.shelby@arm.com

    Matthieu Vial
    Schneider-Electric
    Grenoble
    France
    phone: +33 (0)47657 6522
    eMail: matthieu.vial@schneider-electric.com

    Jintao Zhu
    Huawei
    Xi’an, Shaanxi Province
    China
    email: jintao.zhu@huawei.com 

Changelog
=========
draft-ietf-core-conditional-attributes-04

* Reference code updated to include behaviour for edge attribute.

draft-ietf-core-conditional-attributes-03

* Attribute names updated to create uniqueness for use as conditional observe attributes.

draft-ietf-core-conditional-attributes-02

* Clarifications on usage and value of the band parameter
* Implementation considerations for proxies added
* Security considerations added
* IANA considerations added

draft-ietf-core-conditional-attributes-01

* Clarifications on True and False values for Edge and Con Attributes
* Alan Soloway added as author

draft-ietf-core-conditional-attributes-00

* Conditional Atttributes section from draft-ietf-core-dynlink-13 separated into own WG draft

--- back

Examples
========

This appendix provides some examples of the use of binding attribute / observe attributes.

Note: For brevity the only the method or response code is shown in the header field.

Minimum Period (c.pmin) example
--------------------------

~~~~
        Observed   CLIENT  SERVER     Actual
    t   State         |      |         State
        ____________  |      |  ____________
    1                 |      |
    2    unknown      |      |     18.5 Cel
    3                 +----->|                  Header: GET
    4                 | GET  |                   Token: 0x4a
    5                 |      |                Uri-Path: temperature
    6                 |      |               Uri-Query: c.pmin="10"
    7                 |      |                 Observe: 0 (register)
    8                 |      |
    9   ____________  |<-----+                  Header: 2.05
   10                 | 2.05 |                   Token: 0x4a
   11    18.5 Cel     |      |                 Observe: 9
   12                 |      |                 Payload: "18.5 Cel"
   13                 |      |  ____________
   14                 |      |
   15                 |      |     23 Cel
   16                 |      |
   17                 |      |
   18                 |      |
   19                 |      |  ____________
   20   ____________  |<-----+                  Header: 2.05
   21                 | 2.05 |     26 Cel        Token: 0x4a
   22    26 Cel       |      |                 Observe: 20
   23                 |      |                 Payload: "26 Cel"
   24                 |      |
   25                 |      |
~~~~
{: #figbindexp1 title="Client registers and receives one notification of the current state and one of a new state state when c.pmin time expires."}

Maximum Period (c.pmax) example
--------------------------

~~~~
        Observed   CLIENT  SERVER     Actual
    t   State         |      |         State
        ____________  |      |  ____________
    1                 |      |
    2    unknown      |      |     18.5 Cel
    3                 +----->|                  Header: GET
    4                 | GET  |                   Token: 0x4a
    5                 |      |                Uri-Path: temperature
    6                 |      |               Uri-Query: c.pmax="20"
    7                 |      |                 Observe: 0 (register)
    8                 |      |
    9   ____________  |<-----+                  Header: 2.05
   10                 | 2.05 |                   Token: 0x4a
   11    18.5 Cel     |      |                 Observe: 9
   12                 |      |                 Payload: "18.5 Cel"
   13                 |      |
   14                 |      |
   15                 |      |  ____________
   16   ____________  |<-----+                  Header: 2.05
   17                 | 2.05 |     23 Cel        Token: 0x4a
   18    23 Cel       |      |                 Observe: 16
   19                 |      |                 Payload: "23 Cel"
   20                 |      |
   21                 |      |
   22                 |      |
   23                 |      |
   24                 |      |
   25                 |      |
   26                 |      |
   27                 |      |
   28                 |      |
   29                 |      |
   30                 |      |
   31                 |      |
   32                 |      |
   33                 |      |
   34                 |      |   
   35                 |      |
   36                 |      |  ____________
   37   ____________  |<-----+                  Header: 2.05
   38                 | 2.05 |     23 Cel        Token: 0x4a
   39    23 Cel       |      |                 Observe: 37
   40                 |      |                 Payload: "23 Cel"
   41                 |      |
   42                 |      |
~~~~
{: #figbindexp2 title="Client registers and receives one notification of the current state, one of a new state and one of an unchanged state when "c.pmax" time expires."}

Greater Than (c.gt) example
--------------------------

~~~~
     Observed   CLIENT  SERVER     Actual
 t   State         |      |         State
     ____________  |      |  ____________
 1                 |      |
 2    unknown      |      |     18.5 Cel
 3                 +----->|                  Header: GET 
 4                 | GET  |                   Token: 0x4a
 5                 |      |                Uri-Path: temperature
 6                 |      |               Uri-Query: c.gt=25
 7                 |      |                 Observe: 0 (register)
 8                 |      |
 9   ____________  |<-----+                  Header: 2.05 
10                 | 2.05 |                   Token: 0x4a
11    18.5 Cel     |      |                 Observe: 9
12                 |      |                 Payload: "18.5 Cel"
13                 |      |                 
14                 |      |
15                 |      |  ____________
16   ____________  |<-----+                  Header: 2.05 
17                 | 2.05 |     26 Cel        Token: 0x4a
18    26 Cel       |      |                 Observe: 16
29                 |      |                 Payload: "26 Cel"
20                 |      |                 
21                 |      |
~~~~
{: #figbindexp3 title="Client registers and receives one notification of the current state and one of a new state when it passes through the greater than threshold of 25."}

Greater Than (c.gt) and Period Max (c.pmax) example
----------------------------------

~~~~
     Observed   CLIENT  SERVER     Actual
 t   State         |      |         State
     ____________  |      |  ____________
 1                 |      |
 2    unknown      |      |     18.5 Cel
 3                 +----->|                  Header: GET 
 4                 | GET  |                   Token: 0x4a
 5                 |      |                Uri-Path: temperature
 6                 |      |         Uri-Query: c.pmax=20;c.gt=25
 7                 |      |                 Observe: 0 (register)
 8                 |      |
 9   ____________  |<-----+                  Header: 2.05 
10                 | 2.05 |                   Token: 0x4a
11    18.5 Cel     |      |                 Observe: 9
12                 |      |                 Payload: "18.5 Cel"
13                 |      |                 
14                 |      |
15                 |      |
16                 |      |
17                 |      |
18                 |      |
19                 |      |
20                 |      |
21                 |      |
22                 |      |
23                 |      |
24                 |      |
25                 |      |
26                 |      |
27                 |      |
28                 |      |
29                 |      |  ____________
30   ____________  |<-----+                  Header: 2.05
31                 | 2.05 |     23 Cel        Token: 0x4a
32    23 Cel       |      |                 Observe: 30
33                 |      |                 Payload: "23 Cel"
34                 |      |                 
35                 |      |
36                 |      |  ____________
37   ____________  |<-----+                  Header: 2.05 
38                 | 2.05 |     26 Cel        Token: 0x4a
39    26 Cel       |      |                 Observe: 37
40                 |      |                 Payload: "26 Cel"
41                 |      |                 
42                 |      |
~~~~
{: #figbindexp4 title="Client registers and receives one notification of the current state, one when "c.pmax" time expires and one of a new state when it passes through the greater than threshold of 25."}
