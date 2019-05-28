## Intro


### What is OpenConfig

* The OpenConfig Working Group is a group of network operators that defines a set of vendor-neutral data models for switch configuration and status
* The data models are expressed in YANG, a data modelling language
* The OpenConfig working group doesn’t mandate an API (or protocol) for client access
* The OpenConfig data model is accessed and manipulated over a choice of  APIs; NETCONF (over SSH), gNMI (gRPC) and RESTCONF (HTTPS)

### History

* Initially NETCONF, IETF group started 2003,  first spec was RFC4741 Dec 2006
* YANG modelling language, IETF group started 2008,  first spec was RFC6020 in Oct 2010
* Each vendor provided their own unique YANG models
* OpenConfig working group was established in 2014
* The working group published its first models including BGP in 2015 

### Pitch 

*  Better programmatic interface to config and status
* Better than the CLI for automated config management
* Better than SNMP for getting telemetry
    * Streaming, not polling!
* Models developed in the open by operators, with feedback from vendors
    * github.com/openconfig/public
* Allows for cross-vendor tools to develop
    * Where switches from different vendors are managed in the same way.
* Models are not from the usual standard bodies IETF, IEEE not involved
    * allows for rapid development of models

### Customer Use Case

* Do all config with YANG models
    * Use OpenConfig models wherever defined
    * Fill in gaps with proprietary models when necessary, proprietary models may be auto-generated.
    * Common switch management, pushed to switches in a standard fashion. 
* Stream telemetry using YANG model


## Useful links


[GNMI specification](https://github.com/openconfig/reference/blob/master/rpc/gnmi/gnmi-specification.md)

[OpenConfig YANG models](https://github.com/openconfig/public/tree/master/release/models)

[Arista OpenConfig YANG models](https://github.com/aristanetworks/yang)

[GNOI](https://github.com/openconfig/gnoi) - file transfer, ping, traceroute, clear ARP/STP/LLDP/BGP

[GRIBI](https://github.com/openconfig/gribi/blob/master/proto/service/gribi.proto)

## Demo 

### 1. Setup

```bash
export CEOS_VERSION=ceos:4.21.5F

docker-compose up -d
docker inspect ceos-1 |  jq '.[0].NetworkSettings.Networks.oc_management.IPAddress'
docker inspect ceos-2 |  jq '.[0].NetworkSettings.Networks.oc_management.IPAddress'
```

### 2. Exploring YANG models

```bash
docker exec -it gnmi-client bash

GO111MODULE=on go build github.com/openconfig/goyang

git clone --depth 1 https://github.com/aristanetworks/yang arista
git clone --depth 1 https://github.com/YangModels/yang.git yang

./goyang --format tree --path ./arista/ --path ./yang/standard/ietf/ \
./arista/EOS-4.21.3F/openconfig/public/release/models/interfaces/* | more
```



### 3. Build Arista gNMI client

```
docker exec -it gnmi-client bash

GO111MODULE=on go build -o arista-gnmi github.com/aristanetworks/goarista/cmd/gnmi
```



### 4. Demonstrate gNMI client 

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

1. Tmux-split the screen into three and start Cli sessions in two smaller screens, default interface  Ethernet1

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