# skyPC fork of sctp-proto

Branch `skypc-v0.10.6` is upstream sctp-proto 0.10.6 (`2039e04`) plus
[webrtc-rs/sctp-proto#57](https://github.com/webrtc-rs/sctp-proto/pull/57),
which fixes premature PR-SCTP abandonment, and two congestion fixes found
streaming bursty video over a zero-retransmit channel:

1. **Fix premature PR-SCTP abandonment and fragment cleanup** and
   **Simplify shared fragment state to sender-only atomic flags** (#57 as
   submitted). Zero-retransmit DATA was abandoned on its first send, so every
   SACK triggered a FORWARD-TSN over data still in flight, and Chrome answered
   each with another SACK. On a WebRTC datagram channel at ~28 Mbps that was
   ~7,700 FORWARD-TSNs/s and ~18% extra UDP; with the fix FORWARD-TSNs follow
   only real losses and the overhead is ~6% (headers).
2. **Adapt the FORWARD-TSN wraparound test to shared message state.** The
   0.10.6 wraparound test used fields #57 replaces.
3. **Repeat a FORWARD-TSN only when it advances or a round trip passed.**
   Every SACK the peer sent before the last FORWARD-TSN reached it asked for
   another, and Chrome answers each FORWARD-TSN with a SACK, so after a loss
   burst the peers traded ~20,000 packets/s each way and no DATA left for
   400 ms.
4. **Count Fast Recovery misses and exit as RFC 9260 Sec 7.2.4 says.** Misses
   were not counted in Fast Recovery unless the Cumulative TSN Ack Point
   advanced, so a TSN lost during it waited for T3-rtx (1 s, cwnd to one MTU);
   and Fast Recovery ended at the highest TSN newly acknowledged instead of the
   highest outstanding, halving cwnd again for the same losses.

Drop this fork once upstream 0.10.x releases contain #57 (or the equivalent
#58) and fixes for 3 and 4; str0m's `sctp-proto = "0.10.5"` requirement accepts
them without a str0m release.
