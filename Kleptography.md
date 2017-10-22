# Kleptography

## Introduction

Kleptography is defined in the paper as "the study of stealing information securely
and subliminally."  We'll be talking about how "black box" devices like USB tokens
of chip cards can be designed to leak their keys in such a way that only the designer
can recover them - or indeed know that keys are being leaked at all.


## SETUPs

A **S**ecretly **E**mbedded **T**rapdoor with **U**niversal **P**rotection is an
algorithm that can be embeddded in a cryptosystem to leak secret key information
to the attacker through the output of the cryptosystem.  In general, a SETUP:

- Has input and output polynomially indistinguishable from that of the uncompromised system.
- Does not contain the attacker's decryption function or decryption key.
- Leaks bits of the user's secret key in its output.
- Even with full knowledge of the SETUP, only the attacker can determine past or future keys.

A "strong" SETUP is more powerful than a regular SETUP.  Consider a cryptosystem
that  uses a SETUP mechanism  with 50% probability  and the unmodified algorithm
with 50% probability.  In a strong SETUP, a user would not be able to tell which
algorithm was used.


### Leakage Bandwidth

Leakage bandwidth refers to the number of keys that can be leaked  per number of
output messages. A SETUP that leaks one key every four messages would be said to
have (1, 4)-leakage.


## A Discrete Log Based SETUP against Diffie-Hellman

### Diffie-Hellman Key Exchange

Suppose Alice and Bob need to agree  on a secret key,  but  can only communicate
over a public channel.  The Diffie-Hellman protocol lets them do this:

- Assume a public large strong prime _p_.
- Assume a public _g_ which is a generator mod _p_.
- Alice generates a random value _a_ < _p_ - 1.
- Bob generates a random value _b_ < _p_ - 1.
- Alice sends Bob _g_<sup>_a_</sup> mod _p_.
- Bob sends Alice _g_<sup>_b_</sup> mod _p_.
- Both of them can now compute a shared secret _k_ = _g_<sup>_ab_</sup> mod _p_.


### The Discrete Log Attack

In the above protocol, the only information that Alice sends over the network is
_g_<sup>_a_</sup> mod _p_.  How can she leak her secret _a_ to an accomplice?

Assume  Alice and Eve  have agreed on a few extra numbers beforehand  (there are
some extra requirements for _A_ and _B_ that we won't get into here):
- Eve's public key _g_<sup>_x_</sup>.
- An odd number _W_.
- Some number _A_.
- Some number _B_.

Alice can leak her  _second_  secret _a_ as follows. The first time she performs
the protocol:

- She chooses _a_<sub>1</sub> < _p_ - 1 uniformly at random.
- She sends Bob _m_<sub>1</sub> = _g_<sup>_a_<sub>1</sub></sup> mod _p_.
- She stores _a_<sub>1</sub> to use again next time.

The second time through the protocol:

- She chooses _t_ randomly from {0, 1}.
- She calculates _z_ = _g_<sup>_a_<sub>1</sub> - _Wt_ - _Axa_<sub>1</sub> - _Bx_</sup>.
- She calculates _a_<sub>2</sub> = H(_z_).
- She sends Bob _m_<sub>2</sub> = _g_<sup>_a_<sub>2</sub></sup> mod _p_.

Eve can just watch the messages passing through the network.  She can observe values
_m_<sub>1</sub> and _m_<sub>2</sub> and Bob's second response, _g_<sup>_b_<sub>2</sub></sup>.
These are all she needs to solve for _a_<sub>2</sub> and use it to calculate Alice
and Bob's second shared secret key, _k_<sub>2</sub>:

- She calculates _r_ = _m_<sub>1</sub><sup>_A_</sup>_g_<sup>_B_</sup> mod _p_.
  - _r_ = _m_<sub>1</sub><sup>_A_</sup>_g_<sup>_B_</sup> mod _p_
  - _r_ = _g_<sup>_Aa_<sub>1</sub></sup>_g_<sup>_B_</sup> mod _p_
  - _r_ = _g_<sup>_Aa_<sub>1</sub> + _B_</sup> mod _p_
- She calculates _z_<sub>1</sub> = _m_<sub>1</sub>_r_<sup>-_x_</sup> mod _p_.
  - _z_<sub>1</sub> = _m_<sub>1</sub>_r_<sup>-_x_</sup> mod _p_.
  - _z_<sub>1</sub> = _g_<sup>_a_<sub>1</sub></sup>_g_<sup>-_Axa_<sub>1</sub> - _Bx_</sup> mod _p_.
  - _z_<sub>1</sub> = _g_<sup>_a_<sub>1</sub> - _Axa_<sub>1</sub> - _Bx_</sup> mod _p_.
- If _m_<sub>2</sub> = _g_<sup>H(_z_<sub>1</sub>)</sup> mod _p_:
  - Then Eve has found _a_<sub>2</sub> = H(_z_<sub>1</sub>).
- Otherwise she calculates _z_<sub>2</sub> = _z_<sub>1</sub>_g_<sup>-_W_</sup>.
  - _z_<sub>2</sub> = _z_<sub>1</sub>_g_<sup>-_W_</sup> mod _p_
  - _z_<sub>2</sub> = _g_<sup>_a_<sub>1</sub> - _Axa_<sub>1</sub> - _Bx_</sup>_g_<sup>-_W_</sup> mod _p_
  - _z_<sub>2</sub> = _g_<sup>_a_<sub>1</sub> - _W_ - _Axa_<sub>1</sub> - _Bx_</sup> mod _p_
- If _m_<sub>2</sub> = _g_<sup>H(_z_<sub>2</sub>)</sup> mod _p_:
  - Then Eve has found _a_<sub>2</sub> = H(_z_<sub>2</sub>).
- Eve can now calculate _k_<sub>2</sub> = (_g_<sup>_b_<sub>2</sub></sup>)<sup>_a_<sub>2</sub>.

Only Eve can do this, because only she knows her secret key, _x_.

