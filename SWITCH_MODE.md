## Switch Mode

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

```shell
cd ../xxx
```

---

```shell
docker-compose -f postgres/docker-compose-ledger.yml up -d
```

or 

```shell
docker-compose -f postgres/docker-compose-auditor.yml up -d
```
