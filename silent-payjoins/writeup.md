# Reducing BIP-77 by one round of communication and improving UX with Silent Payments

Async Payjoin (BIP-77) uses a scan-and-send interaction model. The sender scans the receiver’s BIP21 address, which encodes the store-and-forward server endpoint that the receiver monitors. This design enables asynchronous communication. However, the sender must request a fresh BIP21 address for every payment. This document specifies the changes required to support static Payjoin URIs using Silent Payments, referred to here as **SilentPayjoins**.

<!-- A sender capable of both Silent Payments and Payjoin with a compatible receiver (Cake wallet for example) must currently choose between two protocols. Typical users cannot reliably distinguish between these options or evaluate their tradeoffs. -->
<!-- 
Both protocols should coexist. Payjoin can incorporate Silent Payments in a clean and composable way. -->

## Static Payjoin URI

The receiver encodes their scan key, spend key, and directory endpoint into a BIP21 URI. This construction yields a static Payjoin URI.

Example BIP21: `bitcoin:sp1q...?&pj=https://directory.example.com`

## Changes to BIP77 Directory

The directory must not learn the financial history of users who reuse a static Payjoin URI. In BIP-77 terminology, reusing the same subdirectory or mailbox introduces longitudinal privacy leakage.

The BIP-77 directory should instead operate a rate-limited, expiring bulletin board for all SilentPayjoin proposals. A SilentPayjoin-capable client downloads the entire bulletin board and locally scans for proposals "addressed" to them. Silent Payjoin proposals are HPKE-encrypted using the same scan key associated with the static Payjoin URI. This process is anologous to how silent payments light clients scan the network for payments.

Clients persist retrieved proposals and periodically synchronize with the directory to obtain updates. The directory should support time-range pagination as not to download the entire set.

The HPKE payload does not contain the proposal itself, but a pointer to the subdirectory containing the proposal.

// TODO: size of the HPKE payload?

Anonymous credential-based rate limiting provides flood protection.

This design does not require PIR. The directory can return the complete proposal set, and clients perform filtering locally.

## The fallback

The BIP-77 fully signed fallback transaction becomes a BIP-352 transaction constructed with the receiver’s scan and spend keys.

If the sender supports BIP-352 but not SilentPayjoin, they broadcast a BIP-352 transaction.

## Collaboratively creating the tweak

In BIP-352, the output key derives from a tweak computed via the ECDH shared secret between the input private keys and the receiver’s scan public key. In Payjoin, both sender and receiver contribute inputs, so they must collaboratively compute the shared secret because neither party controls all input private keys.

The Silent Payment tweaked key is $P = B_{spend} + H(S) G$ where $S = a_{tot} B_{scan} = b_{scan} A_{tot}$ and $a_{tot}$ represents the sum of all input private keys. Let the sender keys be $a_s$, the receiver keys be $a_r$, and the total keys in the finalized proposal be $a_{tot} = a_s + a_r$. And in the fallback case, $a_{tot} = a_s$ as the receiver has not added inputs.

The receiver verifies correct BIP-352 output construction and caches the resulting txid to reduce on-chain scanning requirements -- asuming only segwit inputs are used.

The receiver contributes inputs of their own and computes their tweak contribution as $C_r = a_r B_{scan}$, and includes it under `PSBT_IN_SP_TWEAK_CONTRIBUTION` in the payjoin PSBT. The receiver signs and returns the finalized proposal to the sender along with $C_r$ and a DLEQ proof demonstrating that $C_r = a _r B_{scan}$ and $A_r = a_r G$ share the same scalar $a_r$. The proof would be included under `PSBT_IN_SP_DLEQ_PROOF`.
The receiver calculates the response mailbox id as the truncated hash of the sender's reply key -- this is not different from the response mailbox id in BIP-77.

The sender must verify the DLEQ proof, recompute the shared secret $S$ using the receiver's $C_r$, and derive the tweaked key $P'$.

$$
C_s + C_r = (a_s B_{scan}) + (a_r B_{scan}) = (a_s + a_r) B_{scan} = a_{tot}  B_{scan} = S
$$

If all verifications succeed, the sender signs and broadcasts the collaborative transaction.
