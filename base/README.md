# Single node setup with monolithic gNB

## Setup the node

Download and run the 5G core following the instructions at https://gitlab.eurecom.fr/oai/openairinterface5g/-/blob/develop/doc/NR_SA_Tutorial_OAI_CN5G.md.

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
./build_oai --ninja --gNB --nrUE --build-lib "nrscope" -C
```

Download config files:
```sh
git clone
RB_CONFIG=$(pwd)/oai-radiobox-setup/base
```

## Run

```sh
cd ~/openairinterface5g/cmake_targets/ran_build/build/

# Run the gNB
sudo ./nr-softmodem -O $RB_CONFIG/gnb.sa.band78.106prb.rfsim.conf --rfsim --sa

# Run the UE
sudo ./nr-uesoftmodem -r 106 --numerology 1 --band 78 -C 3619200000 --rfsim --sa --uicc0.imsi 001010000000001 --rfsimulator.serveraddr 127.0.0.1
```

Run a ping:
```sh
# UE to DN
ping 192.168.70.135 -I oaitun_ue1

# DN to UE
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER ping 12.1.1.2
```

Exchange data with iperf:
```sh
DN_CONTAINER=$(docker ps | grep trf-gen-cn5g | awk '{ print $1 }')
docker exec -it $DN_CONTAINER iperf3 -s
iperf3 -c 192.168.70.135 -B 12.1.1.2
```