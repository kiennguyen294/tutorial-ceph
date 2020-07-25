# Tunning RBD-mirror
Các tham số cần quan tâm khi triển khai RBD-mirror
- What type of network connection between long-distance 2 Cluster for rbd-mirror( for case long distance between 2 Cluster)
- What option we can control for rbd-mirror (speed, interval time...blabla) I saw some config in there ceph-conf --show-config | grep mirror so some config clear to me some config ***
- So how can spin-off rbd mirror or control bandwidth of rbd-mirror on ceph-public network
- How can default enable journalling for each new volume
- How can multil promote and depromote volume in pool for disaster recovery
## Tự động enable các feature `rbd default features = 125`
```sh 
- Layering: Layering enables you to use cloning
  Config numeric value: 1
  CLI value: layering
- Striping v2: Striping spreads data across multiple objects. Striping
  helps with parallelism for sequential read/write workloads.
  Config numeric value: 2
  CLI value: striping
- Exclusive locking: When enabled, it requires a client to get a lock
  on an object before making a write. Exclusive lock should only be
  enabled when a single client is accessing an image at the same time.
  Config numeric value: 4
  CLI value: exclusive-lock
- Object map: Object map support depends on exclusive lock support.
  Block devices are thin provisioned -- meaning, they only store data
  that actually exists. Object map support helps track which objects
  actually exist (have data stored on a drive). Enabling object map
  support speeds up I/O operations for cloning; importing and exporting
  a sparsely populated image; and deleting.
  Config numeric value: 8
  CLI value: object-map
- Fast-diff: Fast-diff support depends on object map support and
  exclusive lock support. It adds another property to the object map,
  which makes it much faster to generate diffs between snapshots of an
  image, and the actual data usage of a snapshot much faster.
  Config numeric value: 16
  CLI value: fast-diff
- Deep-flatten: Deep-flatten makes rbd flatten work on all the
  snapshots of an image, in addition to the image itself. Without it,
  snapshots of an image will still rely on the parent, so the parent
  will not be delete-able until the snapshots are deleted. Deep-flatten
  makes a parent independent of its clones, even if they have
  snapshots.
  Config numeric value: 32
  CLI value: deep-flatten
- Journaling: Journaling support depends on exclusive lock support.
  Journaling records all modifications to an image in the order they
  occur. RBD mirroring utilizes the journal to replicate a crash
  consistent image to a remote cluster.
  Config numeric value: 64
  CLI value: journaling"
```

## Các config của RBD-mirror
```sh 
rbd_journal_commit_age = 5.000000
rbd_journal_max_concurrent_object_sets = 0
rbd_journal_max_payload_bytes = 16384
rbd_journal_object_flush_age = 0.000000
rbd_journal_object_flush_bytes = 1048576
rbd_journal_object_flush_interval = 0
rbd_journal_object_max_in_flight_appends = 0
rbd_journal_object_writethrough_until_flush = true
rbd_journal_order = 24
rbd_journal_pool = 
rbd_journal_splay_width = 4
rbd_localize_parent_reads = false
rbd_localize_snap_reads = false
rbd_mirror_concurrent_image_deletions = 1
rbd_mirror_concurrent_image_syncs = 5
rbd_mirror_delete_retry_interval = 30.000000
rbd_mirror_image_policy_migration_throttle = 300
rbd_mirror_image_policy_rebalance_timeout = 0.000000
rbd_mirror_image_policy_type = simple
rbd_mirror_image_policy_update_throttle_interval = 1.000000
rbd_mirror_image_state_check_interval = 30
rbd_mirror_journal_commit_age = 5.000000
rbd_mirror_journal_max_fetch_bytes = 32768
rbd_mirror_journal_poll_age = 5.000000
rbd_mirror_leader_heartbeat_interval = 5
rbd_mirror_leader_max_acquire_attempts_before_break = 3
rbd_mirror_leader_max_missed_heartbeats = 2
rbd_mirror_perf_stats_prio = 5
rbd_mirror_pool_replayers_refresh_interval = 30
rbd_mirror_sync_point_update_age = 30.000000
rbd_mirroring_delete_delay = 0
rbd_mirroring_replay_delay = 0
rbd_mirroring_resync_after_disconnect = false
```
