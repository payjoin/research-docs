# Fuzzy Message Detection (FMD)

Note: these are Armin's personal notes / scratch pad.

Protocols that utilize store-and-forward servers / bulletin boards often require the
offline user to download and scan the entire dataset to find messages relevant to them.
Consider a system like [blindbit](https://github.com/blindbitbtc) where BIP352 tweaks
are precomputed and a paginated version is served to the user. Users compute the ECDH
and filter for transactions that belong to them. Alternatively, users can delegate the
filtering responsibility to a trusted server (viewing keys in zcash). Trusting it with both their privacy and
liveness.

Instead, we want to delegate only a *probabilistic* detection capability to the server.
Analogous to bloom filters: the server identifies all messages to a particular receiver
with no false negatives, but also picks up some unrelated messages at a tunable false
positive rate. This is exactly what [FMD](https://eprint.iacr.org/2021/089) claims to achieve.

This is especially relevant to the silentpayjoins work, where the BIP-77 directory
could serve as the FMD store-and-forward server.

Spoiler: FMD is extremely fragile. The params need to be chosen carefully. Even then the privacy guarantees are
questionable at best: <https://github.com/arminsabouri/bitcoin-papers/blob/master/privacy/pir/fmd-analysis.pdf>

## Details

For the rest of this document we assume a silentpayjoin setting. Sender and receiver
map to BIP-77 nomenclature and the detector store-and-forward server is the BIP-77 directory.

### Setup

The receiver generates a root secret $z$ and a set of keypairs:

$$k_i = H(z \| i), \quad K_i = k_i G \quad \text{for } i = 1, \ldots, n$$

The set of public keys is known as the **clue key** $\mathsf{ck} = (K_1, \ldots, K_n)$.

The **detection key** at precision $\gamma$ is $\mathsf{dk}(\gamma) = (k_1, \ldots, k_\gamma)$ where $\gamma < n$.

The receiver hands $\mathsf{dk}(\gamma)$ to the directory and keeps $z$ private.
Note that the directory cannot derive the remaining flag secrets $k_{\gamma+1}, \ldots, k_n$
from $\mathsf{dk}(\gamma)$ alone.

The receiver also makes $\mathsf{ck}$ available to senders. It is fully public and
can be posted openly along side the static payjoin URI.

### Sender

The sender samples a fresh ephemeral scalar $r$ and computes:

$$R = r \cdot G$$

For each $i = 1, \ldots, \gamma$, the sender computes one bit via ECDH against each
flag public key $K_i$:

$$u_i = H(r \cdot K_i) \bmod 2$$

The clue sent to the directory alongside the HPKE-encrypted silent payjoin proposal is:

$$c = (R,\ u_1,\ \ldots,\ u_\gamma)$$

### Detection

The directory holds the detection key $\mathsf{dk}(\gamma) = (k_1, \ldots, k_\gamma)$
and receives clues from senders. When the receiver comes online and requests relevant
messages, the directory runs the following test on every stored clue
$c = (R, u_1, \ldots, u_\gamma)$.

For each $i = 1, \ldots, \gamma$, the directory computes:

$$\hat{u}_i = H(k_i \cdot R) \bmod 2$$

This is ECDH between the ephemeral point $R$ and each flag private key $k_i$. The key
identity that makes the scheme work is:

$$k_i \cdot R = k_i \cdot r \cdot G = r \cdot k_i \cdot G = r \cdot K_i$$

So an honest sender's $u_i$ and the directory's $\hat{u}_i$ are computing the **same
value** from opposite sides. This guarantees that every real message is always flagged.

The directory flags the message iff all bits agree:

$$\forall\, i = 1, \ldots, \gamma, \quad \hat{u}_i = u_i$$

The receiver then downloads all flagged messages and decrypts locally.

### False Positives

For a random unrelated message, the sender did not use the flag keys to set $u_i$, so
the directory's $\hat{u}_i$ and the clue's $u_i$ are independent uniform bits. They
agree by coincidence with probability:

$$\Pr[\text{flag} \mid \text{random}] = \prod_{i=0}^{\gamma} \frac{1}{2} = 2^{-\gamma} =: p$$

The security claim is that the directory's view of a flagged clue is computationally
indistinguishable between a true positive and a false positive. Holding only
$(k_1, \ldots, k_\gamma)$ and not the receiver's private key $b$, the directory cannot
distinguish the two cases on any individual clue.

## Chosen Ciphertext Attack (CCA)

Since the querying mechanism is not authenticated an advesary (Eve) can craft payloads and query the directory for them. Because Eve knows the receiver's clue keys she can continue to probe the directory for payloads that belong may belong to the receiver by iterating on $R_i$.

If querying is left unauthenticated any sufficiently motivated advesary can probe the directory. FMD2 aims to solve the unbounded ciphertext query issue by binding the clue to the receiver's key. This mechanism introduces a more crypto machinary. The notes above are just on FMD1.

### Tying it all together

This scheme reduces the scanning requirement from $O(N)$ to $O(N2^{-\gamma})$. This is presumably a natural incentive
for any BIP-352 receiver to prefer silentpayjoins over delegated scanning or something like blindbit. Other notification based
systems proposals, such as, using [nostr](https://delvingbitcoin.org/t/silent-payments-notifications-via-nostr/2203) but if [network metadata](https://github.com/nostr-protocol/nips/pull/2088)
is not protected carefully then the whole system will have leaks.

$\gamma$ must be carefully chosen to balance bandwidth and privacy. A larger $\gamma$
means fewer false positives ($p = 2^{-\gamma}$ shrinks), so the receiver downloads less
;however,    the directory gains more statistical inference power over the receiver's activity.
A smaller $\gamma$ floods the directory with noise, protecting privacy at the cost
of the receiver downloading more data.

A mobile user may prefer a smaller $\gamma$ to preserve privacy, accepting the bandwidth
overhead as the price of keeping the directory in the dark.

SilentPayjoins senders re-uses the receiver's scan key to encrypt their proposal. A receiver could use
their scan key as the root master key ($z$).
