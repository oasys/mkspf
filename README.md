# mkspf

script to expand/flatten SPF records


### The problem

[RFC7208](https://tools.ietf.org/html/rfc7208#section-4.6.4)
defines a 10-query limit for clients, upon which they will fail open.
With increased delegation (via the include: mechanism) to various cloud
services, the useful scalability of SPF is questionable.  The very act
of delegating control over SPF in this way, allows a third party to
(intentionally or accidentally) break SPF for your domain by, for example,
including multiple levels of SPF that cause you to exceed the limit of
10 DNS lookups.

### A solution

This script will walk through all the include/a/mx directives in a seed
TXT record, along with any local ip4 and ip6 directives, and build a local
list of all the network blocks, formatted for inclusion in a zone file.

## Installation

Consists of a single perl script intended to be run via a commit-hook on
the DNS repository (or manually, if desired).

## Usage

Provide name of the zone file (assumed to be same as domain name) as only
argument on the commandline.  Script will search for a 'mkspf' TXT record
and output a file for inclusion in the main zone.

```bash
$ mkspf.pl ~/dns/bind/namedb/example.com
```

Enter something like this in the `example.com` zonefile:
```zone
@               IN      TXT     "v=spf1 redirect=_spf.example.com"
$INCLUDE _spf.example.com
mkspf           IN      TXT     ("v=spf1"
                                " mx"
                                " ip4:192.0.2.0/24" ; comment
                                " include:example.org include:example.net"
                                " ~all")
```

## TODO

Initially, the script was limited to building a
single <256 character string for each TXT record.
[RFC4408](https://tools.ietf.org/html/rfc4408#section-3.1.3) details a
way to supply multiple <256 character strings in a single TXT record in
order to return more data per RR and still be under the 10-query limit.
The RFC states that "If a published record contains multiple strings,
then the record MUST be treated as if those strings are concatenated
together without adding spaces."

Of courre, we then have to consider limiting to total size so that
the response would be <512 byte UDP limit, to avoid failover to
(often-filtered) TCP with non-EDNS0-capable clients.  See this
[summary](Overhead.md) for how we estimate the maximum size for the TXT
data.

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request :D

## History

- initial commit
- README edits
- support multiple strings per TXT RR

## Credits

Inspiration from other projects:

- [Net::DNS::SPF:Expander](http://search.cpan.org/~amiri/Net-DNS-SPF-Expander/lib/Net/DNS/SPF/Expander.pm)
- [SPFlatten](https://github.com/0x9090/SPFlatten)
- [SPF-tools](https://github.com/jsarenik/spf-tools)
- "Record flattening" feature at [dmarcian SPF Surveyor](https://dmarcian.com/spf-survey/)

## License

GNU GPL
