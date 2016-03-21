# cloudsnap

A cirrus-based program for managing AWS EBS snapshots.

## Dependencies

Cloudsnap depends on github.com/nemac/cirrus; you must have cirrus installed and on your $PATH
in order to use cloudsnap.

## Overview

Cloudsnap connects to your AWS account and creates and/or deletes snapshots
of your EBS volumes according to a configured schedule.

The schedule is configured by tags that you set on the volumes; see below
for details.

Cloudsnap takes no arguments -- just run it with no args, and it will connect
to your AWS account, initiate any snapshots that need to be created, and
delete any that need to be deleted.

## Configuration Tags

Use the following tags on your EBS volumes to determine which volumes
get snapshotted, how often, nd for how long those snapshots are kept
around before being deleted.

  * `Backup` 
    Set the `Backup` tag to `true` (the word "true" spelled out) to indicate
    that snapshots should be made on a volume.  Cloudsnap will completely
    ignore any volumes that do not have a `Backup` tag value of `true`.
    
  * `Frequency` 
    The `Frequency` tag indicates how often a volume should be snapshotted;
    its value should be a positive number (integer or floating point) which
    indicates a number of days.  Whenever cloudsnap runs, it will initiate a
    snapshot of any volume whose most recent snapshot (with CIRRUS=true)
    is older than `Frequency` days.

  * `Retention` 
    The `Retention` tag indicates how long snapshots should be kept;
    its value should be a positive number (integer or floating point) which
    indicates a number of days.  Whenever cloudsnap runs, it will delete
    any snapshot for the volume (with CIRRUS=true) which is older than
    `Retention` days, except that it will never delete the last snapshot for
    a volume.

## Operating Frequency

Each time it runs, it decides which volumes to create new snapshots
for, if any, and which pre-existing snapshots should be deleted,
according to the above described tags and rules, and initiates the
commands to create and/or delete those snapshots.  If there are no
snapshots to be created or deleted, it simply exits.  So you can run
it as often as you want to maintain a collection of snapshots in
accordance with the configured Frequency and Retention settings for
each volume.  In particular, the frequency with which cloudsnap runs
is completely independent of the Frequency setting for any particular
volume.  In general, you should run cloudsnap at least as often as
your shortest volume Frequency setting; it is harmless (and probably a
good idea) to run it more often than that.

## Snapshot Tags

Cloudsnap assigns the tag value CIRRUS=true to any snapshots it creates,
and it only considers snapshots for which CIRRUS=true in deciding which
snapshots to create and/or delete.  So you can feel free to manually
create whatever snapshots you want without risk of cloudsnap deleting them
or getting confused by them, as long as you don't give them a CIRRUS=true
tag.