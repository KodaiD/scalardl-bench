# Getting Started

## Setup

Mount the local SSD.

```shell
sudo apt update && sudo apt install -y mdadm --no-install-recommends zip unzip
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
sudo mkdir -p /mnt/disks/ssds
sudo mount /dev/md0 /mnt/disks/ssds
sudo chmod a+rw /mnt/disks/ssds
```

Setup Java.

```shell
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 8.0.482-tem
sdk use java 8.0.482-tem
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
cd xxx
```

```shell
echo "SCALARDL_VERSION=3.12.2" > postgres/.env
```

```shell
echo "YOUR_GHCR_PAT" | docker login ghcr.io -u KodaiD --password-stdin
```

```shell
sdk use java 8.0.482-tem
```

```shell
git clone https://github.com/scalar-labs/scalardl.git
cd scalardl
```

```shell
git checkout VERSION
```

```shell
./gradlew :ledger:docker -x test
cd ../
```

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

On the client side:

```shell
sudo apt update && sudo apt install -y openjdk-8-jre openjdk-8-jdk unzip
```

```shell
git clone https://github.com/KodaiD/scalardl-bench.git
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
VERSION=3.12.2
```

```shell
curl -OL https://github.com/scalar-labs/scalardl/releases/download/v$VERSION/scalardl-java-client-sdk-$VERSION.zip
unzip scalardl-java-client-sdk-$VERSION.zip
mv scalardl-java-client-sdk-$VERSION client
```

```shell
vim build.gradle
```

```shell
vim src/main/java/com/scalar/dl/benchmarks/ycsb/YcsbLoader.java
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
