# Lab 5 Analysis Questions

## Q1
Comparing the two pcaps with `tcpdump -r <file> -nn -A`, three things visible in the
plaintext capture but hidden in the TLS capture:
1. **MQTT topic** — `sensor/temp` is clearly visible in the packet body.
2. **Payload content** — the temperature values (temp:20, temp:21, etc.) are readable ASCII.
3. **Client metadata** — the MQTT CONNECT packet reveals the client identifier and protocol details.

For an attacker intercepting traffic, plaintext MQTT exposes sensor data, device identity,
and topic structure — enough to clone a device, inject false readings, or map the entire
IoT topology. TLS removes all of this from view; the attacker sees only that a TLS session
exists between two IPs on port 8883.

## Q2
With `require_certificate true`, any client that connects without presenting a valid
certificate signed by our CA is rejected at the TLS handshake — before any MQTT data
is exchanged. Running `mosquitto_sub -h <VM_IP> -p 8883 -t '#'` without cert flags
produces an SSL handshake error. This is a major improvement over `allow_anonymous true`
because unauthenticated clients cannot connect at all. Every connected device must hold
a certificate issued by our CA, creating a cryptographic allowlist of authorized devices.

## Q3
Even though Suricata cannot decrypt TLS payloads on port 8883, it can still observe:
- **Connection metadata** — source/destination IPs, ports, and session duration are
  visible in flow events.
- **Packet volume and timing** — spikes in connection frequency can indicate scanning
  or a compromised device beaconing.
- **TLS handshake fields** — cipher suites and certificate details are unencrypted
  during the handshake and can reveal weak configurations.

In Project 2, a Suricata rule could alert on unusual connection rates to port 8883 from
unknown source IPs, or on TLS handshakes using weak cipher suites — both detectable
without decrypting the payload.
