# Two nodes setup with RFsimulator

## Setup the Core/UE node

Download and run the 5G core following the instructions at https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo brctl addif demo-oai enp94s0f0
```

Build the UE:
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
./build_oai --ninja --nrUE --build-lib "nrscope" -C
```

## Setup the gNB node

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo ip addr add 192.168.70.140/24 dev enp94s0f0
```

Build the gNB:
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
./build_oai --ninja --gNB --build-lib "nrscope" -C
```

## Run

On the gNB node:
```sh
# Run the CU-CP
sudo ./nr-softmodem -O ../../../radiobox/config/gnb-cucp.sa.f1.conf --sa

# Run the CU-UP
sudo ./nr-cuup -O ../../../radiobox/config/gnb-cuup.sa.f1.conf --sa

# Run the DU
sudo ./nr-softmodem -O ../../../radiobox/config/gnb-du.sa.band78.106prb.rfsim.conf --rfsim --sa
```

Run the UE on the Core/UE node:
```sh
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --uicc0.imsi 001010000000001 --rfsimulator.serveraddr 192.168.70.140
```

Run a ping, on the Core/UE node:
```sh
# UE to DN
ping 192.168.70.135 -I oaitun_ue1

# DN to UE
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER ping 12.1.1.2
```

Exchange data with iperf, on the Core/UE node:
```sh
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER iperf3 -s
iperf3 -c 192.168.70.135 -B 12.1.1.2
```