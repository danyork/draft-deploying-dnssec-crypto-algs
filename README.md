# Observations on Deploying New DNSSEC Cryptographic Algorithms

This repo is for the development of an Internet-Draft related to deploying
new cryptographic algorithms within DNSSEC.  The latest submitted version 
can be found at:

https://datatracker.ietf.org/doc/draft-york-dnsop-deploying-dnssec-crypto-algs/

or

https://tools.ietf.org/html/draft-york-dnsop-deploying-dnssec-crypto-algs

This draft has been discussed on the DNSOP mailing list and has been presented at the
following IETF meetings:

- IETF 96, July 2016, Berlin
- [IETF 97](https://www.ietf.org/proceedings/97/slides/slides-97-dnsop-deploying-new-dnssec-cryptographic-algorithms-00.pdf), November 2016, Seoul


----

## Building/editing this document

This Internet-Draft is built using the `mmark` tool created by Miek Giebben and the Docker container created by Paul E. Jones to make using `mmark` and `xml2rfc` much easier.

If you have Paul Jones `rfctools` Docker container installed all you should have to do is type:

`make`

and both the necessary XML and TXT files will be generated.

Paul's Docker image can be found at:

https://hub.docker.com/r/paulej/rfctools/

and more information can be found at:

https://github.com/paulej/rfctools
https://github.com/miekg/mmark