# Two nodes stup with nFAPI and gNB/UE/Proxy on the same node

## Setup the Core node

Download and run the 5G core following the instructions at https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md.

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo brctl addif demo-oai enp94s0f0
```

## Setup the gNB/UE node

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo ip addr add 192.168.70.140/24 dev enp94s0f0
```

Build the UE and the gNB:
```sh
git clone https://gitlab.eurecom.fr/oai/openairinterface5g.git
cd openairinterface5g

# Install OAI dependencies
cd ~/openairinterface5g/cmake_targets
./build_oai -I

# nrscope dependencies
sudo apt install -y libforms-dev libforms-bin

# Build OAI gNB
cd ~/openairinterface5g
source oaienv
cd cmake_targets
./build_oai --ninja --gNB --nrUE --build-lib "nrscope"
```

Download config files:
```sh
git clone
RB_CONFIG=$(pwd)/oai-radiobox-setup
```

Download and build the Multi-UE Proxy:
```sh
git clone https://github.com/EpiSci/oai-lte-5g-multi-ue-proxy.git
cd oai-lte-5g-multi-ue-proxy
make
```

## Run

On the gNB/UE node:
```sh
cd ~/openairinterface5g/cmake_targets/ran_build/build/

# Run the CU-CP
sudo ./nr-softmodem -O $RB_CONFIG/nfapi-local/gnb-cucp.sa.f1.conf --sa

# Run the CU-UP
sudo ./nr-cuup -O $RB_CONFIG/nfapi-local/gnb-cuup.sa.f1.conf --sa

# Run the DU
sudo ./nr-softmodem -O $RB_CONFIG/nfapi-local/gnb-du.sa.band78.106prb.nfapi.conf --nfapi VNF --emulate-l1 --sa

# Run the Multi-UE Proxy
cd ~/oai-lte-5g-multi-ue-proxy
sudo ./build/proxy --nr 1

# Run the UE
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-uesoftmodem -O $RB_CONFIG/nfapi-local/nrue.nfapi.conf --nfapi STANDALONE_PNF --node-number 2 --emulate-l1 --sa
```

Run a UE to DN ping, on the gNB/UE node:
```sh
ping 192.168.70.135 -I oaitun_ue1
```

Run a DN to UE ping, on the Core node:
```sh
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER ping 12.1.1.2
```

Exchange data with iperf, on the Core node:
```sh
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER iperf3 -s
```
On the gNB/UE node:
```sh
iperf3 -c 192.168.70.135 -B 12.1.1.2
```