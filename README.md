# wormhole
UDP/TCP wormhole with master/slave routing capabilities, encryption, multipath and hole-punching

### Desired features:
- UDP and TCP
- Hole-punching
- Direct communication between clients (if possible)
- Encryption
- Simulate packet loss, latency, jitter, data alteration (for UDP)
- Congestion control (clients should be able to report when maximum throughput is reached)
- All the configuration should be done in one file on server - config should be reloaded when saved
- Support 1 to 1, 1 to N, N to 1, N to N modes
- Clients should send from/listen on 127.1.0.1 (to not interfere with anything else) and flows (connections) too ("video", "mavlink")
- There should be no IP configuration: clients should have IDs (like "drone3", "gcs4")
- Server should have a REST API to edit routes (so that a client could quickly request to be added to a flow)
