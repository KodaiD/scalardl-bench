# Getting Started

## Setup

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
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

On the auditor side:

```shell
docker-compose -f postgres/docker-compose-auditor.yml up -d
```

On the client side:

```shell
sudo apt install -y openjdk-8-jre openjdk-8-jdk unzip
```

```shell
git clone https://github.com/scalar-labs/scalardl-samples
```

```shell
git clone https://github.com/scalar-labs/scalardl-benchmarks
cd scalardl-benchmarks
```

```shell
cp -r ../scalardl-samples/fixture ./
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
load_concurrency = 16
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
concurrency = 4
run_for_sec = 300
ramp_for_sec = 60
```

2. Run the benchmark:

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --only-pre
```

```shell
./kelpie-1.2.4/bin/kelpie --config ycsb-benchmark-config.toml --except-pre
```
