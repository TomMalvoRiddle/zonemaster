# NAMESERVER15: Checking for revealed software version

## Test case identifier
**NAMESERVER15**

## Table of contents

* [Objective](#objective)
* [Scope](#scope)
* [Inputs](#inputs)
* [Summary](#summary)
* [Test procedure](#test-procedure)
* [Outcome(s)](#outcomes)
* [Special procedural requirements](#special-procedural-requirements)
* [Intercase dependencies](#intercase-dependencies)
* [Terminology](#terminology)

## Objective

This Test Case verifies if a name server responds to TXT queries in the CHAOS class, specifically
about its software version as it may sometimes be desirable not to reveal that information.

A description of DNS classes can be found in [RFC2929], section 3.2.

## Scope

It is assumed that *Child Zone* is also tested and reported by [Connectivity01].
This Test Case will just ignore non-responsive name servers or name servers not
giving a correct DNS response for an authoritative name server.

## Inputs

* "Child Zone" - The domain name to be tested.

## Summary

Message Tag                 | Level   | Arguments                      | Message ID for message tag
:---------------------------|:--------|:-------------------------------|:----------------------------------------------------------------------------------------------------------------------------
N15_SOFTWARE_VERSION        | INFO    | ns_ip_list, query_name, string | The following name server(s) respond to software version query "{query_name}" with string "{string}". Returned from name servers: "{ns_ip_list}"
N15_NO_VERSION              | INFO    | ns_ip_list                     | The following name server(s) do not respond to software version queries. Returned from name servers: "{ns_ip_list}"

The value in the Level column is the default severity level of the message. The
severity level can be changed in the [Zonemaster-Engine Profile]. Also see the
[Severity Level Definitions] document.

The argument names in the Arguments column lists the arguments used in the
message. The argument names are defined in the [Argument List].

## Test procedure

1. Create the following empty sets:
   1. Name server IP, query name and string ("TXT Data")
   2. Name server IP ("No Version")

2. Create [DNS Query] with query type TXT and query class CHAOS ("TXT Query").

3. Create the set of query names with values "version.bind"
   and "version.server" ("Query Names").

4. Obtain the set of name server IP addresses using [Method4] and
   [Method5] ("Name Server IP").

5. For each name server in *Name Server IP* do:
   1. For each query name in *Query Names* do:
      1. [Send] *TXT Query* with query name to
         the name server and collect the response.
      2. If at least one of the following criteria is met, go to next query name:
         1. There is no DNS response.
         2. The [DNS Response] does not have the [RCODE Name] NoError.
         3. The [DNS Response] does not have any TXT record in the
            answer section with query name as owner name.
      3. For each TXT record in the answer section of the [DNS Response]
         with owner name query name, extract and [concatenate] the string(s)
         from the RDATA of the record.
      4. If the extracted string is non-empty, add *Name Server IP*, query name
         and the string to the *TXT Data* set.
   2. If nothing was added to the *TXT Data* set in the previous steps,
      add name server to the *No Version* set.

6. If the *TXT Data* set is non-empty, then, for each unique
   string and query name pair in the set, output *[N15_SOFTWARE_VERSION]*
   with name server IP list, query name and string.

7. If the *No Version* set is non-empty, then output
   *[N15_NO_VERSION]* with name server IP list.

## Outcome(s)

The outcome of this Test Case is "fail" if there is at least one message
with the severity level *[ERROR]* or *[CRITICAL]*.

The outcome of this Test Case is "warning" if there is at least one message
with the severity level *[WARNING]*, but no message with severity level
*[ERROR]* or *[CRITICAL]*.

In other cases, no message or only messages with severity level
*[INFO]* or *[NOTICE]*, the outcome of this Test Case is "pass".

## Special procedural requirements

The *Child Zone* must be a valid name meeting
"[Requirements and normalization of domain names in input]".

## Intercase dependencies

None

## Terminology

* "Concatenate" - The term is used to refer to the conversion of a TXT
  resource record’s data to a single contiguous string, as specified in [RFC
  7208, section 3.3][RFC7208#3.3].

* "Send" - The term is used when a DNS query is sent to
  a specific name server (name server IP address).

[Argument List]:                                                https://github.com/zonemaster/zonemaster-engine/blob/master/docs/logentry_args.md
[Concatenate]:                                                  #terminology
[Connectivity01]:                                               ../Connectivity-TP/connectivity01.md
[CRITICAL]:                                                     ../SeverityLevelDefinitions.md#critical
[DEBUG]:                                                        ../SeverityLevelDefinitions.md#notice
[DNS Query and Response Defaults]:                              ../DNSQueryAndResponseDefaults.md
[DNS Query]:                                                    ../DNSQueryAndResponseDefaults.md#default-setting-in-dns-query
[DNS Response]:                                                 ../DNSQueryAndResponseDefaults.md#default-handling-of-a-dns-response
[ERROR]:                                                        ../SeverityLevelDefinitions.md#error
[INFO]:                                                         ../SeverityLevelDefinitions.md#info
[Message Tag Specification]:                                    ../../../../internal/templates/specifications/tests/MessageTagSpecification.md
[Method4]:                                                      ../Methods.md#method-4-obtain-glue-address-records-from-parent
[Method5]:                                                      ../Methods.md#method-5-obtain-the-name-server-address-records-from-child
[Methods]:                                                      ../Methods.md
[N15_NO_VERSION]:                                               #summary
[N15_SOFTWARE_VERSION]:                                         #summary
[NOTICE]:                                                       ../SeverityLevelDefinitions.md#notice
[RCODE Name]:                                                   https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-6
[Requirements and normalization of domain names in input]:      ../RequirementsAndNormalizationOfDomainNames.md
[RFC2929]:                                                      https://datatracker.ietf.org/doc/html/rfc2929#section-3.2
[RFC7208#3.3]:                                                  https://datatracker.ietf.org/doc/html/rfc7208#section-3.3
[Send]:                                                         #terminology
[Severity Level Definitions]:                                   ../SeverityLevelDefinitions.md
[Test Case Identifier Specification]:                           ../../../../internal/templates/specifications/tests/TestCaseIdentifierSpecification.md
[WARNING]:                                                      ../SeverityLevelDefinitions.md#warning
[Zonemaster-Engine Profile]:                                    ../../../configuration/profiles.md
