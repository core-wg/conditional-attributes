---
v: 3

title: "Conditional Attributes for Constrained RESTful Environments"
abbrev: Conditional Attributes for CoRE
docname: draft-ietf-core-conditional-attributes-latest

category: std
stream: IETF
area: Applications
workgroup: CoRE Working Group

venue:
  mail: core@ietf.org
  github: core-wg/conditional-attributes

author:
- name: Bilhanan Silverajan
  org: Tampere University
  street: Kalevantie 4
  city: Tampere
  code: 'FI-33100'
  country: Finland
  email: bilhanan.silverajan@tuni.fi
- name: Michael Koster
  org: Dogtiger Labs
  street: 524 H Street
  city: Antioch, CA
  code: 94509
  country: USA
  email: michaeljohnkoster@gmail.com
- name: Alan Soloway
  organization: Qualcomm Technologies, Inc.
  street: 5775 Morehouse Drive
  city: San Diego
  code: 92121
  country: USA
  email: asoloway@qti.qualcomm.com

contributor:
- name: Christian Groves
  country: Australia 
  email: cngroves.std@gmail.com
- name: Zach Shelby
  organization: ARM
  city: Vuokatti
  country: Finland
  email: zach.shelby@arm.com
- name: Matthieu Vial
  organization: Schneider-Electric
  city: Grenoble
  country: France
  email: matthieu.vial@schneider-electric.com
- name: Jintao Zhu
  organization: Huawei
  city: Xi’an, Shaanxi Province
  country: China
  email: jintao.zhu@huawei.com 
  

normative:
  RFC7252:
  RFC7641:
  RFC8126:


informative:
  I-D.irtf-t2trg-amplification-attacks: t2trg-attacks

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This specification defines Conditional Notification and Control Attributes that work with CoAP Observe (RFC7641).

--- middle

