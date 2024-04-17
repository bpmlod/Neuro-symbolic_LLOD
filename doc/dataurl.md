# use Data URL to encode embeddings for persistent storage/exchange

- allows to encode arbitrary data in a URI
- not (meant to be) resolvable, because it just provides all the information explicitly encoded in the URL
- originally intended for including image data as part of an HTML file
- not a formal standard, but supported by most browsers
- https://datatracker.ietf.org/doc/html/rfc2397
  - basic schema `data:[<mediatype>][;base64],<data>`
  - `<mediatype>`: as defined by https://datatracker.ietf.org/doc/html/rfc6838
  - `base64`: for base64 encoding, otherwise, the standard `%xx` hex encoding is used
- https://datatracker.ietf.org/doc/html/rfc6838, mediatypes and mediatype registration
  - no specific mediatype for embeddings, but we would propose one: https://www.iana.org/form/media-types
  - unless we have anything more domain-specific, we'd have to use [`application/octet-stream`](https://www.rfc-editor.org/rfc/rfc2046.html#section-4.5.1)
    - this doesn't say anything about the actual type of data, basically means "uninterpreted binary data"
    - `application/octet-stream` has an optional type parameter intended to provide a human-readable label. we can (ab)use that to identify/declare/spot embeddings and distinguish them from anything else
      - not sure how this looks like, exactly, maybe `application/octet-stream;TYPE=list-of-64bit-floats` or `application/octet-stream;TYPE=list-of-32bit-floats` ?
    - `application/octet-stream`
    - it could be preferrable to encode the name of the knowledge graph that an embedding belongs to. in an older version ([RFC 1341, §7.4](https://datatracker.ietf.org/doc/html/rfc1341)), there was a parameter `NAME=` to provide the name of a file to be saved, but this has been deprecated. If recsurrected (by us, for this purpose), it could be used to provide the name of the embedding space, instead.
- base64: https://datatracker.ietf.org/doc/html/rfc4648#section-4
  - use (64) US-ASCII characters to encode 6 bits by character
  - if necessary, use `=` as padding characters

for converting an array of 16-bit floats into a base64 encoding:
- convert each number individually to a base
  - break the (16 bit) float into 3 chunks of 6 bits (i.e., 18 bits, so add two `0` bits before the numerical value)
  - map each 6bit group to an ASCII/base64 character
- concatenate the base64 characters
- for a list of 50 16-bit floats, we need 50 x 18 / 6 = 150 characters/bytes.

> note that this will only work for plain list of floats (or doubles, or halves, etc.). If we have tensors (lists of lists of (etc.) floats), we need to encode the dimensionality directly. this could not be done with current RFC conventions.

alternative conversion:
- convert every float to a sequence of bits
- concatenate all bits into a sequence
- group each group of 6 continuous bits
- map each group to an ASCII/base64 character

for a list of 50 16-bit floats, we need 50 x 16 / 6 = 134 characters/bytes

> the alternative conversion will is about 10% more compact, but is (IMHO) less easily interpretable/processable

## PRO and CON

- PRO
  - fairly compact encoding, using a widely used community (but not a formal) standard, well-suited for exchange
  - in particular if compared with encoding as JSON array or plain text (as in CSV)
    - CSV/TSV: 50 16-bit floats ~ 50 x 5 (= `.` followed by up to four numbers) characters plus separator
      - 50x5+49 = 299 characters/bytes, so we have a reduction by 50%
  - nothing stops us from using this in conjunction with FrAC embeddings (then, just without the `rdf:value`)
- CON, not solvable without introducing a new mediatype
  - not human-readable
    - this is a feature, not a bug ;)
  - we abuse existing conventions, in particular, tools would need how to spot "our" `application/octet-stream` implementations
    - can be solved be introducing a new mediatype
  - we can only encode vectors, not matrices or vectors
    - can be solved by introducing a dimensionality attribute, but this requires registering/getting approval for a new mediatype
  - not as compact as exchanging plain binary data
    - 50 16-bit floats are 100 bytes, so base64 encoding is about 50% larger than that
- CON, solvable without introducing a new mediatype
  - data URLs do not provide provenance information. in particular, we cannot specify that one embedding comes from one embedding space/tool, and one from the other
    - maybe, this is not actually an issue, uin combination with, say FrAC, we can provide exactly that information
  - because there are different encoding strategies, developers can be tempted to use the native `base64` encoding of their programming language, and this might be different from the default encoding we specify
    - in order to address this, we could add an **obligatory** checksum byte
    
