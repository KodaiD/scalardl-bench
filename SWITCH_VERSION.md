## Switch Version

```shell
docker-compose -f postgres/docker-compose-ledger.yml down -v
```

or

```shell
docker-compose -f postgres/docker-compose-auditor.yml down -v
```

---

```shell
rm -rf /mnt/disks/ssds/postgres-data
```

---

```shell
cd scalardl
git checkout <TARGET_VERSION>
./gradlew :ledger:docker
cd ../
```

or 

```shell
cd scalardl-enterprise
git checkout <TARGET_VERSION>
./gradlew :auditor:docker
cd ../
```

---

```shell
echo "SCALARDL_VERSION=<VERSION>" > postgres/.env
```

---

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

or

```shell
docker-compose -f postgres/docker-compose-auditor.yml up -d
``` 
