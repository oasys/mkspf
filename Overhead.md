## UDP DNS Overhead estimate calculation ##

Here I attempt to calculate the available space for
TXT strings in a UDP packet in a worst-case scenario
(a non-EDNS0 client) to keep the response under the
[RFC1035-defined](https://tools.ietf.org/html/rfc1035#section-2.3.4)
512 byte limit and avoid truncated responses.

--- 

### Header ###

- 2 bytes, id
- 2 bytes, flags
- 2 bytes, number of questions
- 2 bytes, number of answer RRs
- 2 bytes, number of authority RRs
- 2 bytes, number of additional RRs

### Queries ###

Names are encoded per label with a 1-byte length followed by the ASCII
representation of the label.  Each label is encoded this way in turn,
followed by a null byte, indicating the root zone.  As a rule, one can
then consider a name in the format "foo.bar.baz" to be the sum of the
string length (11, here) plus 2, 13 bytes in this case.

- 2 bytes, name (not including length of name)
- 2 bytes, type
- 2 bytes, class

### Answers ###

The data for answer RR's, in this case TXT strings, are encoded each
as a 1-byte length followed by the ASCII representation of the string.
Since this limits a string to a max 255 characters, and our script does
its best to fill the first string before starting a second, we can
assume there will be a max of two strings.  This will reduce the data
size calculation to the length of the strings plus two bytes (one for
each string's length).

We assume that the DNS server is using a compression pointer of two
bytes to reference the offset within the UDP payload to the name in the
Queries section.

- 2 bytes, name
- 2 bytes, type
- 2 bytes, class
- 4 bytes, ttl
- 2 bytes, total data length
- 2 bytes, string length (not including the length of the string itself)

----

### Total ###

- 12 bytes, Header
-  6 bytes, Queries
- 14 bytes, Answers

Added together this gives us 32 bytes estimated overhead.  We use this
number along with the length of the queried name, to subtract from the
maximum UDP payload size, 512 bytes, to find the maximum data size we
can return in a single UDP packet.

For example, for a query of `_1._spf.example.com` (19 bytes), we can
support 461 bytes (512 - 32 - 19) of TXT string data.

----

### Other Approaches ###

Note that the [Record Size section of
RFC7208](https://tools.ietf.org/html/rfc7208#section-3.4) suggests a
more conservative approach of "450 octets" for "the combined length of
the DNS name and the text of all the records of a given type" so that
the response can fit in a UDP packet.

----
