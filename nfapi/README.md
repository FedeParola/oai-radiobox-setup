# Two nodes setup with nFAPI

## Setup the Core/UE node

Download and run the 5G core following the instructions at https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md.

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo brctl addif demo-oai enp94s0f0
```

Build the the gNB:
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
./build_oai --ninja --nrUE --build-lib "nrscope"
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

## Setup the gNB/UE node

Setup the networking:
```sh
sudo ip addr flush dev enp94s0f0
sudo ip addr add 192.168.70.140/24 dev enp94s0f0
```

Build the the gNB:
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
./build_oai --ninja --gNB --build-lib "nrscope"
```

Download config files:
```sh
git clone
RB_CONFIG=$(pwd)/oai-radiobox-setup
```

## Run

On the gNB node:
```sh
cd ~/openairinterface5g/cmake_targets/ran_build/build/

# Run the CU-CP
sudo ./nr-softmodem -O $RB_CONFIG/nfapi/gnb-cucp.sa.f1.conf --sa

# Run the CU-UP
sudo ./nr-cuup -O $RB_CONFIG/nfapi/gnb-cuup.sa.f1.conf --sa

# Run the DU
sudo ./nr-softmodem -O $RB_CONFIG/nfapi/gnb-du.sa.band78.106prb.nfapi.conf --nfapi VNF --emulate-l1 --sa
```

On the Core/UE node:
```sh
# Run the Multi-UE Proxy
cd ~/oai-lte-5g-multi-ue-proxy
sudo ./build/proxy --nr 1 192.168.70.140 192.168.70.129 192.168.70.129

# Run the UE
cd ~/openairinterface5g/cmake_targets/ran_build/build/
sudo ./nr-uesoftmodem -O $RB_CONFIG/nfapi/nrue.nfapi.conf --nfapi STANDALONE_PNF --node-number 2 --emulate-l1 --sa
```

