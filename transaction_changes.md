# Summary of v3.0 Transaction Changes

For the most part, many of the traditional transaction fields remain. Some underwent a dramatic reduction in size (src/dst/chg addresses), and some were split up for additional functionality (signature, seed, adrs). There are also some additional fields for function and security.

## TXQENTRY Struct

*Reference: [TXQENTRY](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/types.h#L431-L451)*

Documentation has been commented above the struct for noteworthy changes if you prefer.

### Extendable dstaddr/xdata

By far the most complicated part of the transaction...
With address sizes reduced to 44 bytes, this presented an issue with maintaining compatibility with Multi-Destination (MDST) Transactions which relied on the sheer size of an address to simply fill with many tags and balances. In an attempt to retain this feature and perhaps provide an avenue for future "extended" data features, transactions will vary in length. Simple transactions will contain only the basic Transaction Entry Data and where specified by the dstaddr field as being an eXtended Transaction (XTX) type, per legacy terminology, an additional eXtended Data (XDATA) field will be present after the dstaddr field. Per the comment [documentation](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/types.h#L383-L405) the type of data, and size of this field will vary depending on the values present in the dstaddr.

*NOTE: the byte positions are subject to change if we decide tags should precede address (simple change)*

### New tx_btl

Transaction block-to-live (live; as in the *verb*) is a simple expiration indicator, where the transaction is no longer valid after this specified block number, EXCEPT where set to zero for no expiration.

### Split signature

Since the Ledger no longer holds onto the WOTS+ Public Seed (SEED) and the WOTS+ Address Scheme (ADRS), they must be written into the transaction for validation purposes. These are WOTS+ specific. So to prevent wasted space for when we eventually adapt multiple Digital Signature Algorithms (DSA), we can actually reuse at least part of these fields to identify these things. Nothing is set in stone but examples of what this MIGHT look like are documented [here](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/types.h#L415-L423).

### New tx_nonce

The transaction nonce ensures the integrity of unique transaction IDs (within reasonable doubt) by always containing the block number of the block it is solved into. Perhaps the word nonce is then not overly accurate, as the nonce won't be unique between transactions of the same block, but in terms of serving it's purpose, this was the most certain way.

### Node-ONLY fields

tx_nonce and tx_id are only handled by the Node only. Filling these fields and adjusting the buffer length of the PDU to accomodate, does nothing. The Node will generate it's own copies of these fields.

## Multi-Destination Type Transaction

I don't think I can summarize better than I already have [here](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/types.h#L383-L405).
