
# Silent payments tweaked key and BIP-77 Payjoins

Goal: provide methematical intuition for collaboratively creating silent payments outputs for BIP-77 Payjoins.

For now lets assume the recv produces a static payjoin URI including their scan key.
The sender creates the fallback tx with a change output and a standard [BIP 352](https://bips.dev/352/) payment output. Where S is the ECDH shared secret between the sender's public keys and the recv's scan pk.

The receiver upon adding more inputs and finalizing the input set must recalculate the ECDH secret. The math works out nicely here because the receiver already knows the sender's public keys.

The SP tweaked key is then $P = B_{spend} + H(S) G$ where $S = a_{tot} * B_{scan} = b_{scan} * A_{tot}$ and $a_{tot}$ is the sum of all the input keys.
Lets denote the sender keys as $a_s = \sum a_{si}$ , the receiver keys as $a_r = \sum a_{ri}$, and the total keys in the finalized proposal as $a_{tot} = a_s + a_r$.

The receiver upon receiving the proposal and contributing additional inputs, the reciever calculates their share of the tweak as $C_r = a_r * B_{scan}$.
The receiver sends the signs and sends the finalized proposal to the sender alongside $C_r$ and a DLEQ proof that $C_r = a_r * B_{scan}$ and $A_r = a_r * G$ are using the same basepoint scalar ($a_r$).

// TODO: the sender also needs to send $C_s$ and a corresponding DLEQ proof for the receiver to re-calulate $S$. Because the tweaks commit to input ordering this may not after the receiver contributes their inputs.

The sender will verify the DLEQ proof and re-calculate the shared secret $S$ and the tweaked key $P'$. Again $S = a_{tot} * B_{scan} = A_{tot} * b_{scan}$.

$$
C_s + C_r = (a_s * B_{scan}) + (a_r * B_{scan}) = (a_s + a_r) * B_{scan} = a_{tot} * B_{scan} = S
$$

## DLEQ proof

The receiver can use any secret scalar point for $C_r$. The sender will need to verify that $C_r = a_r * B_{scan}$ and $A_r = a_r * G$ are using the same basepoint scalar ($a_r$).

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
