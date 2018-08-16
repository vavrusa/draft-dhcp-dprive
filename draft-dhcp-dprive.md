% Title = "Dynamic Host Configuration Protocol (DHCP) Domain Name Server Capability Option"
% abbrev = "dns-dhcp-dprive"
% category = "std"
% docName = "draft-vavrusa-net-dns-dhcp-dprive-00"
% ipr= "trust200902"
% area = "Internet"
% workgroup = "Network Working Group"
% keyword = ["DNS", "NET"]
%
% date = 2018-08-16T00:00:00Z
%
% [[author]]
% initials="M."
% surname="Vavrusa"
% fullname="Marek Vavrusa"
% #role="editor"
% organization = "Cloudflare Inc."
%   [author.address]
%   email = "mvavrusa@cloudflare.com"
%   [author.address.postal]
%   street = "101 Townsend St."
%   city = "San Francisco"
%   code = "94107"
%   country = "USA"

.# Abstract

This document describes Dynamic Host Configuration Protocol options for passing a list of DNS recursive name server capabilities to a client.

{mainmatter}

# Introduction

This document describes an option for passing additional configuration information related to Domain Name Service (DNS) ([@!RFC1034] and [@!RFC1035]) in DHCPv6 ([@RFC3315]).

## Requirements

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in [@RFC2119].

## Terminology

The reader is assumed to be familiar with the basic DNS concepts described in [@!RFC1034], [@!RFC1035], [@!RFC2181] and [@!RFC6891]. The DNS recursive name server option is defined in [@!RFC3646] for DHCPv6, and [@!RFC2132] for DHCPv4.

The DHCPv6 and DHCPv4 are commonly referred to as DHCP.

# DNS Recursive Name Server Capability option

The DNS Recursive Name Server Capability option provides a list of capabilities for one or more DNS recursive name server addresses as specified in the DNS Recursive Name Server option. The order of entries corresponds to the order of name servers as per [@!RFC3646] or [@!RFC2132] DNS Recursive Name Server option.

Each capability list can contain zero or more capabilities.

The format of the DNS Recursive Name Server Capability option is:

{#fig-capability-option type="ascii-art" align=center}

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          option-code          |         option-len            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           list-count          |       capability-code         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |            data-len           |            data ...           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                              ...                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

   option-code:               OPTION_DNS_SERVER_CAPABILITIES (147)

   option-len:                Length of the list of DNS recursive name
                              server capabilities in octets; each capability
                              list has a variable length.
   list-count:                Number of entries in the capability list.
   capability-code:           Capability entry variant that defines the type of the entry.
   data-len:                  Length of the capability variant data in octets.
   data:                      Optional data for the capability entry.

Legal values for the capability-code options are:

{#fig-capability-code type="ascii-art" align=center}

           Value   Message Type
           -----   ------------
             1     TLS_SPKI
             2     TLS_ADN
             3     HTTPS

## The TLS_SPKI capability code

If this option is present, the associated DNS recursive name server supports [@!RFC7858] DNS over Transport Layer Security with SPKI key pinning, as per [@!RFC7858], section 4.2. The data-len MUST be set to 32 (length of the [@!RFC6234] SHA-256 hash), and the data must contain the SHA-256 hash of the DER-encoded ASN.1 representation of the SPKI of an X.509 certificate.

## The TLS_ADN capability code

If this option is present, the associated DNS recursive name server supports [@!RFC7858] DNS over Transport Layer Security with ADN + IP authentication, as per [@!RFC8310], section 6.2. The data-len MUST bet between 0 and 255, and the data must contain a [@!RFC1035] Authentication Domain Name in wire format.

## The HTTPS capability code

This capability SHOULD have data-len set to 0. If this option is present, the associated DNS recursive name server supports draft-ietf-doh-dns-over-https DNS Queries over HTTPS (DoH) using [@!RFC5280] X.509 Public Key Infrastructure (PKI) for the Internet.

# Appearance of these options

The DNS Recursive Name Server option MUST NOT appear in any other than the following messages: Solicit, Advertise, Request, Renew, Rebind, Information-Request, and Reply.

# Security Considerations

The DNS Recursive Name Server Capability option may be used by an intruder DHCP server to cause DHCP clients to send DNS queries to an intruder DNS
recursive name server, or prevent clients from verifying the authenticity of secured DNS recursive name servers.

To avoid attacks through the DNS Recursive Name Server Capability option, the DHCP client SHOULD require DHCP authentication (see section "Authentication of DHCP messages" in [@!RFC3315]) before installing a list of DNS recursive name servers obtained through authenticated DHCP.

Security considerations from [@!RFC3315], section 23 apply.

The guidelines for client use of encrypted DNS protocol capabilities are described in [@!RFC8310].

# Examples

## DNS over TLS server offering both TLS_SPKI and TLS_ADN


{#fig-dot-example-1 type="ascii-art" align=center}

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |               147             |         option-len            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                2              |          TLS_SPKI             |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |            PZXN3lRAy+8tBKk2Ox6F7jIlnzr2Yzmwqc3JnyfXoCw=       |
   |                   (base64 encoded for posterity)              |
   |                                                               |
   |                                                               |
   |                                                               |
   |                                                               |
   |                                                               |
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                TLS_ADN        |       \7example\3com\0        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


# IANA Considerations

TODO: IANA has assigned an option code to the DNS Recursive Name Server Capability option (147) from the DHCPv6 option code space defined in section "IANA Considerations" of [@!RFC3315], and from the DHCPv4 option code space defined in section "Defining new extensions" of [@!RFC2132].

# Acknowledgements

# Notes

{backmatter}
