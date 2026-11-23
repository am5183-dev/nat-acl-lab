# Branch Network with NAT/PAT and Access Control

A small branch network built and tested in **Cisco Packet Tracer 9.0.1** for NSSA-241 Introduction to Routing and Switching at RIT Kosovo (Fall 2026).

Staff and guest users sit on separate VLANs behind one router. Both share a single public IP address through PAT, and named extended ACLs control what each group can reach.

## Topology

```
Staff-PC (VLAN 10) ─┐
                    ├─ SW1 ══ 802.1Q trunk ══ R1 ── ISP ── Public-Server
Guest-PC (VLAN 20) ─┘                    (PAT, ACLs)      (DNS + HTTP)
```

| Network | Subnet | Gateway |
|---|---|---|
| Staff (VLAN 10) | 192.168.10.0/24 | 192.168.10.1 (R1 G0/0.10) |
| Guest (VLAN 20) | 192.168.20.0/24 | 192.168.20.1 (R1 G0/0.20) |
| R1 to ISP | 203.0.113.0/30 | R1 = .2 (PAT address), ISP = .1 |
| Public | 198.51.100.0/24 | Server 198.51.100.10 |

## What it does

- **Inter-VLAN routing:** router-on-a-stick subinterfaces on R1 over one 802.1Q trunk.
- **PAT (NAT overload):** both private networks are translated to 203.0.113.2. The translation table shows staff and guest flows sharing one address, kept apart by port number.
- **WEB_ONLY ACL (staff):** allows only DNS, HTTP and ICMP. FTP to the server is blocked.
- **GUEST_POLICY ACL (guest):** blocks the guest VLAN from reaching staff but allows DNS, HTTP and ICMP to the Internet.
- Both ACLs are applied **inbound on the subinterfaces**, so they match the real private source address before NAT changes it.

## Troubleshooting

Three faults were introduced on purpose, diagnosed with two pieces of evidence each, fixed with the smallest change, and regression-tested:

| Fault | Symptom | Fix |
|---|---|---|
| `ip nat inside` missing on G0/0.20 | Guest can reach its gateway but not the Internet | Re-add `ip nat inside` on G0/0.20 |
| `deny udp any any eq domain` inserted above the DNS permit | Guest DNS lookups time out; ping by IP still works | Remove the misordered entry (`no 15`) |
| GUEST_POLICY moved outbound on G0/1 | Guest can reach staff, and both VLANs lose the Internet | Move the ACL back inbound on G0/0.20 |

## Repository contents

- `packet-tracer/`: final Packet Tracer file (open with Packet Tracer 9.0 or later)
- `configs/`: device configurations for R1, SW1 and ISP

## Key verification commands

```
show vlan brief
show interfaces trunk
show ip interface brief
show ip nat translations
show ip nat statistics
show access-lists
```
