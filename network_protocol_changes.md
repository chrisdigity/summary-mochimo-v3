# Summary of v3.0 Network Protocol Changes

In v3.0, among other adjustments, the Mochimo Network Protocol has (completely) transitioned to a variable sized Protocol Data Unit (PDU). The PDU header and trailer data remain static in size, however the size of the buffer data can vary, as indicated by the `PDU->len` parameter. This is not too much of an issue for synchronous workloads but, asynchronously, may present the need for additional structure parameters and conditional handling.

The change that remains present over most of these following categories is the use of a LARGE `PDU->buffer`, over multiple fields of data representing the parts of a transaction (something that worked well for handling transactions, but made the handling of other data less intuitive).

NOTE: Asynchronous examples ARE more complex, requiring additional care and error handling over multiple steps.

## Receiving

For receiving data, we must first receive the static length PDU "header" into the PDU struct to determine the length of the buffer data that is being sent via the `PDU->len` parameter. After which we can receive this amount of data, PLUS the size of the trailer (4 bytes) which is also static. This additional trailer data is moved to it's appropriate field, in case the buffer length did not fill the maximum buffer field. After this, the standard PDU structure checks still apply, EXCEPT for the buffer length check. `PDU->len` is an unsigned integer value (word16) only capable of representing a maximum of 65535 bytes. Since the buffer within the PDU structure is capable of holding this amount of data, there is no need for checks. We only recv() that which can be represented in the PDU fields.
- [Synchronous recv() example](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L154-L165)
- [Asynchronous recv() example](https://github.com/mochimodev/mochimo/blob/dca195b626a171f90875f0b595d7b7855f7ad90d/src/protocol.c#L275-L313)
   - async example is intended for use with the network connection handling improvements but may serve as a decent example for similar asynchronous server handling implementations

### Files/Multi-PDU

Very similar to the v2.x loop for receiving files over multiple PDU, except that our EOF check is now based on whether or not our buffer data length is less that the buffer data field, which may be inferred as unsigned integer max constants like `WORD16_MAX`. Since they refer to the appropriate size, sizeof(pdu->buffer) is used in the examples instead.
- [Synchronous example](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L218-L247)
- [Asynchronous example](https://github.com/mochimodev/mochimo/blob/dca195b626a171f90875f0b595d7b7855f7ad90d/src/protocol.c#L578-L623)

## Sending

For sending data, we fill the PDU header as normal (buffer and appropriate len parameter should already be filled), and send the PDU in 2 parts. Firstly, we send the PDU header data and the intended buffer data, per the `PDU->len` parameter. IF this is successful, we perform a crc16() hash, fill the trailer data and send the trailer data through. No additional checks are required, unlike the receive.
- [Synchronous send example](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L264-L289)
- [Asynchronous send example](https://github.com/mochimodev/mochimo/blob/dca195b626a171f90875f0b595d7b7855f7ad90d/src/protocol.c#L360-L384)

### Files/Multi-PDU

Similar to the v2.x implementation, no major changes other than that of the [amount of data](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L370) read into the buffer data.

## OP_NACK

Previously, OP_NACK was not "officially" used for anything. However, it's somewhat beneficial to receive a sort of confirmation or error particularly proceeding a OP_TX request. Not between nodes, but from wallet clients sending an initial transaction. So OP_NACK now serves to send Mochimo specific errors encountered during transaction validation processes. 
- [send_nack() process](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L320-L337)
- [example use of send_nack()](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L414)

## Balances and tag resolutions

Previously these used various fields of the old TX struct that worked ok but were somewhat unintuitive. Now that we have a single buffer field, we instead send the whole LENTRY object that is returned from the search. Being far more lightweight than they once were, LENTRY objects feel more appropriate and simple to just copy into the buffer and send along, OR send a simple NACK with error details where unable to do so.
- [send_balance()](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L402-L418)
- [send_resolve()](https://github.com/mochimodev/mochimo/blob/9fc0c0fa3304cb6b66d47ef82393f165cdcabb41/src/network.c#L502-L516)
