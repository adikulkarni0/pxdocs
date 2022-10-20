---
title: Portworx Metrics for monitoring
keywords: portworx, learning, lightboard
description: See reference tables for all Portworx metrics available for monitoring applications.
weight: 600
noicon: true
---


## backup_stats stats
| Name | Description |
| :--- | :--- |
| px_backup_stats_backup_status | Status for this backup (0=InProgress,1=Done,2=Failed) |
| px_backup_stats_backup_size | Size in bytes for this backup |
| px_backup_stats_backup_duration_seconds | Duration in seconds for this backup |
| px_backup_stats_backup_uploaded_bytes_mbs | Delta bytes uploaded in MB/s from last interval for this backup |
| px_backup_stats_download_size_bytes | Size of downloaded bytes during backup/restore from cloud |
| px_backup_stats_upload_size_bytes | Size of uploaded bytes during backup/restore from cloud |
| px_backup_stats_get_apis_invoked | Number of times GET API was invoked |
| px_backup_stats_put_apis_invoked | Number of times PUT API was invoked |

## cluster stats
| Name | Description |
| :--- | :--- |
| px_cluster_cpu_percent | Percentage of CPU Used |
| px_cluster_memory_utilized_percent | Percentage of memory utilization |
| px_cluster_disk_total_bytes | Total storage space in bytes for this node |
| px_cluster_disk_available_bytes | Available storage space in bytes for this node |
| px_cluster_disk_utilized_bytes | Utilized storage space in bytes for this node |
| px_cluster_pendingio | Number of read and write operations currently in progress for this node |
| px_cluster_trashcan_volumes | Number of volumes currently in the trashcan |

## cluster_status stats
| Name | Description |
| :--- | :--- |
| px_cluster_status_cluster_size | Node count for your portworx cluster. **Deprecated**. |
| px_cluster_status_size | Node count for your portworx cluster |
| px_cluster_status_cluster_quorum | Indicates if the cluster is in quorum. **Deprecated**. |
| px_cluster_status_quorum | Indicates if the cluster is in quorum |
| px_cluster_status_nodes_online | Number of online nodes in the cluster (includes storage and storageless) |
| px_cluster_status_nodes_offline | Number of offline nodes in the cluster (includes storage and storageless) |
| px_cluster_status_nodes_storage_down | Number of nodes where the storage is full or down |
| px_cluster_status_storage_nodes_online | Number of storage nodes that are online |
| px_cluster_status_storage_nodes_offline | Number of storage nodes that are offline |
| px_cluster_status_storage_nodes_decommissioned | Number of storage nodes that are decommissioned |

## disk_stats stats
| Name | Description |
| :--- | :--- |
| px_disk_stats_used_bytes | Total storage in bytes for this disk |
| px_disk_stats_interval_seconds | interval_seconds |
| px_disk_stats_io_seconds | Time spent doing IO in seconds for this disk |
| px_disk_stats_progress_io | IO's currently in progress for this disk |
| px_disk_stats_disk_read_bytes | Total bytes read for this disk. **Deprecated**. |
| px_disk_stats_read_bytes | Total bytes read for this disk |
| px_disk_stats_write_bytes_seconds | Total written bytes for this disk. **Deprecated**. |
| px_disk_stats_written_bytes | Total written bytes for this disk |
| px_disk_stats_read_seconds | Total time spend reading in seconds for this disk |
| px_disk_stats_write_seconds | Total time spend writing in seconds for this disk |
| px_disk_stats_read_latency_seconds | Average time spent per read operation in seconds for this disk |
| px_disk_stats_write_latency | Average time spent per write operation in seconds for this disk. **Deprecated**. |
| px_disk_stats_write_latency_seconds | Average time spent per write operation in seconds |
| px_disk_stats_disk_num_reads | Total number of read operations completed successfully for this disk. **Deprecated**. |
| px_disk_stats_disk_num_writes | Total number of write operations completed successfully for this disk. **Deprecated**. |
| px_disk_stats_num_reads | Total number of read operations completed successfully for this disk |
| px_disk_stats_num_writes | Total number of write operations completed successfully for this disk |
| px_disk_stats_num_reads_total | Total number of read operations completed successfully for this disk |
| px_disk_stats_num_writes_total | Total number of write operations completed successfully for this disk |
| px_disk_stats_written_bytes_total | Total bytes written for this disk |
| px_disk_stats_read_bytes_total | Total bytes read for this disk |
| px_disk_stats_read_seconds_total | Total time spend reading in seconds for this disk |
| px_disk_stats_write_seconds_total | Total time spend writing in seconds for this disk |

