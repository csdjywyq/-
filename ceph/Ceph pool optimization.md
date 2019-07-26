# Ceph pool optimization池优化

## How to rebalance an empty pool如何重新平衡空池

Rebalancing a pool may move a lot of PGs around and slow down the cluster if they contain a lot of objects. This is not a concern when the pool was just created and is empty.

重新平衡池可能会移动很多PG，如果它们包含大量对象，则会减慢群集的速度。 刚刚创建池并且为空时，这不是问题。

Get the report from the ceph cluster: it contains the crushmap, the osdmap and the information about all pools:

从ceph集群获取报告：它包含crushmap，osdmap和有关所有池的信息：

```
$ ceph report > report.json
```

Run the optimization for a given pool with:

使用以下命令运行给定池的优化：

```
$ crush optimize --crushmap report.json --out-path optimized.crush --pool 3
```

Upload the crushmap to the ceph cluster with:

使用以下命令将crushmap上传到ceph集群：

```
$ ceph osd setcrushmap -i optimized.crush
```

## How to rebalance a pool step by step如何逐步重新平衡池

When a pool contains objects, rebalancing can be done in small increments (as specified by –step) to limit the number of PGs being moved.

当池包含对象时，可以以小增量（由-step指定）完成重新平衡，以限制要移动的PG的数量。

Get the report from the ceph cluster: it contains the crushmap, the osdmap and the information about all pools:

```
$ ceph report > report.json
```

Run the optimization for a given pool and move as few PGs as possible with:

从ceph集群获取报告：它包含crushmap，osdmap和有关所有池的信息：

```
$ crush optimize \
        --step 1 \
        --crushmap report.json --out-path optimized.crush \
        --pool 3
```

Upload the crushmap to the ceph cluster with:

使用以下命令将crushmap上传到ceph集群：

```
$ ceph osd setcrushmap -i optimized.crush
```

Repeat until it cannot be optimized further.

重复，直到无法进一步优化。

## Compare the crushmaps比较crushmaps

To verify which OSD are going to get or give PGs, the crush compare command can be used:

要验证哪个OSD将要获取或给PG，可以使用crush compare命令：

```
$ crush compare --origin report.json \
                --destination optimized.crush \
                --pool 3
```

## Adding and removing OSDs in Luminous (or above) clusters在Luminous（或以上）群集中添加和删除OSD

When an OSD is added and the osd_crush_update_on_start option is true (which is the default), its value in the weight set will be the same as its target weight. For instance if an OSD with weight 4 is added, its weight set will also be 4. It is unlikely to be the desired value in the weight set and it is recommended to run crush optimize to rebalance the cluster.

添加OSD并且osd_crush_update_on_start选项为true（默认值）时，其权重集中的值将与其目标权重相同。 例如，如果添加了权重为4的OSD，其权重集也将为4.它不太可能是权重集中的期望值，建议运行压缩优化以重新平衡群集。

Alternatively the crushmap can be manually edited and uploaded.

或者，可以手动编辑和上载crushmap。

## Requirements要求

- All buckets in the pool must use the straw2 algorithm.
- 池中的所有桶必须使用straw2算法。
- The OSDs must all have the default primary affinity.
- OSD必须都具有默认的主要关联。
- The cluster must be HEALTH_OK
- 群集必须是HEALTH_OK