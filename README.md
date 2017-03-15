# sbt-pillar - manage Cassandra migrations from sbt

[![Build Status](https://travis-ci.org/zendesk/sbt-pillar-plugin.svg?branch=master)](https://travis-ci.org/zendesk/sbt-pillar-plugin)

A rewrite of the plugin https://github.com/inoio/sbt-pillar-plugin and added:
* Allow use of Authentication credentials.
* Allow passing in hosts as a comma-separated string.
* Allow creating of `cql` migration files.
* Will allow use of `NetworkTopologyStrategy` for creating keyspaces.

This fork also adds [Consul](https://www.consul.io/) support for retrieving the host list, using a service tag.

This sbt plugin enables running Cassandra schema/data migrations from sbt (using [pillar](https://github.com/comeara/pillar)). For details on migration files check out the [pillar documentation](https://github.com/comeara/pillar#migration-files).

The plugin is built for sbt 0.13.6+.

## Installation

To install this fork of the plugin you have to add it to `project/plugins.sbt`:

```
addSbtPlugin("com.zendesk" %% "sbt-pillar" % "0.3.0")
```

## Configuration

Add appropriate configuration to `build.sbt` like this:

```
pillarConfigFile in ThisBuild := file("db/pillar.conf")
pillarMigrationsDir in ThisBuild := file("db/migrations")
```

The shown configuration assumes that the settings for your cassandra are configured in `db/pillar.conf` (relative to `build.sbt`), and that pillar migration files are kept in `db/migrations`. For the format of migration files check out the [pillar documentation](https://github.com/comeara/pillar#migration-files).

An example configuration file (based on [typesafe-config](https://github.com/typesafehub/config)) is:

```
development {
  cassandra {
    keyspace = "pigeon"
    hosts = "localhost"
    port = 9042
    replicationFactor = 1
    defaultConsistencyLevel = 1
    replicationStrategy = "SimpleStrategy"
    consul {
      host = "localhost"
      port = 8500
      url = "http://localhost:8500"
      service = "cassandra"
      tag = ""
    }
  }
}

test = ${development}
test {
  cassandra {
    keyspace = "pigeon_test"
  }
}

master {
  cassandra {
    hosts = ${?CASSANDRA_HOSTS}
    port = 9042
    keyspace = ${?CASSANDRA_KEYSPACE}
    username = ${?CASSANDRA_USERNAME}
    password = ${?CASSANDRA_PASSWORD}
    replicationFactor = ${?CASSANDRA_REPLICATION_FACTOR}
    defaultConsistencyLevel = ${?CASSANDRA_CONSISTENCY_LEVEL}
    replicationStrategy = "SimpleStrategy"
  }
}

staging = ${master}
production = ${master}
```

## Usage

The sbt pillar plugin provides the following tasks:

<dl>
<dt>createKeyspace</dt><dd>Creates the keyspace (and creates pillar's <code>applied_migrations</code> table)</dd>
<dt>dropKeyspace</dt><dd>Drops the keyspace</dd>
<dt>migrate</dt><dd>Runs pillar migrations (assumes <code>createKeyspace</code> was run before)</dd>
<dt>cleanMigrate</dt><dd>Recreates the keyspace (drops if exists && creates) and runs pillar migrations (useful for continuous integration scenarios)</dd>
<dt>createMigration</dt><dd>Create a new migration file from the name passed in.</dd>
</dl>

For example:

```bash
$ sbt 'createMigration create_user_settings'
[info] Loading config file: db/pillar.conf for environment: development
[info] Creating migration file: db/migrations/20160520105550_create_user_settings_timeseries.cql
[success] Created migration 'db/migrations/20160520105550_create_user_settings_timeseries.cql'
```

```bash
$ sbt cleanMigrate
[info] Loading config file: db/pillar.conf for environment: development
[info] Dropping keyspace pigeon at 192.168.42.45:9042
[info] Creating keyspace pigeon at 192.168.42.45:9042
[info] Migrating keyspace pigeon at 192.168.42.45:9042
[success] Total time: 10 s, completed 20-May-2016 11:07:05
```

## To Do

Currently only the replication strategy 'SimpleStrategy' works. This will be resolved in next version.

## License

The license is MIT (https://opensource.org/licenses/MIT). Have at it!