## kvdb stats
| Name | Description |
| :--- | :--- |
| px_kvdb_get_requests_total | Total number of get requests for a key |
| px_kvdb_snapshot_requests_total | Total number of kvdb snapshot requests for a list of prefixes |
| px_kvdb_put_requests_total | Total number of put requests for a key |
| px_kvdb_create_requests_total | Total number of create requests for a key |
| px_kvdb_update_requests_total | Total number of update requests for a key |
| px_kvdb_enumerate_requests_total | Total number of enumerate requests for a key |
| px_kvdb_delete_requests_total | Total number of delete requests for a key |
| px_kvdb_keys_requests_total | Total number of list key requests for a prefix |
| px_kvdb_cas_requests_total | Total number of compare and sets for a key |
| px_kvdb_cad_requests_total | Total number of compare and deletes for a key |
| px_kvdb_lock_requests_total | Total number of lock requests for a key |
| px_kvdb_unlock_requests_total | Total number of unlock requests for a key |
| px_kvdb_watchkey_requests_total | Total number of watch requests for a key from a node |
| px_kvdb_watchtree_requests_total | Total number of watch requests for a prefix from a node |
| px_kvdb_adduser_requests_total | Total number of add user requests |
| px_kvdb_removeuser_requests_total | Total number of remove user requests |
| px_kvdb_grantuser_access_requests_total | Total number of grant access requests for a user |
| px_kvdb_revokeuser_access_requests_total | Total number of revoke access requests for a user |
| px_kvdb_addmember_requests_total | Total number of add member requests from a node |
| px_kvdb_removemember_requests_total | Total number of remove member requests from a node |
| px_kvdb_updatemember_requests_total | Total number of update member requests from a node |
| px_kvdb_listmembers_requests_total | Total number of list member requests from a node |
| px_kvdb_setendpoints_requests_total | Total number of set endpoint requests from a node |
| px_kvdb_getendpoints_requests_total | Total number of get endpoint requests from a node |
| px_kvdb_defragment_requests_total | Total number of defragment requests from a node |
| px_kvdb_watch_latency_seconds | Time taken in seconds between a kvdb put and a corresponding watch update |

## kvdb_health_state stats
| Name | Description |
| :--- | :--- |
| px_kvdb_health_state_node_view | This node's kvdb state (1 = healthy, 2 = not healthy) |
| px_kvdb_health_state_cluster_view | This node's view of other node's kvdb state (1 = healthy, 2 = not healthy) |

## network_io stats
| Name | Description |
| :--- | :--- |
| px_network_io_bytessent | Number of bytes sent during this interval |
| px_network_io_received_bytes | Number of bytes received during this interval |
| px_network_io_sent_bytes_total | Total number of bytes sent |
| px_network_io_received_bytes_total | Total number of bytes received |
| px_network_io_reservation_waits | Total number of SM waits for reservation |
| px_network_io_reservation_wake_ups | Total number of SM wake ups after getting the reservation |

