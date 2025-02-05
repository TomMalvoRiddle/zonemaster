# CONNECTIVITY04: IP Prefix Diversity

## Test case identifier
**CONNECTIVITY04**

## Table of contents

* [Objective](#objective)
* [Scope](#scope)
* [Inputs](#inputs)
* [Summary](#summary)
* [Test procedure](#test-procedure)
* [Outcome(s)](#outcomes)
* [Special procedural requirements](#special-procedural-requirements)
* [Prefix lookup methods](#prefix-lookup-methods)
  * [Cymru prefix lookup](#cymru-prefix-lookup)
  * [RIPE prefix lookup](#ripe-prefix-lookup)
* [Intercase dependencies](#intercase-dependencies)
* [Terminology](#terminology)

## Objective

The objective in this Test Case is to verify that all IP addresses of the domain's
authoritative name servers are announced from different IP prefixes.

This Test is done separately on IPv4 and IPv6, and both must match the criterion.

[RFC 2182, section 3.1][RFC 2182#3.1], clearly specifies that distinct authoritative
name servers for a child domain should be placed in different topological and 
geographical locations. The objective is to minimise the likelihood of a single 
failure disabling all of them.

## Scope

It is assumed that *Child Zone* is also tested and reported by [Connectivity01].
This Test Case will just ignore non-responsive name servers or name servers not
giving a correct DNS response for an authoritative name server.

## Inputs

* "Child Zone" - The domain name to be tested.
* "Prefix Database" - The database of IP Prefix data to be used. Possible values
  are "RIPE" and "Cymru" (the default value).
* "Cymru Base Name" - If the *Prefix Database* is "Cymru", the default value 
  is "asnlookup.zonemaster.net".
* "RIS Whois Server" - If the *Prefix Database* is "RIPE", the default value 
  is "riswhois.ripe.net".

## Summary

Message Tag                 | Level   | Arguments                   | Message ID for message tag
:---------------------------|:--------|:----------------------------|:------------------------------------------------------------------------------------------------
CN04_EMPTY_PREFIX_SET       | ERROR   | ns_ip                       | Prefix database returned no information for IP address {ns_ip}.
CN04_ERROR_PREFIX_DATABASE  | ERROR   | ns_ip                       | Prefix database error. No data to analyze for IP address {ns_ip}.
CN04_IPV4_SAME_PREFIX       | WARNING | ns_list, ip_prefix          | The following name server(s) are announced in the same IPv4 prefix ({ip_prefix}): "{ns_list}"
CN04_IPV4_DIFFERENT_PREFIX  | INFO    | ns_list                     | The following name server(s) are announced in unique IPv4 prefix(es): "{ns_list}"
CN04_IPV6_SAME_PREFIX       | WARNING | ns_list, ip_prefix          | The following name server(s) are announced in the same IPv6 prefix ({ip_prefix}): "{ns_list}"
CN04_IPV6_DIFFERENT_PREFIX  | INFO    | ns_list                     | The following name server(s) are announced in unique IPv6 prefix(es): "{ns_list}"

The value in the Level column is the default severity level of the message. The
severity level can be changed in the [Zonemaster-Engine Profile]. Also see the
[Severity Level Definitions] document.

The argument names in the Arguments column lists the arguments used in the
message. The argument names are defined in the [Argument List].

## Test procedure

1. Create the following empty sets:
   1. Name server name and IP address ("NS IPv4")
   2. Name server name and IP address ("NS IPv6")
   3. IP prefix, name server name and IP address ("IPv4 Prefix")
   4. IP prefix, name server name and IP address ("IPv6 Prefix")

2. Obtain the set of name server names and IP addresses using [Method4]
   and [Method5] ("Name Servers").

3. For each name server IP address in *Name Servers* do:
   1. Add IPv4 addresses with name server name to the *NS IPv4* set.
   2. Add IPv6 addresses with name server name to the *NS IPv6* set.

4. For each IP address in *NS IPv4* and *NS IPv6* ("NS IP Address"),
   respectively, do:
   1. Determine the IP prefix in which *NS IP Address* is announced
      using *Prefix Database*. See [Prefix Lookup Methods] section below.
   2. Add found IP prefix, if any, with *NS IP Address* and name server name
      to the *IPv4 Prefix* and *IPv6 Prefix* sets, respectively.

5. If the *IPv4 Prefix* set is non-empty, then do:
   1. For each IP prefix in the set that has two or more members, output
      *[CN04_IPV4_SAME_PREFIX]* with the prefix and list of all members
      (name server names and IP addresses) for that prefix.
   2. For all IP prefixes in the set that have exactly one member, output
      *[CN04_IPV4_DIFFERENT_PREFIX]* with the combined set of their associated
      members (name server names and IP addresses).

6. If the *IPv6 Prefix* set is non-empty, then do:
   1. For each IP prefix in the set that has two or more members, output
      *[CN04_IPV6_SAME_PREFIX]* with the prefix and list of all members
      (name server names and IP addresses) for that prefix.
   2. For all IP prefixes in the set that have exactly one member, output
      *[CN04_IPV6_DIFFERENT_PREFIX]* with the combined set of their associated
      members (name server names and IP addresses).

## Outcome(s)

The outcome of this Test Case is "fail" if there is at least one message
with the severity level *[ERROR]* or *[CRITICAL]*.

The outcome of this Test Case is "warning" if there is at least one message
with the severity level *[WARNING]*, but no message with severity level
*ERROR* or *CRITICAL*.

In other cases, no message or only messages with severity level
*[INFO]* or *[NOTICE]*, the outcome of this Test Case is "pass".

## Special procedural requirements

This Test Case is dependent on one of two possible services that can provide
ASN lookup (Cymru or RIPE RIS). The service must be available over the network.

The *Child Zone* must be a valid name meeting
"[Requirements and normalization of domain names in input]".

## Prefix lookup methods

Use the prefix method set in *Prefix Database* in "Inputs" and refer to
the appropriate section below.

### Cymru prefix lookup

The Cymru prefix lookup is described on the Team Cymru [IP to ASN Mapping]
using DNS lookup.

1. Prepend the *Cymru Base Name* with the label "origin" (IPv4) or 
   "origin6" (IPv6) ("Expanded Base Name"). Example of expanded basenames :
   
```
origin.asnlookup.zonemaster.net
origin6.asnlookup.zonemaster.net
```

2. Reverse the IP address with the same method as is used for
   reverse lookup ("Reverse IP"). For description see [RFC 1035][RFC 1035#3.5],
   section 3.5, for IPv4 and [RFC 3596][RFC 3596#2.5], section 2.5, for IPv6.
 
3. Prepend the *Expanded Base Name* with *Reverse IP* ("Query Name").
   See [IP to ASN Mapping] for details.

4. Create a [DNS Query] with query type TXT and query name *Query Name*.
   ("TXT Query").

5. [Send] *TXT Query* to the *Cymru Base Name*.

6. If at least one of the following criteria is met, output
   *[CN04_EMPTY_PREFIX_SET]* and exit this lookup:
   1. The [DNS Response] has the [RCODE Name] NXDomain.
   2. The [DNS Response] has the [RCODE Name] NoError and an empty answer section.

7. If at least one of the following criteria is met, output
   *[CN04_ERROR_PREFIX_DATABASE]* and exit this lookup:
   1. There is no DNS response.
   2. The [DNS Response] does not have the [RCODE Name] NoError or NXDomain.

8. Extract the TXT record(s) from the response (see [IP to ASN Mapping]
   for examples), and do:
   1. If there are multiple strings (i.e., TXT records), ignore all strings
      except for the string with the most specific subnet.
   2. Extract and [concatenate] the IP prefix from the string.
   3. If it was not possible to extract the IP prefix (i.e., malformed response),
      output *[CN04_ERROR_PREFIX_DATABASE]* and exit this lookup.

9. Return the IP prefix.

### RIPE prefix lookup

The RIPE Prefix lookup is described on the RIPE [RISwhois] page.

1. Create a query string by prepending the IP address with 
   " -F -M " ("WHOIS String"). E.g., using IP address "192.0.2.10":
   
   ```
   " -F -M 192.0.2.10" 
   ```

2. Create a WHOIS query (port 43 with the nicname ((whois)) protocol)
   using the *WHOIS String* ("WHOIS Query"). E.g., on Linux:

```
whois -h riswhois.ripe.net " -F -M 192.0.2.10"
```

3. [Send] *WHOIS Query* to the *RIS Whois Server*.

4. If there is no response, output *[CN04_ERROR_PREFIX_DATABASE]* and exit this lookup.

5. Extract the string (non-empty line not prepended with "%") from the response, and do:
   1. If there is no such string, output *[CN04_EMPTY_PREFIX_SET]* and exit this lookup.
   2. Extract the IP prefix from the second field of the string.
   3. If it was not possible to extract the IP prefix (i.e., malformed response),
      output *[CN04_ERROR_PREFIX_DATABASE]* and exit this lookup.

6. Return the IP prefix.

## Intercase dependencies

None

## Terminology

* "Concatenate" - The term is used to refer to the conversion of a TXT
  resource record’s data to a single contiguous string, as specified in [RFC
  7208, section 3.3][RFC7208#3.3].

* "Send" - The term is used when a query is sent to
  a specific server (server IP address).

[Argument List]:                                                https://github.com/zonemaster/zonemaster-engine/blob/master/docs/logentry_args.md
[CN04_EMPTY_PREFIX_SET]:                                        #outcomes
[CN04_ERROR_PREFIX_DATABASE]:                                   #outcomes
[CN04_IPV4_DIFFERENT_PREFIX]:                                   #outcomes
[CN04_IPV4_SAME_PREFIX]:                                        #outcomes
[CN04_IPV6_DIFFERENT_PREFIX]:                                   #outcomes
[CN04_IPV6_SAME_PREFIX]:                                        #outcomes
[Concatenate]:                                                  #terminology
[Connectivity01]:                                               connectivity01.md
[CRITICAL]:                                                     ../SeverityLevelDefinitions.md#critical
[Cymru Database]:                                               #cymru-prefix-lookup
[DEBUG]:                                                        ../SeverityLevelDefinitions.md#notice
[DNS Query and Response Defaults]:                              ../DNSQueryAndResponseDefaults.md
[DNS Query]:                                                    ../DNSQueryAndResponseDefaults.md#default-setting-in-dns-query
[DNS Response]:                                                 ../DNSQueryAndResponseDefaults.md#default-handling-of-a-dns-response
[ERROR]:                                                        ../SeverityLevelDefinitions.md#error
[INFO]:                                                         ../SeverityLevelDefinitions.md#info
[IP to ASN Mapping]:                                            https://www.team-cymru.com/ip-asn-mapping
[Method4]:                                                      ../Methods.md#method-4-obtain-glue-address-records-from-parent
[Method5]:                                                      ../Methods.md#method-5-obtain-the-name-server-address-records-from-child
[Methods]:                                                      ../Methods.md
[NOTICE]:                                                       ../SeverityLevelDefinitions.md#notice
[Prefix Lookup Methods]:                                        #prefix-lookup-methods
[RCODE Name]:                                                   https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-6
[Requirements and normalization of domain names in input]:      ../RequirementsAndNormalizationOfDomainNames.md
[RFC 1035#3.5]:                                                 https://datatracker.ietf.org/doc/html/rfc1035#section-3.5
[RFC 2182#3.1]:                                                 https://datatracker.ietf.org/doc/html/rfc2182#section-3.1
[RFC 3596#2.5]:                                                 https://datatracker.ietf.org/doc/html/rfc3596#section-2.5
[RFC7208#3.3]:                                                  https://datatracker.ietf.org/doc/html/rfc7208#section-3.3
[RIPE Database]:                                                #ripe-prefix-lookup
[RISwhois]:                                                     https://www.ripe.net/analyse/archived-projects/ris-tools-web-interfaces/riswhois
[Send]:                                                         #terminology
[Severity Level Definitions]:                                   ../SeverityLevelDefinitions.md
[WARNING]:                                                      ../SeverityLevelDefinitions.md#warning
[Zonemaster-Engine Profile]:                                    ../../../configuration/profiles.md
