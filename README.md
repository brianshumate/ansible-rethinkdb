# RethinkDB

This role installs RethinkDB on nodes to form a clustered environment.

Define you target cluster nodes in the host inventory file; example host
inventory, playbooks, and a `Vagrantfile` are in the `examples` directory.

## Requirements

This role has basic functionality verified with the following software:

* RethinkDB (version 2.2.4)
* Ansible (version 2.0.0.2)
* CentOS (version 7)
* Debian (version 8)
* Ubuntu (version 14.04)

## Role Variables

In cases where you want simple clusters for development or other
non-production use, the default values for this role's common variables can
be left as-is.

Should you need specific performance or otherwise wish to tweak them
for your particular purpose, this section describes all the user editable
variables in detail including their default values for your reference.

### Common Variables

| Name                                 | Default                                                                                   | Description                             |
| ------------------------------------ | ----------------------------------------------------------------------------------------- | --------------------------------------- |
| rethinkdb_runuser | `rethinkdb` | RethinkDB OS user |
| rethinkdb_rungroup | `rethinkdb` | RethinkDB OS group |
| rethinkdb_cache_size | `1024` | The cache size in megabytes |
| rethinkdb_cores | `2` | Number of CPU cores to allocate |
| rethinkdb_io_threads | `64` | Number of disk I/O threads to allocate |
| rethinkdb_admin_port | `8080` | Administration and web console port |
| rethinkdb_driver_port | `28015` | Driver port |
| rethinkdb_cluster_port | `29015` | Intracluster communication port |
| rethinkdb_filesystem | `ext4` | Default filesystem for data and index volumes |
| rethinkdb_mountpoint | `/` | Logical volume mountpoint |
| rethinkdb_partition | `/dev/mapper/VolGroup-lv_root` | Logical volume partition |
| rethinkdb_mount_options | `noatime,barrier=0,errors=remount-ro` | Additional mount options |
| rethinkdb_data_path | `/var/lib/rethinkdb/default` | Path to data files |
| rethinkdb_log_path | `/var/log/rethinkdb` | Path to log file |

### Special Variables

*Set the following variables with caution* as they have potential
negative performance implications; do not enable them without knowledge
of the OS level changes applied:

| Name                                 | Default  | Description                                    |
| ------------------------------------ | -------- | ---------------------------------------------- |
| rethinkdb_tune_os          | `false`    | Whether to tune OS with optimized settings |

## Examples

The `examples` directory contains host inventory examples, some basic
playbooks and a `Vagrantfile` (primarily aimed at Mac OS X development use):

* `build_cluster.yml` installs RethinkDB and creates a cluster
* `example_hosts` example hosts inventory in format required by this project
* `Vagrantfile` example Vagrant development cluster definition
* `centos` CentOS hosts inventory for Vagrant based development cluster
* `debian` Debian hosts inventory for Vagrant based development cluster
* `redhat` Red Hat hosts inventory for Vagrant based development cluster
* `ubuntu` Ubuntu hosts inventory for Vagrant based development cluster

### Quick Start for Simple Vagrant Based Cluster

Follow these steps to have a simple 3 node development or evaluation
cluster on a >= 8GB Mac with Vagrant and VirtualBox:

1. export ROLEPATH=ANSIBLE_ROLE_PATH
2. Edit `/etc/hosts` or use included `examples/bin/preinstall` script to add
   the following entries to your development system's `/etc/hosts` file:
 * 10.1.42.101 node1.local node1
 * 10.1.42.102 node2.local node2
 * 10.1.42.103 node3.local node3
3. `ansible-galaxy install brianshumate.rethinkdb`
4. cd $ROLEPATH/brianshumate.rethinkdb/examples
5. vagrant plugin install vagrant-hosts
6. vagrant up

This will install three (3) Debian 8 nodes with 1.5GB RAM each and cluster
them together. The nodes will be available at 10.1.42.101, 10.1.42.102, and
10.1.42.103 as defined in the `Vagrantfile`.`

To install Ubuntu based nodes, change the command in step 6 to:

```
BOX_NAME=ubuntu/trusty64 CLUSTER_HOSTS=ubuntu vagrant up
```

*NOTE*: Due to [issue 83](https://github.com/rethinkdb/rethinkdb/issues/83)
you will see a duplicate database error when initially accessing the cluster
since the default *test* database is created on each of the 3 nodes before
the cluster is formed. Just rename/delete the databases as needed to work
around this.

## Dependencies

None

## License

BSD

## Author Information

- Brian Shumate (brian at brianshumate.com)
