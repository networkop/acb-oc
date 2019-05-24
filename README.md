## Intro

![](img/1.png)

![](img/2.png)

![](img/3.png)

![](img/4.png)

## Useful links


[GNMI specification](https://github.com/openconfig/reference/blob/master/rpc/gnmi/gnmi-specification.md)

[OpenConfig YANG models](https://github.com/openconfig/public/tree/master/release/models)

[Arista OpenConfig YANG models](https://github.com/aristanetworks/yang)

[GNOI](https://github.com/openconfig/gnoi) - file transfer, ping, traceroute, clear ARP/STP/LLDP/BGP

[GRIBI](https://github.com/openconfig/gribi/blob/master/proto/service/gribi.proto)

## Demo 

### 1. Setup

```
export CEOS_VERSION=ceos:4.21.5F


docker-compose up -d
docker inspect ceos-1 |  jq '.[0].NetworkSettings.Networks.oc_management.IPAddress'
docker inspect ceos-2 |  jq '.[0].NetworkSettings.Networks.oc_management.IPAddress'
```

### 2. Build Arista gNMI client

```

docker exec -it gnmi-client bash

GO111MODULE=on go build -o arista-gnmi github.com/aristanetworks/goarista/cmd/gnmi
```

### 3. Demonstrate gNMI client 

```
./arista-gnmi -addr 172.20.0.2:6030 -password admin -username admin capabilities
```
```
./arista-gnmi -addr 172.20.0.2:6030 -password admin -username admin get /
```
```
./arista-gnmi -addr 172.20.0.2:6030 -password admin -username admin subscribe "/network-instances/network-instance[name=default]/protocols/protocol[identifier=BGP][name=BGP]/bgp/" &
```
```
./arista-gnmi -addr 172.20.0.2:6030 -username admin -password admin update "/interfaces/interface[name=Ethernet1]/config/enabled" "false"
```
```
./arista-gnmi -addr 172.20.0.2:6030 -username admin -password admin update "/interfaces/interface[name=Ethernet1]/config/enabled" "true"
```

## Terraform demo

[Link](https://github.com/networkop/terraform-yang)

1. Tmux-split the screen into three and start Cli sessions two smaller screens, default interface  Ethernet1

2. Open `main.tf` and `provider.tf`

```
cd ~/tf-yang
rm -f terraform.tfstate*        
code main.tf
code provider.tf
```

3. First configure L2 interface


```
resource "gnmi_interface" "SW1_Eth1" {
    provider = "gnmi.s1"
    name = "Ethernet1"
    description = "TF_INT_ETH1"
    switchport = true
    trunk_vlans = [100]
}
resource "gnmi_interface" "SW2_Eth1" {
    provider = "gnmi.s2"
    name = "Ethernet1"
    description = "TF_INT_ETH1"
    switchport = true
    access_vlan = 200
}
```

4. Next configure IPv4 interface

```
resource "gnmi_interface" "SW1_Eth1" {
    provider = "gnmi.s1"
    name = "Ethernet1"
    description = "TF_INT_ETH1"
    switchport = false
    ipv4_address = "12.12.12.1/24"
}
resource "gnmi_interface" "SW2_Eth1" {
    provider = "gnmi.s2"
    name = "Ethernet1"
    description = "TF_INT_ETH1"
    switchport = false
    ipv4_address = "12.12.12.2/24"
}
```

5. Examine the [code](https://github.com/networkop/terraform-yang/blob/master/resource_interface.go)


## Cleanup

```
docker-compose down            
```