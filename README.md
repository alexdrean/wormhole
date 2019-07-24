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

## Connection procedure
1. Establish relay connection.
    - If this fails, then all else will. We return an error.
    - If this method is being used, we try to establish a direct P2P connection in a repeated manner (with a cooldown of 10 seconds).
2. Try to connect directly A <> B.
    - This will work if at least one host has public ports.
    - We don't try connecting A <> B through their internal ports right now. Although this is a good strategy to improve reliability and speed, it interferes with making sure the solution works on the public internet. (We very likely won't have this bandwidth/ping/simplicity at scale) 
3. Check what type of NAT both ends are behind
    - We use different public IPs on the rendezvous server for that.
4. Apply corresponding strategy
    - If this fails, we try twice again. A network error could have made the process fail.
5. Monitor the connection
    - If we don't see any keepalive for more than 1 second, we fallback to the relay connection and try a P2P connection again (our IP could have changed).

### Strategies
#### Relay
Create regular connections to a main server.

If clients A and B want to communicate, we assign port 2xxx to A and port 3xxx to B on a rendezvous server.

We pipe ports together and achieve communication.

This design works with any NAT that doesn't aggressively block UDP traffic, i.e. virtually any non-enterprise NAT.

This is the strategy that is used initially for connections (to provide fast connection initiation). It is therefore the fallback strategy.

#### Direct P2P connection
##### When one end has publicly accessible ports
When one end has open ports, we just ask the other end to connect directly.

##### When both ends are behind a (full) cone NAT
We use the rendezvous server to determine the public IPs and ports. Both clients then send hole-punching packets to each other and the connection is initiated.

##### When one end (A) is behind a cone NAT and the other (B) is behind a symmetric end
We use the rendezvous server to determine A's IP/Port and B's IP.

B sends 256 hole punching packets to A's IP/Port from 256 different local ports.
This will effectively create 256 holes in A's NAT.

B then sends packets to A's IP on random ports from the same (hole-punched on A's side) port.
When A receives one packet on either of the 256 listeners it has, it replies from the port it has received the packet on.

The connection has now been established.

##### When both ends are between symmetric NATs
There is no strategy for this right now. We just default to the relay method.
