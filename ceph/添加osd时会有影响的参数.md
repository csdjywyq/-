#### OSDMAP_FLAGS

One or more cluster flags of interest has been set. These flags include:

- *full* - the cluster is flagged as full and cannot service writes
- *pauserd*, *pausewr* - paused reads or writes
- *noup* - OSDs are not allowed to start
- *nodown* - OSD failure reports are being ignored, such that the monitors will not mark OSDs down
- *noin* - OSDs that were previously marked out will not be marked back in when they start
- *noout* - down OSDs will not automatically be marked out after the configured interval
- *nobackfill*, *norecover*, *norebalance* - recovery or data rebalancing is suspended
- *noscrub*, *nodeep_scrub* - scrubbing is disabled
- *notieragent* - cache tiering activity is suspended

With the exception of *full*, these flags can be set or cleared with:

```shell
ceph osd set <flag>
ceph osd unset <flag>
```



## BACKFILLING

When you add or remove Ceph OSD Daemons to a cluster, the CRUSH algorithm will want to rebalance the cluster by moving placement groups to or from Ceph OSD Daemons to restore the balance. The process of migrating placement groups and the objects they contain can reduce the cluster’s operational performance considerably. To maintain operational performance, Ceph performs this migration with ‘backfilling’, which allows Ceph to set backfill operations to a lower priority than requests to read or write data.

```
osd max backfills
```

| Description: | The maximum number of backfills allowed to or from a single OSD. |
| :----------- | ------------------------------------------------------------ |
| Type:        | 64-bit Unsigned Integer                                      |
| Default:     | `1`                                                          |

```
osd backfill scan min
```

| Description: | The minimum number of objects per backfill scan. |
| :----------- | ------------------------------------------------ |
| Type:        | 32-bit Integer                                   |
| Default:     | `64`                                             |

```
osd backfill scan max
```

| Description: | The maximum number of objects per backfill scan. |
| :----------- | ------------------------------------------------ |
| Type:        | 32-bit Integer                                   |
| Default:     | `512`                                            |

```
osd backfill retry interval
```

| Description: | The number of seconds to wait before retrying backfill requests. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Double                                                       |
| Default:     | `10.0`                                                       |

## RECOVERY