## node_stats stats
| Name | Description |
| :--- | :--- |
| px_node_stats_used_mem | Used memory in bytes |
| px_node_stats_cpu_usage | Percent of CPU consumption |
| px_node_stats_busy_threads_percent | Percentage of Busy Threads |
| px_node_stats_thread_pool_high_priority_queue_size | Thread Pool High Priority Queue Size |
| px_node_stats_writeback_list_num_dirty_pools | Writeback List Number of Dirty Pools |
| px_node_stats_writeback_list_num_pending_log_bufs | Writeback List Number of Pending Log Buffers |
| px_node_stats_writeback_list_low_cos_queue_size | Writeback List Low Cos Queue Size |
| px_node_stats_writeback_list_high_cos_queue_size | Writeback List High Cos Queue Size |
| px_node_stats_sm_run_queue_size | Statemachine Run Queue Size |
| px_node_stats_kaio_pending_queue_size | Kaio Pending Queue Size |
| px_node_stats_request_scheduler_queue_size | Request Scheduler Queue Size |
| px_node_stats_procfs_mem_available_bytes | Available memory in bytes |
| px_node_stats_procfs_mem_dirty_bytes | The total amount of memory waiting to be written back to the disk |
| px_node_stats_procfs_mem_writeback_bytes | The total amount of memory actively being written back to the disk |
| px_node_stats_procfs_mem_total_bytes | Total amount of usable RAM which is physical RAM minus a number of reserved bits and the kernel binary code |
| px_node_stats_procfs_mem_free_bytes | The amount of physical RAM, in bytes, left unused by the system. |
| px_node_stats_procfs_mem_buffers_bytes | The amount, in bytes, of temporary storage for raw disk blocks |
| px_node_stats_procfs_mem_cached_bytes | The amount of physical RAM, in bytes, used as cache memory |
| px_node_stats_procfs_mem_active_anon_bytes | The amount of anonymous and tmpfs/shmem memory, in bytes, that is in active use, or was in active use since the last time the system moved something to swap |
| px_node_stats_procfs_mem_inactive_anon_bytes | The amount of anonymous and tmpfs/shmem memory, in bytes, that is a candidate for eviction |
| px_node_stats_procfs_mem_active_file_bytes | The amount of file cache memory, in bytes, that is in active use, or was in active use since the last time the system reclaimed memory |
| px_node_stats_procfs_mem_inactive_file_bytes | The amount of file cache memory, in bytes, that is newly loaded from the disk, or is a candidate for reclaiming |
| px_node_stats_procfs_mem_unevictable_bytes | The amount of memory, in bytes, discovered by the pageout code, that is not evictable because it is locked into memory by user programs |
| px_node_stats_procfs_mem_mlocked_bytes | The total amount of memory, in bytes, that is not evictable because it is locked into memory by user programs |
| px_node_stats_procfs_mem_anon_pages_bytes | The total amount of memory, in bytes, used by pages that are not backed by files and are mapped into userspace page tables |
| px_node_stats_procfs_mem_mapped_bytes | The memory, in bytes, used for files that have been mmaped, such as libraries |
| px_node_stats_procfs_mem_sh_mem_bytes | The total amount of memory, in bytes, used by shared memory (shmem) and tmpfs |
| px_node_stats_num_skinnysnaps | The number of SkinnySnaps |
| px_node_stats_skinnysnaps_num_repls_skips | The number of target replicas skipped for a snapshot due to SkinnySnap |
| px_node_stats_skinnysnaps_num_repls_snapshots | The number of target replicas that underwent a snapshot operation due to SkinnySnap |
| px_node_stats_relaxed_reclaim_pending | The number of volumes in RelaxedReclaim pending queue |
| px_node_stats_relaxed_reclaim_skipped | The number of volume deletes that were skipped from being delayed even when RelaxedReclaim was enabled. |
| px_node_stats_relaxed_reclaim_deleted | The number of volume deletes done through RelaxedReclaim |
| px_node_stats_num_volumes | The number of volumes hosted on this node |

