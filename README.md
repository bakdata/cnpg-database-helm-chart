# cnpg-database-helm-chart

Helm chart for deploying a CNPG database

## Install

```shell
helm repo add cnpg-database https://bakdata.github.io/cnpg-database-helm-chart/
helm repo update
helm install cnpg-database/cnpg-database
```

## High availability

Set `postgres.instances` to at least 2 for a primary and standby. The existing
`postgres.affinity` and `postgres.synchronous` values still work. For fields the chart does
not expose directly, use `postgres.clusterSpec`. For example:

```yaml
postgres:
  instances: 3
  enablePDB: true
  clusterSpec:
    failoverDelay: 60
    affinity:
      enablePodAntiAffinity: true
      podAntiAffinityType: required
      topologyKey: kubernetes.io/hostname
    postgresql:
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

`postgres.clusterSpec` accepts any additional CNPG Cluster `spec` fields and deep-merges them
with the chart's generated spec. Nested settings such as `postgresql.parameters` are retained;
values in `clusterSpec` win when a field is set in both places. `failoverDelay` is in seconds
and delays promotion after CNPG detects an unhealthy primary, allowing brief interruptions
to recover without a failover.
