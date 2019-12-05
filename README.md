# egeneralov.quagga

Role for hypervisors/container hosts for mesh-ing it. Do not use not in private networks, it's not managing your trafic encryption.

## Role Variables

- **password**: `zebra`

## Example result

### playbook

    - hosts: all
      vars:
        password: 54ae945
      roles:
        - egeneralov.disable_ipv6
        - egeneralov.quagga

### inventory

    cat tests/inventory 
    [all]
    test-1 as_number=64511
    test-2 as_number=64512
    test-3 as_number=64513
    test-4 as_number=64514
    test-5 as_number=64515

#### cat /etc/quagga/zebra.conf 
    hostname test-1
    password 54ae945
    log stdout
    
#### cat /etc/quagga/bgpd.conf

    hostname test-1
    password 54ae945
    log syslog
    debug bgp events
    debug bgp updates
    !
    router bgp 64511
     bgp router-id 10.211.55.108
     network 10.0.1.0/24
     neighbor 10.211.55.112 remote-as 64515
     neighbor 10.211.55.112 description test-5
     neighbor 10.211.55.112 ebgp-multihop 255
     neighbor 10.211.55.112 next-hop-self
     neighbor 10.211.55.112 route-map test-1-map out
     neighbor 10.211.55.111 remote-as 64514
     neighbor 10.211.55.111 description test-4
     neighbor 10.211.55.111 ebgp-multihop 255
     neighbor 10.211.55.111 next-hop-self
     neighbor 10.211.55.111 route-map test-1-map out
     neighbor 10.211.55.109 remote-as 64513
     neighbor 10.211.55.109 description test-3
     neighbor 10.211.55.109 ebgp-multihop 255
     neighbor 10.211.55.109 next-hop-self
     neighbor 10.211.55.109 route-map test-1-map out
     neighbor 10.211.55.110 remote-as 64512
     neighbor 10.211.55.110 description test-2
     neighbor 10.211.55.110 ebgp-multihop 255
     neighbor 10.211.55.110 next-hop-self
     neighbor 10.211.55.110 route-map test-1-map out
     address-family ipv4
     exit-address-family
     exit
    !
    access-list all permit any
    !
    ip prefix-list test-1-networks description "Networks for announces here"
    ! br0
    ip prefix-list test-1-networks seq 10 permit 10.0.1.0/24
    !
    route-map set-nexthop permit 10
     match ip address all
     set ip next-hop 10.211.55.108
    !
    route-map test-1-map permit 10
     description "Include network for announces"
     match ip address prefix-list test-1-networks
    !
    line vty
    !
    
#### ip route

    default via 10.211.55.1 dev enp0s5 
    10.0.1.0/24 dev br0 proto kernel scope link src 10.0.1.1 linkdown 
    10.0.2.0/24 via 10.211.55.110 dev enp0s5 proto zebra metric 20 
    10.0.3.0/24 via 10.211.55.109 dev enp0s5 proto zebra metric 20 
    10.0.4.0/24 via 10.211.55.111 dev enp0s5 proto zebra metric 20 
    10.0.5.0/24 via 10.211.55.112 dev enp0s5 proto zebra metric 20 
    10.211.55.0/24 dev enp0s5 proto kernel scope link src 10.211.55.108 
    169.254.0.0/16 dev enp0s5 scope link metric 1000 
    
#### for i in $(seq 5); do ping 10.0.${i}.1 -c 1; done;

    PING 10.0.1.1 (10.0.1.1) 56(84) bytes of data.
    64 bytes from 10.0.1.1: icmp_seq=1 ttl=64 time=0.042 ms
    
    --- 10.0.1.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.042/0.042/0.042/0.000 ms
    PING 10.0.2.1 (10.0.2.1) 56(84) bytes of data.
    64 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=0.234 ms
    
    --- 10.0.2.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.234/0.234/0.234/0.000 ms
    PING 10.0.3.1 (10.0.3.1) 56(84) bytes of data.
    64 bytes from 10.0.3.1: icmp_seq=1 ttl=64 time=0.258 ms
    
    --- 10.0.3.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.258/0.258/0.258/0.000 ms
    PING 10.0.4.1 (10.0.4.1) 56(84) bytes of data.
    64 bytes from 10.0.4.1: icmp_seq=1 ttl=64 time=0.221 ms
    
    --- 10.0.4.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.221/0.221/0.221/0.000 ms
    PING 10.0.5.1 (10.0.5.1) 56(84) bytes of data.
    64 bytes from 10.0.5.1: icmp_seq=1 ttl=64 time=0.329 ms
    
    --- 10.0.5.1 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.329/0.329/0.329/0.000 ms


License
-------

MIT

Author Information
------------------

Eduard Generalov <eduard@generalov.net>