## node_status stats
| Name | Description |
| :--- | :--- |
| px_node_status_node_status | Status of this node (https://libopenstorage.github.io/w/master.generated-api.html#status) |
| px_node_status_license_expiry | Number of days until License (or License lease) expires (<0 means Expired) |

## none_stats stats
| Name | Description |
| :--- | :--- |
| px_none_stats_free_mem | Available memory in bytes |
| px_none_stats_total_mem | Total memory in bytes |

## pool_stats stats
| Name | Description |
| :--- | :--- |
| px_pool_stats_pool_written_bytes | Bytes written since last interval for this pool. **Deprecated**. |
| px_pool_stats_pool_write_latency_seconds | Average time spent per write operation for this pool. **Deprecated**. |
| px_pool_stats_pool_writethroughput | Average number of bytes written per second for this pool. **Deprecated**. |
| px_pool_stats_pool_flushed_bytes | Number of flushed bytes since last interval for this pool. **Deprecated**. |
| px_pool_stats_pool_num_flushes | Number of flush(sync) operations since last interval for this pool. **Deprecated**. |
| px_pool_stats_pool_flushms | Latency for flush for this pool. **Deprecated**. |
| px_pool_stats_pool_provisioned_bytes | Provisioned storage space in bytes for this pool. **Deprecated**. |
| px_pool_stats_pool_status | Status of this Pool (0=Offline,1=Online). **Deprecated**. |
| px_pool_stats_written_bytes | Bytes written since last interval for this pool |
| px_pool_stats_num_writes | Number of write operations in the last interval for this pool |
| px_pool_stats_write_ms | Total time in millisecond spent in writing in the last interval for this pool |
| px_pool_stats_write_latency_seconds | Average time spent per write operation for this pool |
| px_pool_stats_write_iops | Average number of completed write operations per second for this pool |
| px_pool_stats_writethroughput | Average number of bytes written per second for this pool |
| px_pool_stats_flushed_bytes | Number of flushed bytes since last interval for this pool |
| px_pool_stats_num_flushes | Number of flush(sync) operations since last interval for this pool |
| px_pool_stats_flushms | Latency for flush for this pool |
| px_pool_stats_provisioned_bytes | Provisioned storage space in bytes for this pool |
| px_pool_stats_status | Status of this Pool (0=Offline,1=Online,2=Full,3=NotFound,4=Maintenance) |
| px_pool_stats_available_bytes | Available storage space in bytes for this pool |
| px_pool_stats_used_bytes | Used storage space in bytes for this pool |
| px_pool_stats_total_bytes | Total storage space in bytes for this pool |
| px_pool_stats_written_bytes_total | Total bytes written for this pool |
| px_pool_stats_flushed_bytes_total | Total number of flushed bytes |
| px_pool_stats_num_flushes_total | Total number of flush(sync) operations |
| px_pool_stats_flushms_total | Total time spent in flush |

## proc_stats stats
| Name | Description |
| :--- | :--- |
| px_proc_stats_virt | Virtual memory in bytes |
| px_proc_stats_res | Resident set size memory in bytes |
| px_proc_stats_cputime | Amount of time that this process has been scheduled in user and kernel mode measured in clock ticks |

## px_cache stats
| Name | Description |
| :--- | :--- |
| px_px_cache_status | Cache enabled (0=No,1=Yes) |
| px_px_cache_total_blocks | Number of total blocks in the cache |
| px_px_cache_used_blocks | Number of used blocks in the cache |
| px_px_cache_dirty_blocks | Number of dirty blocks in the cache |
| px_px_cache_read_hits | Number of read hits for the cache |
| px_px_cache_read_miss | Number of read misses for the cache |
| px_px_cache_write_hits | Number of write hits for the cache |
| px_px_cache_write_miss | Number of write misses for the cache |
| px_px_cache_block_size | Block size for the cache |
| px_px_cache_mode | Mode of the cache |
| px_px_cache_migrate_promote | Number of blocks promoted to the cache |
| px_px_cache_migrate_demote | Number of block demoted from the cache |
| px_px_cache_io_mbps | Approx cache bandwidth from cache internal actions |

## rebalance stats
| Name | Description |
| :--- | :--- |
| px_rebalance_rebalance_job_state | Rebalance job state (0 = pending, 1 = running, 2 = done, 3 = paused, 4 = cancelled) |
| px_rebalance_provision_space_rebalanced_bytes_total | Total provisioned space rebalanced (only counts add (since remove has equal value as add)) |
| px_rebalance_used_space_rebalanced_bytes_total | Total used space rebalanced (only counts add (since remove has equal value as add)) |
| px_rebalance_volumes_rebalanced_total | Total volumes affected by rebalance operation |
| px_rebalance_overloaded_pools_total | Number of overloaded pools |

## volume stats
| Name | Description |
| :--- | :--- |
| px_volume_usage_bytes | Used storage space in bytes for this volume |
| px_volume_capacity_bytes | Configured size in bytes for this volume |
| px_volume_halevel | Configured HA level for this volume |
| px_volume_currhalevel | Current HA level for this volume |
| px_volume_iopriority | Configured IO priority for this volume |
| px_volume_elapsed_time_since_detached_seconds | Seconds elapsed since the volume is detached |
| px_volume_elapsed_time_since_attached_seconds | Seconds elapsed since the volume is attached |
| px_volume_attached | Attached state for this volume (0=detached,1=attached) |
| px_volume_status | Status for this volume (https://libopenstorage.github.io/w/master.generated-api.html#volumestatus) |
| px_volume_state | State for this volume (https://libopenstorage.github.io/w/master.generated-api.html#volumestate) |
| px_volume_attached_state | Attached state for this volume (valid only if volume is attached) (https://libopenstorage.github.io/w/master.generated-api.html#attachstate) |
| px_volume_fs_health_status | Filesystem health status for this volume (https://libopenstorage.github.io/w/master.generated-api.html#filesystemhealthstatus) |
| px_volume_replication_status | Replication Status for this volume (0 : up, 1 : not in quorum, 2 : resync state, 3 : degraded, 4 : detached, 5 : restore) |
| px_volume_fs_usage_bytes | Used storage space in bytes as reported by the filesystem for this volume |
| px_volume_fs_capacity_bytes | Total storage space in bytes as reported by the filesystem for this volume |
| px_volume_vol_read_bytes | Number of successfully read bytes during this interval for this volume. **Deprecated**. |
| px_volume_vol_written_bytes | Number of successfully written bytes during this interval for this volume. **Deprecated**. |
| px_volume_vol_reads | Number of successfully completed read operations during this interval for this volume. **Deprecated**. |
| px_volume_vol_writes | Number of successfully completed write operations during this interval for this volume. **Deprecated**. |
| px_volume_read_bytes | Number of successfully read bytes during this interval for this volume |
| px_volume_written_bytes | Number of successfully written bytes during this interval for this volume |
| px_volume_reads | Number of successfully completed read operations during this interval for this volume |
| px_volume_writes | Number of successfully completed write operations during this interval for this volume |
| px_volume_read_bytes_total | Total number of successfully read bytes for this volume |
| px_volume_written_bytes_total | Total number of successfully written bytes for this volume |
| px_volume_reads_total | Total number of successfully completed read operations for this volume |
| px_volume_writes_total | Total number of successfully completed write operations for this volume |
| px_volume_iops | Number of successful completed I/O operations per second during this interval for this volume |
| px_volume_write_iops | Average number of completed write operations per second for this volume |
| px_volume_read_iops | Average number of completed read operations per second for this volume |
| px_volume_vol_num_sequential_writes | Number of sequential write I/O operations during this interval for this volume |
| px_volume_vol_num_sequential_reads | Number of sequential read I/O operations during this interval for this volume |
| px_volume_vol_num_random_writes | Number of random write I/O operations during this interval for this volume |
| px_volume_vol_num_random_reads | Number of random read I/O operations during this interval for this volume |
| px_volume_depth_io | Number of I/O operations currently in progress for this volume |
| px_volume_readthroughput | Number of bytes read per second during this interval for this volume |
| px_volume_writethroughput | Number of bytes written per second during this interval for this volume |
| px_volume_vol_bytes_reclaimed | Number of bytes reclaimed by fstrim operation |
| px_volume_vol_bytes_reclaimable | Number of bytes reclaimable on the volume as seen by fstrim operation |
| px_volume_vol_read_latency_seconds | Average time spent per successfully completed read operation in seconds during this interval for this volume. **Deprecated**. |
| px_volume_vol_write_latency_seconds | Average time spent per successfully completed write operation in seconds during this interval for this volume. **Deprecated**. |
| px_volume_read_latency_seconds | Average time spent per successfully completed read operation in seconds for this volume |
| px_volume_write_latency_seconds | Average time spent per successfully completed write operation in seconds for this volume |
| px_volume_num_long_reads | Number of long reads for this volume |
| px_volume_num_long_writes | Number of long writes for this volume |
| px_volume_num_long_flushes | Number of long flushes for this volume |
| px_volume_num_db_flushes | Number of DB flushes for this volume |
| px_volume_replica_read_bytes_total | Total number of successfully read bytes for this replica volume |
| px_volume_replica_written_bytes_total | Total number of successfully written bytes for this replica volume |
| px_volume_replica_reads_total | Total number of successfully completed read operations for this replica volume |
| px_volume_replica_writes_total | Total number of successfully completed write operations for this replica volume |
| px_volume_replica_flushes_total | Total number of successfully completed flush operations for this replica volume |
| px_volume_replica_read_ms_total | Total time spent doing read operations in milliseconds for this replica volume |
| px_volume_replica_write_ms_total | Total time doing write operations in milliseconds for this replica volume |
| px_volume_replica_flush_ms_total | Total time doing write operations in milliseconds for this replica volume |
| px_volume_dev_depth_io | Number of I/O operations currently in progress as reported by the kernel pxd device for this volume |
| px_volume_dev_writethroughput | Number of successfully written bytes per second as reported by the kernel pxd device for this volume |
| px_volume_dev_readthroughput | Number of successfully read bytes per second as reported by the kernel pxd device for this volume |
| px_volume_dev_read_latency_seconds | Average time spent per successfully completed read in seconds as reported by the kernel pxd device for this volume |
| px_volume_dev_write_latency_seconds | Average time spent per successfully completed write in seconds as reported by the kernel pxd device for this volume |
| px_volume_dev_read_bytes_total | Total number of successfully read bytes as reported by the kernel pxd device for this volume |
| px_volume_dev_written_bytes_total | Number of successfully written bytes as reported by the kernel pxd device for this volume |
| px_volume_dev_reads_total | Total number of successfully completed read operations as reported by the kernel pxd device for this volume |
| px_volume_dev_writes_total | Total number of successfully completed write operations as reported by the kernel pxd device for this volume |
| px_volume_dev_read_seconds_total | Total time spend reading in seconds for this disk as reported by the kernel pxd device for this volume |
| px_volume_dev_write_seconds_total | Total time spend writing in seconds for this disk as reported by the kernel pxd device for this volume |
| px_volume_unique_blocks | Size(in bytes) of unique blocks for this volume |
| px_volume_timestamp_records | Number of timestamp records accumulated |
| px_volume_timestamp_records_per_node | Number of timestamp records accumulated for a node |
| px_volume_num_discard_ops | Number of discard operations for this volume |
| px_volume_discarded_bytes | Number of discarded bytes for this volume |
