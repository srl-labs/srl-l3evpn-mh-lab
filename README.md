# SRL L3 EVPN Multi-Homing Lab

This lab showcases L3 EVPN Multi-Homing with centralized routers using SR Linux nodes.

## Lab topology
<div align="center"> ![EDA Arena Rack Topology](images/srl-l3-evpn-mh.clab.png)</div>

## Components
Tested with containerlab version `v0.70.2` and SR Linux version `25.7.2`.

## Deploy lab
All configs are prepared, so you just need to deploy the containerlab file.
```bash
clab deploy -t srl-l3-evpn-mh.clab.yml
```

## Network details

The lab consists of 2 spines and 4 leafs forming an EVPN datacenter fabric. The spines act as route reflector but they also participate in the overlay service as centralized routers for the L3 Multi-Homing.

`CE1` is connected to `Leaf1` and `Leaf2` with 2 L3 links (/31). The same applies for `CE2` towards `Leaf3` and `Leaf4`.

<div align="center"> ![EDA Arena Rack Topology](images/srl-l3-evpn-mh-ethernet-segments.png)</div>

The leafs are explicitly configured as primary nodes within the ethernet-segment, whereas the centralized routers are implicitly configured as backup - not advertising themselves as next-hop for the ES.

### Ethernet-Segment config Leaf
```
system {
    network-instance {
        protocols {
            evpn {
                ethernet-segments {
                    bgp-instance 1 {
                        ethernet-segment ES-1 {
                            type virtual
                            admin-state enable
                            esi 04:01:01:01:01:00:00:00:00:00
                            multi-homing-mode single-active
                            next-hop 1.1.1.1 {
                                evi 1 {
                                }
                            }
                            df-election {
                                algorithm {
                                    type manual
                                    manual-alg {
                                        primary-evi-range 1 {
                                            end-evi 1
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
            bgp-vpn {
                bgp-instance 1 {
                }
            }
        }
    }
}
```

### Ethernet-Segment config spine (centralized router)
```
system {
    network-instance {
        protocols {
            evpn {
                ethernet-segments {
                    bgp-instance 1 {
                        ethernet-segment ES-1 {
                            type virtual
                            admin-state enable
                            esi 04:01:01:01:01:00:00:00:00:00
                            multi-homing-mode single-active
                            next-hop 1.1.1.1 {
                                evi 1 {
                                }
                            }
                            df-election {
                                algorithm {
                                    type manual
                                }
                            }
                        }
                        ethernet-segment ES-2 {
                            type virtual
                            admin-state enable
                            esi 04:02:02:02:02:00:00:00:00:00
                            multi-homing-mode single-active
                            next-hop 2.2.2.2 {
                                evi 1 {
                                }
                            }
                            df-election {
                                algorithm {
                                    type manual
                                }
                            }
                        }
                    }
                }
            }
            bgp-vpn {
                bgp-instance 1 {
                }
            }
        }
    }
}
```
