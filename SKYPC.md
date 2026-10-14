# skyPC fork of sctp-proto

Branch `skypc-v0.10.6` is upstream sctp-proto 0.10.6 (`2039e04`) plus
[webrtc-rs/sctp-proto#57](https://github.com/webrtc-rs/sctp-proto/pull/57),
which fixes premature PR-SCTP abandonment:

1. **Fix premature PR-SCTP abandonment and fragment cleanup** and
   **Simplify shared fragment state to sender-only atomic flags** (#57 as
   submitted). Zero-retransmit DATA was abandoned on its first send, so every
   SACK triggered a FORWARD-TSN over data still in flight, and Chrome answered
   each with another SACK. On a WebRTC datagram channel at ~28 Mbps that was
   ~7,700 FORWARD-TSNs/s and ~18% extra UDP; with the fix FORWARD-TSNs follow
   only real losses and the overhead is ~6% (headers).
2. **Adapt the FORWARD-TSN wraparound test to shared message state.** The
   0.10.6 wraparound test used fields #57 replaces.

Drop this fork once an upstream 0.10.x release contains #57 or the equivalent
#58; str0m's `sctp-proto = "0.10.5"` requirement accepts it without a str0m
release.
