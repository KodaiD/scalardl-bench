# Getting Started

## Setup

Mount the local SSD.

```shell
sudo apt update && sudo apt install mdadm --no-install-recommends
```

```shell
find /dev/ | grep google-local-nvme-ssd
```

```shell
sudo mdadm --create /dev/md0 --level=0 --raid-devices=2 \
 /dev/disk/by-id/google-local-nvme-ssd-0 \
 /dev/disk/by-id/google-local-nvme-ssd-1
```

```shell
sudo mdadm --detail --prefer=by-id /dev/md0
```

```shell
sudo mkfs.ext4 -F /dev/md0
```

```shell
sudo mkdir -p /mnt/disks/ssds
```

```shell
sudo mount /dev/md0 /mnt/disks/ssds
```

```shell
sudo chmod a+rw /mnt/disks/ssds
```

Setup Java.

```shell
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 21.0.10-tem
sdk install java 8.0.482-tem
```

Install Docker and Docker Compose.

```shell
sudo apt update && sudo apt install -y docker.io docker-compose
```

```shell
sudo usermod -aG docker $USER
newgrp docker
```

```shell
git clone https://github.com/KodaiD/scalardl-bench.git
cd scalardl-bench
```

```shell
echo "SCALARDL_VERSION=3.12.2" > postgres/.env
```

```shell
echo "YOUR_GHCR_PAT" | docker login ghcr.io -u KodaiD --password-stdin
```

On the ledger side:

```shell
sdk use java 8.0.482-tem
```

```shell
git clone https://github.com/scalar-labs/scalardl.git
cd scalardl
./gradlew :ledger:docker
cd ../
```

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

On the auditor side:

```shell
vim postgres/auditor.properties
```

```shell
sdk use java 8.0.482-tem
```

```shell
git clone git@github.com:scalar-labs/scalardl-enterprise.git
cd scalardl-enterprise
./gradlew :auditor:docker
cd ../
```

```shell
docker-compose -f postgres/docker-compose-auditor.yml up -d
```

On the client side:

```shell
sudo apt update && sudo apt install -y openjdk-8-jre openjdk-8-jdk unzip
```

```shell
git clone https://github.com/KodaiD/scalardl-bench.git
```

```shell
git clone https://github.com/scalar-labs/scalardl-benchmarks
cd scalardl-benchmarks
```

```shell
cp -r ../scalardl-bench/<MODE>/fixture ./
```

```shell
vim fixture/client.properties
```

```shell
vim fixture/ledger.as.client.properties
```

```shell
vim fixture/auditor.as.client.properties
```

```shell
VERSION=3.12.2
curl -OL https://github.com/scalar-labs/scalardl/releases/download/v$VERSION/scalardl-java-client-sdk-$VERSION.zip
unzip scalardl-java-client-sdk-$VERSION.zip
mv scalardl-java-client-sdk-$VERSION client
client/bin/scalardl register-cert --properties ./fixture/ledger.as.client.properties
client/bin/scalardl register-cert --properties ./fixture/auditor.as.client.properties
```

```shell
vim build.gradle
```

```shell
./gradlew shadowJar
```

```shell
vim ycsb-benchmark-config.toml
```

```
[client_config]
config_file = "./fixture/client.properties"
```

```shell
vim fixture/client.properties
```

```
scalar.dl.client.server.host=<LEDGER_ADDRESS>
scalar.dl.client.auditor.host=<AUDITOR_ADDRESS>
```

```shell
wget https://github.com/scalar-labs/kelpie/releases/download/1.2.4/kelpie-1.2.4.zip
unzip kelpie-1.2.4.zip
```

## Run the benchmark

1. Set the benchmark configuration in `ycsb-benchmark-config.toml`.

```
[ycsb_config]
record_count = 1000000
payload_size = 100
ops_per_tx = 2
workload = "C"
load_concurrency = 48
load_batch_size = 1
```

or

```
[ycsb_config]
record_count = 1000000
payload_size = 100
ops_per_tx = 1
workload = "F"
load_concurrency = 16
load_batch_size = 1
```

```
[common]
concurrency = 1
run_for_sec = 60
ramp_for_sec = 60
```

Run the benchmark:

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --only-pre
```

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --except-pre
```