When the cluster starts or when a Ceph OSD Daemon crashes and restarts, the OSD begins peering with other Ceph OSD Daemons before writes can occur. See [Monitoring OSDs and PGs](http://docs.ceph.com/docs/luminous/rados/operations/monitoring-osd-pg#peering) for details.

If a Ceph OSD Daemon crashes and comes back online, usually it will be out of sync with other Ceph OSD Daemons containing more recent versions of objects in the placement groups. When this happens, the Ceph OSD Daemon goes into recovery mode and seeks to get the latest copy of the data and bring its map back up to date. Depending upon how long the Ceph OSD Daemon was down, the OSD’s objects and placement groups may be significantly out of date. Also, if a failure domain went down (e.g., a rack), more than one Ceph OSD Daemon may come back online at the same time. This can make the recovery process time consuming and resource intensive.

To maintain operational performance, Ceph performs recovery with limitations on the number recovery requests, threads and object chunk sizes which allows Ceph perform well in a degraded state.

```
osd recovery delay start
```

| Description: | After peering completes, Ceph will delay for the specified number of seconds before starting to recover objects. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Float                                                        |
| Default:     | `0`                                                          |

```
osd recovery max active
```

| Description: | The number of active recovery requests per OSD at one time. More requests will accelerate recovery, but the requests places an increased load on the cluster. |
| :----------- | ------------------------------------------------------------ |
| Type:        | 32-bit Integer                                               |
| Default:     | `3`                                                          |

```
osd recovery max chunk
```

| Description: | The maximum size of a recovered chunk of data to push. |
| :----------- | ------------------------------------------------------ |
| Type:        | 64-bit Unsigned Integer                                |
| Default:     | `8 << 20`                                              |

```
osd recovery max single start
```

| Description: | The maximum number of recovery operations per OSD that will be newly started when an OSD is recovering. |
| :----------- | ------------------------------------------------------------ |
| Type:        | 64-bit Unsigned Integer                                      |
| Default:     | `1`                                                          |

```
osd recovery thread timeout
```

| Description: | The maximum time in seconds before timing out a recovery thread. |
| :----------- | ------------------------------------------------------------ |
| Type:        | 32-bit Integer                                               |
| Default:     | `30`                                                         |

```
osd recover clone overlap
```

| Description: | Preserves clone overlap during recovery. Should always be set to `true`. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Boolean                                                      |
| Default:     | `true`                                                       |

```
osd recovery sleep
```

| Description: | Time in seconds to sleep before next recovery or backfill op. Increasing this value will slow down recovery operation while client operations will be less impacted. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Float                                                        |
| Default:     | `0`                                                          |

```
osd recovery sleep hdd
```

| Description: | Time in seconds to sleep before next recovery or backfill op for HDDs. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Float                                                        |
| Default:     | `0.1`                                                        |

```
osd recovery sleep ssd
```

| Description: | Time in seconds to sleep before next recovery or backfill op for SSDs. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Float                                                        |
| Default:     | `0`                                                          |

```
osd recovery sleep hybrid
```

| Description: | Time in seconds to sleep before next recovery or backfill op when osd data is on HDD and osd journal is on SSD. |
| :----------- | ------------------------------------------------------------ |
| Type:        | Float                                                        |
| Default:     | `0.025`                                                      |



# Backfill, Recovery, and Rebalancing

When any component within a cluster fails, be it a single OSD device, a host's worth of OSDs, or a larger bucket like a rack, Ceph waits for a short grace period before it marks the failed OSDs *out*. This state is then updated in the CRUSH map. As soon an OSD is marked out, Ceph initiates recovery operations. This grace period before marking OSDs out is set by the optional ceph.conf tunable mon_osd_down_out_interval, which defaults to 300 seconds (5 minutes). During recovery Ceph moves or copies all data that was hosted on the OSD devices that failed.

Since CRUSH replicates data to multiple OSDs, replicated copies survive and are read during recovery. As CRUSH develops the requisite new mapping of PGs to OSDs to populate the CRUSH map it minimizes the amount of data that must move to other, still-functioning hosts and OSD devices to complete recovery. This helps degraded objects (those that lost their copies due to failures), and thus the cluster as a whole becomes healthy once again as the replication demanded by the configured CRUSH rule policy is restored.

Once a new disk is added to a Ceph cluster, CRUSH will start renewing the cluster state, identify how the new PG placement should take place for all existing PGs and move the new disk to Up Set (and subsequently Acting Set) of the new disk. Once a disk is added newly to the Up Set, Ceph will start rebalancing data on the new disk for the PGs it's supposed to hold. The data is moved from older hosts or disks into our new disk. Rebalancing operation is necessary to keep all the disks equally utilized and thus the placement and workload uniform. For instance, if a Ceph cluster contains 2000 OSDs and a new server is added with 20 OSDs in it, CRUSH will attempt to change the placement assignment of only 1% of thePGs. Multiple OSDs might be employed for pulling relevant data out for the PGs they own, but they will work in parallel to quickly move the data over.

For Ceph clusters running with a high utilization is paramount to take care before adding new disks. The amount of data movement a new disk or a new server can cause is directly proportional to the percentage of the ratio of its CRUSH weight to the original CRUSH weight of the rest of the cluster. Typically for larger clusters, the preferred methodology to add new disks or servers is to add them with CRUSH weight of 0 to the cluster CRUSH map and gradually increase their weights. This lets you control the rate of data movement and speed it up or slow it down depending on your use-case and workloads. Another technique you can use is, when you are adding new servers you can add them to a separate, dummy, CRUSH root when you provision the OSDs on them, so they don't incur any I/O even though Ceph processes are running on the host. And then move the servers one by one into the respective buckets within the CRUSH root that your pools' CRUSH rules are using.