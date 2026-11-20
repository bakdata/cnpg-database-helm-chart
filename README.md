# cnpg-database-helm-chart

Helm chart for deploying a CNPG database

## Install

```shell
helm repo add cnpg-database https://bakdata.github.io/cnpg-database-helm-chart/
helm repo update
helm install cnpg-database/cnpg-database
```

## High availability

Set `postgres.instances` to at least 2 for a primary and standby. The chart passes
`postgres.affinity` and `postgres.synchronous` to the CNPG Cluster spec. For example:

```yaml
postgres:
  instances: 3
  enablePDB: true
  affinity:
    enablePodAntiAffinity: true
    podAntiAffinityType: required
    topologyKey: kubernetes.io/hostname
  synchronous:
    method: any
    number: 1
    dataDurability: required
```

This waits for one standby to acknowledge each commit. If no standby is available,
`dataDurability: required` pauses writes until one returns. With only two instances,
losing the sole standby therefore stops writes; three instances can tolerate the
loss of one standby. Both settings are optional and leave existing releases unchanged
until explicitly configured.
