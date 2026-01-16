
# Silent payments tweaked key and BIP-77 Payjoins

Goal: provide methematical intuition for collaboratively creating silent payments outputs for BIP-77 Payjoins and multi-party payjoins.

For now lets assume the recv produces a static payjoin URI including their scan key.
The sender creates the fallback tx with a change output and a standard [BIP 352](https://bips.dev/352/) payment output. Where S is the ECDH shared secret between the sender's public keys and the recv's scan pk.

The receiver upon adding more inputs and finalizing the input set must recalculate the ECDH secret. The math works out nicely here because the receiver already knows the sender's public keys.

The SP tweaked key is then $P = B_{spend} + H(S) G$ where $S = a_{tot} * B_{scan} = b_{scan} * A_{tot}$ and $a_{tot}$ is the sum of all the input keys.
Lets denote the sender keys as $a_s = \sum a_{si}$ , the receiver keys as $a_r = \sum a_{ri}$, and the total keys in the finalized proposal as $a_{tot} = a_s + a_r$.

The receiver upon receiving the proposal and contributing additional inputs, the reciever calculates their share of the tweak as $C_r = a_r * B_{scan}$.
The receiver sends the signs and sends the finalized proposal to the sender alongside $C_r$ and a DLEQ proof that $C_r = a_r * B_{scan}$ and $A_r = a_r * G$ are using the same basepoint scalar ($a_r$).

// TODO: the sender also needs to send $C_s$ and a corresponding DLEQ proof for the receiver to re-calulate $S$. Because the tweaks commit to input ordering this may not after the receiver contributes their inputs.

// TODO; remove *
// TODO: use \prime instead of '

The sender will verify the DLEQ proof and re-calculate the shared secret $S$ and the tweaked key $P'$. Again $S = a_{tot} * B_{scan} = A_{tot} * b_{scan}$.

$$
C_s + C_r = (a_s * B_{scan}) + (a_r * B_{scan}) = (a_s + a_r) * B_{scan} = a_{tot} * B_{scan} = S
$$

## DLEQ proof

The receiver can use any secret scalar point for $C_r$. The sender will need to verify that $C_r = a_r B_{scan}$ and $A_r = a_r * G$ are using the same basepoint scalar ($a_r$).

### Proof generation

Pick a random nonce $k$.
$R_1 = k * G$
$R_2 = k * B_{scan}$
$e = H(R_1 || R_2)$
$s = k + e * a_r$
Return $\pi = (C_r, A_r, e, s)$

### Proof verification

Sender needs to re-calculate $e$ and verify that $e' = e = H(R_1 || R_2)$.
If $e' \neq e$ the proof is invalid.

per [BIP 374](https://bips.dev/374/): re-construct $R_1$ and $R_2$ using .

$R_1 = s * G - e * A_r$
$R_2 = s * B_{scan} - e * C_r$

$$
R_1' = sG - eA_{r} = (k + e a_{r}) G - e A_{r} = k G + e (a_{r} G) - e A_{r} = k G
$$

$$
R_2' = sB_{scan} - eC_r = (k+ea_r)B_{scan} - eC_r = kB_{scan} + e (a_r B_{scan}) - e C_r = kB_{scan}
$$

$e' = H(R_1' || R_2') = e$

## Silent payment tweaks in multi-party payjoins

// TODO: demonstrate that the tweaks should not be using x-only keys and fail under some circumstances. Blinded $C_r$ should be fine to use x-only.

The constraint is that no receiver wants to reveal the silent payments scan key to the other peers. The silent payment receivers will then post a blinded version of their scan key and peers will perform a blinded ECDH to create the aggregate tweak.

First we assume that there exists some aggreement protocol where peers can aggree on the set of inputs in the session. After they sort and finalize the inputs adding outputs can be phased in.

The silent payment scan key is then broadcasted as $B_{scan}' = B_{scan} + rG$ where $r$ is a random blinding factor.

Individual peers then perform a blinded ECDH:
$C_{i}' = a_{i} B_{scan}' = a_{i} (B_{scan} + rG) = a_{i} B_{scan} + a_{i} rG$
$C' = \sum{ C_{i}'} = \sum{a_{i}} B_{scan}' = a_{tot} B_{scan}' = a_{tot} (B_{scan} + rG) = a_{tot} B_{scan} + a_{tot} rG$

DLEQ proofs need to be provided for each $C_{i}'$ and $A_{i} = a_{i} G$ are using the same secret scalar ($a_{i}$).

The peers who knows $B_{scan}$ will unblind each $C_{i}'$ by subtracting the blinding factor $rA_{tot}$ from $C'$.
$C' - rA_{tot} = C' - a_{tot} rG = a_{tot} B_{scan} + a_{tot} rG - a_{tot} rG = a_{tot} B_{scan} = S$

The silent payments tweaked key is then calculated like above and contributed as an output to multi-party session.

Alternatively during output registration all p2tr outputs have blind dh curve points instead of their spending p2tr pubkey. And they are replaced after all the tweaks have been calculated.

## Batched DLEQ proofs

In the case that we have multiple silent payments outputs for each input we need to provide DLEQ proof for each tweak. In coinjoins with many outputs and inputs this can get cumbersome (n inputs, m outputs, O(n * m) DLEQ proofs per session). In a coinjoin with many inputs (>100) this becomes non-trivial. A batched DLEQ proof would be beneficial here to cover all the tweaks for a $a_i$ at once.

More concretly, we still have $A = a_i * G$ but now we have many $C$'s $C_i = B_{scan_{i}} a_i$. Can we cover DLEQ all $C_i$ in a single proof. 

Relevant discussions: https://github.com/bitcoin/bips/pull/1687#discussion_r1847314644
