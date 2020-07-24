# Replication Notes

Assumption: You have two clusters, access to both, and a pool that
exists in both clustsers and you wish to replicate some or all images in
that pool to the other cluster.

Mirroring in both directions is required for Cinder to properly
implement failover and failback.

Make sure you have the `rbd-mirror` package installed.

## Run the Mirror Daemon

I assume you already have the MONs and OSDs deployed, running, and in a
healthy state.  The mirror daemon doesn't technically have to run on the
same node as the MON or OSD, but I'm developing with single-node
clusters so this is my setup.

### On the primary cluster

    $ rbd-mirror --cluster primary --setuser ceph --setgroup ceph

### On the secondary cluster

    $ rbd-mirror --cluster=primary --setuser ceph --setgroup ceph

## Enable Mirroring (per-image on a specific pool)

This will enable image-mode mirroring on a pool

    $ rbd --cluster primary   mirror pool enable volumes image
    $ rbd --cluster secondary mirror pool enable volumes image

    $ rbd --cluster primary mirror pool info volumes
    Mode: image
    Peers: none

    $ rbd --cluster secondary mirror pool info volumes
    Mode: image
    Peers: none

## Configure Mirror Peers

    $ rbd --cluster primary mirror pool peer add volumes client.admin@secondary
    506b17d9-5157-4c6a-a63e-94617fb34ea6

    $ rbd --cluster secondary mirror pool peer add volumes client.admin@primary
    b2853704-2536-4cb3-860d-51622dcaf353

## Verify Mirror Configuration

    $ rbd --cluster primary mirror pool info volumes
    Mode: image
    Peers:
      UUID                                 NAME      CLIENT
      506b17d9-5157-4c6a-a63e-94617fb34ea6 secondary client.admin

    $ rbd --cluster secondary mirror pool info volumes
    Mode: image
    Peers:
      UUID                                 NAME    CLIENT
      b2853704-2536-4cb3-860d-51622dcaf353 primary client.admin

From here, the cinder driver will take care of the rest.  It will enable
per-image replication for volumes created normally, from an existing
clone, and from retyping.

## Manual Image Mirroring

    $ rbd --cluster primary create volumes/test --size 1M
    $ rbd info volumes/test
    rbd image 'a':
            size 1024 kB in 1 objects
            order 22 (4096 kB objects)
            block_name_prefix: rbd_data.3ba2d74b0dc51
            format: 2
            features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
            flags:

    $ rbd --cluster primary feature enable volumes/test journaling
    $ rbd --cluster primary mirror image enable volumes/test

    $ rbd --cluster primary ls -l volumes
    NAME  SIZE PARENT FMT PROT LOCK 
    a    1024k          2

    $ rbd --cluster secondary ls -l volumes
    NAME  SIZE PARENT FMT PROT LOCK
    a    1024k          2      excl

    $ rbd info volumes/test
    rbd image 'test':
            size 1024 kB in 1 objects
            order 22 (4096 kB objects)
            block_name_prefix: rbd_data.3ba2d74b0dc51
            format: 2
            features: layering, exclusive-lock, object-map, fast-diff, deep-flatten, journaling
            flags: 
            journal: 3ba2d74b0dc51
            mirroring state: enabled
            mirroring global id: aba6f0d2-b044-4ed1-9dcf-d5c9918386a3
            mirroring primary: true

## Manual Failover

    $ rbd --cluster secondary mirror image promote --force volumes/test

## Manual Failback

    $ <resync required>

## Cinder Configuration

The Ceph backend definition in `cinder.conf` must be updated to define the
secondary cluster:

    replication_device = backend_id:secondary,
                         conf:/etc/ceph/secondary.conf
                         user:cinder,
                         pool:volumes

## Cinder Type Configuration

The Cinder RBD driver must be told to enable replication for a
particular volume.  This is done with volume types.

    $ cinder type-create REPL
    $ cinder type-key    REPL set volume_backend_name=ceph
    $ cinder type-key    REPL set replication_enabled='<is> True'

## Replicated Volume Creation

    $ cinder create --volume-type REPL --name fingers-crossed 1

## Cinder Failover

    $ cinder failover-host client@ceph
