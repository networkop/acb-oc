sudo apt install golang-go
Add the following to .bashrc
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

sudo apt install git

go get https://github.com/aristanetworks/goarista/tree/master/cmd/gnmi

mike_mgmt:~$./gnmi -addr 192.168.1.203:6030 -username admin -password admin get '/interfaces/interface[name=Ethernet1]/config/enabled'

/interfaces/interface[name=Ethernet1]/config/enabled:

true

mike_mgmt:~$./gnmi -addr 192.168.1.203:6030 -username admin -password admin update '/interfaces/interface[name=Ethernet1]/config/enabled' 'false’

mike_mgmt:~$./gnmi -addr 192.168.1.203:6030 -username admin -password admin get '/interfaces/interface[name=Ethernet1]/config/enabled'

/interfaces/interface[name=Ethernet1]/config/enabled:

false

mike_mgmt:~$./gnmi -addr 192.168.1.203:6030 -username admin -password admin update 'cli' 'interface ethernet1

no shutdown'    