Introduction        {#introduction}
============

IETF Standards for machine-to-machine communication in constrained environments describe the Constrained Application Protocol (CoAP) {{RFC7252}}, a RESTful application protocol, as well as a set of related information standards that may be used to represent machine data and machine metadata in REST interfaces. 

This specification defines Conditional Notification and Control Attributes for use with CoAP Observe {{RFC7641}}.

Terminology     {#terminology}
===========

{::boilerplate bcp14}

This specification requires readers to be familiar with all the terms and concepts that are discussed in {{RFC7252}} and {{RFC7641}}.  This specification makes use of the following additional terminology:

Notification Band:  
: A resource value range that  may be bounded by a minimum and maximum value or may be unbounded having either a minimum or maximum value.

Conditional Attributes        {#binding_attributes}
=============

This specification defines conditional attributes for use with CoRE Observe {{RFC7641}}. Conditional attributes provide fine-grained control of notification and synchronization of resource states. A CoAP client conveys conditional attributes as metadata using the query component of a CoAP URI. A conditional attribute can be represented as a "name=value" query parameter or simply a "name" without a value. Multiple conditional attributes in a query component are separated with an ampersand "&". A resource marked as Observable in its link description SHOULD support these conditional attributes.
 
This specification assumes that there are finite quantization effects in the internal or external updates to the value representing the state of a resource; specifically, that a resource state may be updated at any time with any valid value. We therefore avoid any continuous-time assumptions in the description of the conditional attributes and instead use the phrase "sampled value" to refer to a member of a sequence of values that may be internally observed from the resource state over time.

## Overview

If a CoAP client is interested in obtaining all the state representations of a resource from a CoAP server as they change, the client is able to do so by using CoAP Observe. If a CoAP client is instead interested in receiving only state representations fulfilling certain constraints (such as a minimum/maximum value), it can do so by indicating conditional attributes as query paramets in its request to a CoAP server, when registering its interest in observing a resource.

The usage of conditional attributes employs the notion of resource state projection, in which the client requests the server to project a new state from the current resource representation. When a server receives a request containing conditional attributes from a client, the server maintains a projected resource state separate from a resource state requested without conditional attributes. 

The mechanism can be explained in the following subsections in terms of registration, operation and cancellation.


## Registration

In this example, 3 CoAP endpoints are shown: Clients A and B are interested in obtaining updates to state representations describing the current CO2 level, provided by a CoAP Server.


~~~~
ClientA         ClientB                   Server
   │               │                        │
   │               │                     ( CO2 )
   │ GET /CO2      │                        │
   │ Token: 0x42   │                        │
   │ Observe: 0    │                        │
   +───────────────┼───────────────────────>│
   │               │                        │
   │               │                        │
   │               │           2.05 Content │
   │               │            Token: 0x42 │
   │               │            Observe: 12 │
   │               │     Payload: "600 ppm" │
   │<──────────────┼────────────────────────+
   │               │                        │
   │               │                        │
   │               │           2.05 Content │
   │               │            Token: 0x42 │
   │               │            Observe: 23 │
   │               │     Payload: "800 ppm" │
   │<──────────────┼────────────────────────+
   │               │                        │

~~~~
{: #fig-reg-client-a title="Client A registers and receives one notification of the current state and one state update."}


Client B, on the other hand is interested in receiving only a subset of updates from the Server. In {{fig-reg-client-b}}, Client B is depicted using CoAP Observe with a conditional attribute to register its interest in receiving specific updates to the C02 resource state from the Server. The Server provides a representation of the current state and creates a new state projection in which the interest of Client B is registered.

~~~~
ClientA         ClientB                   Server
   │               │                        │
   │               │                     ( CO2 )
   │               │                        │
   │               │                        │
   │               │ GET /CO2?c.gt=1000     │
   │               │ Token: 0x66            │
   │               │ Observe: 0             │
   │               +───────────────────────>│
   │               │                        │
   │               │           2.05 Content │
   │               │            Token: 0x66 │
   │               │            Observe: 20 │      Resource State
   │               │     Payload: "800 ppm" │        Projection
   │               │<───────────────────────+    ..................
   │               │                        +--->. /CO2?c.gt=1000 .
   │               │                        │    ..................
   │               │                        │             .
   │               │                        │             .
   │               │                        │             .

~~~~
{: #fig-reg-client-b title="Client B registers with conditional attributes, and receives one notification of the current state and a state projection is created."}



## Operation

In subsequent interactions for providing state updates, the Server will continue to provide all state updates to Client A, while Client B receives state updates fulfilling the conditions specified by the conditional attribute.


~~~~
ClientA         ClientB                   Server
   │               │                        │
   │               │                     ( CO2 )
   │               │                        │
   │               │                        │      Resource State
   │               │                        │        Projection
   │               │                        │    ..................
   │               │                        +--->. /CO2?c.gt=1000 .
   │               │                        │    ..................
   │               │                        │             .
   │               │           2.05 Content │             .
   │               │            Token: 0x42 │             .
   │               │            Observe: 29 │             .
   │               │    Payload: "1000 ppm" │             .
   │<──────────────┼────────────────────────+             .
   │               │                        │             .
   │               │           2.05 Content │             .
   │               │            Token: 0x66 │             .
   │               │            Observe: 23 │             .
   │               │    Payload: "1100 ppm" │             .
   │               │<───────────────────────┤-------------+
   │               │                        │             .
   │               │           2.05 Content │             .
   │               │            Token: 0x42 │             .
   │               │            Observe: 33 │             .
   │               │    Payload: "1100 ppm" │             .
   │<──────────────┼────────────────────────+             .
   │               │                        │             .

~~~~
{: #fig-operation title="Clients A and B receiving C02 state updates from the Server, without and with conditional attributes, respectively."}


## Cancellation

A client that wishes to cancel an existing registration can do so in accordance with Section 3.6 of {{RFC7641}}. If a client wishes to explicitly cancel an existing registration by issuing a GET request, it MUST also additionally supply the original URI containing the conditional attributes that was conveyed to the server during the registration. This is depicted in {{fig-cancellation}} for Client B.


~~~~
ClientA         ClientB                   Server
   │               │                        │
   │               │                     ( CO2 )
   │               │                        │
   │               │                        │      Resource State
   │               │                        │        Projection
   │               │                        │    ..................
   │               │                        +--->. /CO2?c.gt=1000 .
   │               │                        │    ..................
   │               │                        │             .
   │               │                        │             .
   │               │ GET /CO2?c.gt=1000     │             .
   │               │ Token: 0x66            │             .
   │               │ Observe: 1             │             .
   │               +────────────────────────┤------------>.             
   │               │                        │             .
   │               │                        │             .
   │               │           2.05 Content │             .
   │               │            Token: 0x66 │             .
   │               │     Payload: "900 ppm" │             .
   │               │<───────────────────────┤-------------+
   │               │                        │              
   │               │                        │              
   │               │                        │              

~~~~
{: #fig-cancellation title="Client B explicitly cancelling an existing registration."}

## Conditional Notification Attributes

Conditional Notification Attributes define the conditions that trigger a notification. Conditional Notification Attributes SHOULD be evaluated on all potential notifications from a resource, whether resulting from an internal server-driven sampling process or from external update requests to the server. 

The set of Conditional Notification Attributes defined here allows a client to control how often a notification is received and how much a representation state should change in order to trigger a notification. One or more Conditional Notification Attributes MAY be included in an Observe request.

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
When present, Change step indicates how much the value representing a resource state SHOULD change before triggering a notification, compared to the previous resource state. Upon reception of a query including the "c.st" attribute, the current resource state representing the most recently sampled value is reported, and then set as the last reported value (last_rep_v). When a subsequent sampled value or update of the resource state differs from the last reported state by an amount, positive or negative, greater than or equal to "c.st", and the time for "c.pmin" has elapsed since the last notification, a notification is sent and the last reported value is updated to the new resource state sent in the notification. The change step MUST be greater than zero, otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

The Change Step parameter can only be supported on resources with a scalar numeric value.

Note: due to sampling and other constraints, e.g., "c.pmin", the change in resource states received in two sequential notifications may differ by more than "c.st".

###Notification Band (c.band) {#band}

The Notification Band attribute allows a bounded or unbounded (based on a minimum or maximum) value range that may trigger multiple notifications. This enables use cases where different ranges result in differing behaviour. For example, in monitoring the temperature of machinery, whilst the temperature is in the normal operating range, only periodic updates are needed. However as the temperature moves to more abnormal ranges, more frequent state updates may be sent to clients.

Without a notification band, a transition across a Less Than (c.lt), or Greater Than (c.gt) limit only generates one notification.  This means that it is not possible to describe a case where multiple notifications are sent so long as the limit is exceeded.

The "c.band" attribute works as a modifier to the behaviour of "c.gt" and "c.lt". Its use is determined only by its presence, as this attribute takes no value. Therefore, if "c.band" is present in a query, "c.gt", "c.lt", or both, MUST be included.

When "c.band" is present with "c.lt" but without "c.gt", the lower bound for the notification band (notification band minimum) is defined. Notifications occur when the resource value is equal to or above the notification band minimum. No maximum values exist for the band. 

When "c.band" is present with "c.gt" but without "c.lt", the upper bound for the notification band (notification band maximum) is defined. Notifications occur when the resource value is equal to or below the notification band maximum. No minimum values exist for the band.

If "c.band" is specified and the value of "c.gt" is less than that of "c.lt", in-band notification occurs. That is, notification occurs whenever the resource value is between the "c.gt" and "c.lt" values, including equal to "c.gt" or "c.lt". 

If "c.band" is specified and the value of "c.gt" is greater than that of "c.lt", out-of-band notification occurs. That is, notification occurs when the resource value is not between the "c.gt" and "c.lt" values, excluding equal to "c.gt" and "c.lt".

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

When present, Minimum Period indicates the minimum time, in seconds, between two consecutive notifications (whether or not the resource state has changed). In the absence of this parameter, the minimum period is up to the server. Minimum Period MUST be greater than zero, otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

A server MAY update the resource state with the last sampled value that occurred during the "c.pmin" interval, after the "c.pmin" interval expires. 

Note: due to finite quantization effects, the time between notifications may be greater than "c.pmin" even when the sampled value changes within the "c.pmin" interval. "c.pmin" may or may not be used to drive the internal sampling process.

###Maximum Period (c.pmax) {#pmax}

When present, Maximum Period indicates the maximum time, in seconds, between two consecutive notifications (regardless of whether or not the resource state has changed). In the absence of this parameter, the maximum period is up to the server. Maximum Period MUST be greater than zero and MUST be greater than or equal to Minimum Period (if present), otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Minimum Evaluation Period (c.epmin) {#epmin}

When present, Minimum Evaluation Period indicates the minimum time, in seconds, the client recommends to the server to wait between two consecutive evaluations of the conditions of a resource, since the client has no interest in the server doing more frequent evaluations. When the value of Minimum Evaluation Period expires after the previous evaluation, the server MAY immediately perform a new evaluation. In the absence of this parameter, the minimum evaluation period is not defined and thus not used by the server. The server MAY use "c.pmin", if defined, as a guidance on the desired evaluation cadence. Minimum Evaluation Period MUST be greater than zero, otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Maximum Evaluation Period (c.epmax) {#epmax}

When present, Maximum Evaluation Period indicates the maximum time, in seconds, the server MAY wait between two consecutive evaluations of the conditions of a resource. When the value of Maximum Evaluation Period expires after the previous evaluation, the server MUST immediately perform a new evaluation. In the absence of this parameter, the maximum evaluation period is not defined and thus not used by the server. Maximum Evaluation Period MUST be greater than zero and MUST be greater than Minimum Evaluation Period (if present), otherwise the receiver MUST return a CoAP error code 4.00 "Bad Request" (or equivalent).

###Confirmable Notification (c.con) {#con}

When present with a value of 1 (True), Confirmable Notification indicates that a notification MUST be confirmable, i.e., the server MUST send the notification in a confirmable CoAP message, to request an acknowledgement from the client. When present with a value of 0 (False), Confirmable Notification indicates a notification can be confirmable or non-confirmable, i.e., it can be sent in a confirmable or a non-confirmable CoAP message.

## Server processing of Conditional Attributes

Conditional Notification Attributes and Conditional Control Attributes may be present in the same query. However, they are not defined at multiple prioritization levels. The server sends a notification whenever any of the parameter conditions are met, upon which it updates its last notification value and time to prepare for the next notification. When Conditional Notification Attributes and Conditional Control Attributes are present in the same query, notifications may be subjected to the presence of a Conditional Control Attribute such as "c.pmin" or "c.pmax". Only one notification occurs when there are multiple conditions being met at the same time. As a general example, the pseudocode illustrated in {{pseudocode}} shows one way to determine when a notification is to be sent.



Implementation Considerations   {#Implementation}
=======================

When "c.pmax" and "c.pmin" are equal, the expected behaviour is that notifications will be sent every (c.pmin == c.pmax) seconds. However, these notifications can only be fulfilled by the server on a best effort basis. Because "c.pmin" and "c.pmax" are designed as acceptable tolerance bounds for sending state updates, a query from an interested client containing equal "c.pmin" and "c.pmax" values must not be seen as a hard real-time scheduling contract between the client and the server.

The use of the notification band minimum and maximum allows for a synchronization whenever a change in the resource value occurs. Theoretically, this could occur in-line with the server internal sample period or as defined by the "c.epmin" and "c.epmax" values for determining the resource value. Implementors SHOULD consider the resolution needed before updating the resource, e.g., updating the resource when a temperature sensor value changes by 0.001 degree versus 1 degree.

When a server has multiple observations with different measurement cadences as defined by the "c.epmin" and "c.epmax" values, the server MAY evaluate all observations when performing the measurement of any one observation.

This specification defines conditional attributes that can be used with CoAP Observe relationships between CoAP clients and CoAP servers. However, it is recognised that the presence of one or more proxies between a client and a server can interfere with clients receiving resource updates, if a proxy does not supply resource representations when the value remains unchanged (e.g., if "c.pmax" is set, and the server sends multiple updates when the resource state contains the same value). A server SHOULD use the Max-Age option to mitigate this, by setting Max-Age to be less than or equal to "c.pmax".



# Security Considerations {#seccons}

The security considerations in {{Section 11 of RFC7252}} apply. 

Additionally, the security considerations in {{Section 7 of RFC7641}} also apply, particularly towards mitigating amplification attacks.

As noted in {{Section 2.2 of I-D.irtf-t2trg-amplification-attacks}}, an attacker might choose to craft GET requests, in which observations are requested together with conditional attributes such as c.pmax or c.epmax with values that are below a minimum implementation-specific threshold. If a server receives such a request and is unwilling to register the observer client, the server MAY silently ignore the registration request and process the GET request as usual.  The resulting response MUST NOT include an Observe Option, the absence of which signals to the client that it will not be added to the list of observers by the server.


# IANA Considerations {#iana}

This document has the following actions for IANA:

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

This document establishes the "Conditional Attributes" registry within the "Constrained RESTful Environments (CoRE) Parameters" registry group, in order to ensure that attributes map uniquely to query parameter names.

Each entry in the registry must include:

* Attribute: This is the human-readable name and description of the attribute,
* Parameter: This is the short name, as used in query parameters,
* Value Type: The value type of the attribute (if any),
* Reference: The link to reference documentation, which must give details describing the conditional notification or control attribute and how it is to be processed.


Initial entries in this subregistry are as follows:

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
{: #conditionalattributes-registry title="New Conditional Attributes registry"}



The IANA policy for future additions to the subregistry is Expert Review, as described in {{RFC8126}}. The evaluation should consider formal criteria and duplication of functionality (i.e., is the new entry redundant with an existing one). To reduce potential for conflict with commonly used query parameter names, it is strongly recommended that new entry names be prepended with "c." (such as entries described in {{conditionalattributes-registry}} ).


--- back

# Pseudocode: Processing Conditional Attributes {#pseudocode}

This appendix is informative. It describes the possible logic of how a server processes conditional attributes to determine when to send a notification to a client. 

Note: The pseudocode is not exhaustive nor should it be treated as reference code. It depicts a subset of the conditional attributes described in this specification.


~~~~

// struct Resource {
//
//  bool band;
//  int pmin;
//  int pmax;
//  int epmin;
//  int epmax;
//  int st;
//  int gt;
//  int lt;
//  
//  time_t last_sampled_time;
//  time_t last_rep_time;

//  int curr_state;
//  int prev_state;   
//
//  ...
//
// };


boolean is_notifiable( Resource * r ) {

  time_t curr_time = get_current_time();

  #define BAND_EXISTS ( r->band )

  #define LT_EXISTS ( r->lt )
  #define GT_EXISTS ( r->gt )
   
  #define EPMIN_TRUE ( curr_time - r->last_sampled_time >= r->epmin )
  #define EPMAX_TRUE ( curr_time - r->last_sampled_time > r->epmax )
    
  #define PMIN_TRUE ( curr_time - r->last_reported_time >= r->pmin )
  #define PMAX_TRUE ( curr_time - r->last_reported_time > r->pmax )

  #define LT_TRUE ( r->curr_state < r->lt ^ r->prev_state < r->lt )
  #define GT_TRUE ( r->curr_state > r->gt ^ r->prev_state > r->gt )

  #define ST_TRUE ( abs( r->curr_state - r->prev_state ) >= r->st )

  #define INBAND_TRUE ( gt < lt && \\
                       (gt <= curr_state && curr_state <= lt ))
  #define OUTOFBAND_TRUE ( lt < gt && \\
                           (gt < curr_state || curr_state < lt ))
    
  #define BANDMIN_TRUE ( r->lt <= r->curr_state)
  #define BANDMAX_TRUE (r->curr_state <= r->gt)


  if PMAX_TRUE {
        return true;
  }
    
  if PMIN_TRUE {
      if !BAND_EXISTS {
          if LT_TRUE || GT_TRUE || ST_TRUE {
              return true;
          }
      }
      else {
       if ( (BANDMIN_TRUE && !GT_EXISTS) || \
            (BANDMAX_TRUE && !LT_EXISTS) || \
             INBAND_TRUE || \
             OUTOFBAND_TRUE ) {
           return true;
       }
      }
  }

  return false;
    
}

~~~~
{: #figattrint title="Pseudocode showing the logic for processing conditional attributes"}


#Examples


This appendix is informative. It provides some examples of the use of Conditional Attributes.

Note: For brevity, only the method or response code is shown in the header field.

Minimum Period (c.pmin) example
--------------------------

~~~~aasvg
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

~~~~aasvg
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
{: #figbindexp2 title="Client registers and receives one notification of the current state, one of a new state and one of an unchanged state when c.pmax time expires."}

Greater Than (c.gt) example
--------------------------

~~~~aasvg
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

~~~~aasvg
     Observed   CLIENT  SERVER     Actual
 t   State         |      |         State
     ____________  |      |  ____________
 1                 |      |
 2    unknown      |      |     18.5 Cel
 3                 +----->|                  Header: GET 
 4                 | GET  |                   Token: 0x4a
 5                 |      |                Uri-Path: temperature
 6                 |      |         Uri-Query: c.pmax=20&c.gt=25
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
{: #figbindexp4 title="Client registers and receives one notification of the current state, one when c.pmax time expires, and one of a new state when it passes through the greater than threshold of 25."}


# Acknowledgements
{: numbered='no'}

Hannes Tschofenig and Mert Ocak highlighted syntactical corrections in the usage of pmax and pmin in a query. David Navarro proposed allowing for pmax to be equal to pmin. Marco Tiloca and Ines Robles provided extensive reviews. Suggestions from Klaus Hartke aided greatly in clarifying how conditional attributes work with CoAP Observe. Security considerations were improved based on authors' observations in {{Section 2.2 of I-D.irtf-t2trg-amplification-attacks}}.

# Changelog # {#changelog}
{: numbered='no'}
{:removeinrfc}


draft-ietf-core-conditional-attributes-08

* Various editorial fixes and corrections based on review comments on mailing list from Marco Tiloca.

draft-ietf-core-conditional-attributes-07

* Expanded how conditional attributes work with Observe in sections 3.1 to 3.4
* Addressed early review from IoT Directorate
* Security Considerations section expanded 

draft-ietf-core-conditional-attributes-06

* Removed code block from Section 3.5
* Added an appendix containing pseudocode for server processing.


draft-ietf-core-conditional-attributes-05

* Multiple (mostly editorial) clarifications and updates based on review comments on mailing list from Marco Tiloca.

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
