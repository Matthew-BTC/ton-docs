# External messages


External messages are messages that are `sent from outside the TON Blockchain` to the smart contracts running on TON to make them perform specific actions. 

For instance, a wallet smart contract expects to receive external messages containing orders (e.g., internal messages to be sent from the wallet smart contract) signed by the wallet's owner. When the wallet smart contract receives such an external message, it first checks the signature, then accepts the message (by running the TVM primitive `ACCEPT`), and then performs the required actions.

:::caution
Note that all external messages `must be protected` against replay attacks. The validators usually remove an external message from the pool of suggested external messages (received from the network); however, in some situations, `another validator` could process the same external message twice. This would result in creating a second transaction for the same external message, duplicating the original action. Even worse, a `malicious actor could extract` the external message from the block containing the processing transaction and re-send it later. For example, this could force a wallet smart contract to repeat a payment.
:::

export const Highlight = ({children, color}) => (
<span
style={{
backgroundColor: color,
borderRadius: '2px',
color: '#4a080b',
padding: '0.2rem',
}}>
{children}
</span>
);


The <Highlight color="#ffeced">simplest way to protect smart contracts from replay attacks</Highlight> related to external messages is to store a 32-bit counter `cur-seqno` in the persistent data of the smart contract and to expect a `req-seqno` value in (the signed part) any inbound external messages. The external message is accepted only if the signature is valid and `req-seqno` equals `cur-seqno`. After successful processing, the `cur-seqno` value in the persistent data is increased by one, so the <Highlight color="#ffeced">same external message will never be accepted again</Highlight>.

<Highlight color="#ffeced">One could also</Highlight> include an `expire-at` field in the external message and accept an external message only if the current Unix time is less than the value of this field. This approach can be used in conjunction with `seqno`; alternatively, the receiving smart contract could store the set of (the hashes of) all recent (not expired) accepted external messages in its persistent data and reject a new external message if it is a duplicate of one of the stored messages. To avoid bloating the persistent data, garbage collection of expired messages in this set should be performed. 

:::note
In general, an external message begins with a 256-bit signature (if needed), a 32-bit `req-seqno` (if needed), a 32-bit `expire-at` (if needed), and possibly a 32-bit `op` and other required parameters depending on the `op`. The layout of external messages does not need to be as standardized as internal messages because external messages are not used for interaction between smart contracts written by different developers and managed by different owners.
:::