# Terraform OpenStack support for Trove

## Data Sources

### OpenStack Flavor

Command `data.openstack_flavor`

https://developer.openstack.org/api-ref/database/#list-flavors

```go
data "openstack_flavor" "tiny" {
  account_id = "${var.openstack-account-id}"
  filter {
    name = "name"
    value = "*.tiny"
  }
}

resource "openstack_db_instance" "main" {
  flavor = "${data.openstack_db_flavor.tiny.id}"
  ...
}
```

Attributes:

* `ephemeral` => 0
* `id` => 1
* `name` => "m1.tiny"
* `ram` => 512
* `str_id` => "1"
* `vcpus` => 10
* `disk` => 12
* `link` => "https://troveapi.org/flavors/1"


### Trove Datastore

Command `data.openstack_db_datastore`

https://developer.openstack.org/api-ref/database/#list-datastore-versions

Arguments:

* `datastore_name` (Optional), path, string, The name of the data store.
* `account_id` (Optional), path, string, The account ID of the owner of the instance.

```go
data "openstack_db_datastore" "mysql56" {
  account_id = "${var.openstack-account-id}"
  filter {
    name = "datastore_name"
    value = "mysql5.*"
  }
}

data "openstack_db_datastore" "mysql56" {
  account_id = "${var.openstack-account-id}"
  datastore_name = "mysql5.6"
}
```



Attributes:

* `name` => "5.6"
* `link` => "https://10.240.28.38:8779/datastores/versions/2dc7faa0-efff-4c2b-8cff-bcd949c518a5"
* `image"` => "b69fbd9e-b31d-46ff-8afb-cbf452f6f835"
* `active"` => 1
* `datastore"` => "3a8968d8-e5f5-4452-83ca-f6c90b5de06a"
* `packages"` => "mysql-server-5.6"
* `id"` => "2dc7faa0-efff-4c2b-8cff-bcd949c518a5"


## Resources

### Trove Instance

Command: `openstack_db_instance`

https://developer.openstack.org/api-ref/database/#create-database-instance

Arguments:

* `users`, body, array, A users object.
* `password` (Optional), body, string, The password for those users on instance creation.
* `datastore_version` (Optional), body, string, Name of the datastore version to use when creating the instance.
* `name`, body, string, Name of the configuration group you are creating.
* `flavorRef`, body, string, Reference (href), which is the actual URI to a flavor as it appears in the list flavors response. Rather than the flavor URI, you can also pass the flavor ID (integer) as the flavorRef value. For example, 1.
* `characterSet` (Optional), body, string, A set of symbols and encodings. Default is utf8. For information about supported character sets and collations, see Character Sets and Collations in MySQL.
* `replica_count` (Optional), body, integer, Number of replicas to create (defaults to 1).
* `instance`, body, object, An instance object.
* `collate` (Optional), body, string, A set of rules for comparing characters in a character set. Default is `utf8_general_ci`. For information about supported character sets and collations, see Character Sets and Collations in MySQL.
* `databases` (Optional), body, array, A databases object.
* `datastore`, body, string, Data store assigned to the configuration group. Required if you did not configure the default data store.
* `configuration`, body, string, ID of the configuration group that you want to attach to the instance.
* `type` (Optional), body, string, The volume type to use. You can list the available volume types on your system by using the cinder type- list command. If you want to specify a volume type, you must also specify a volume size.
* `replica_of` (Optional), body, string, ID or name of an existing instance to replicate from.
* `size`, body, integer, The volume size, in gigabytes (GB). A valid value is from 1 to 50.
* `accountId` (Optional), path, string, The account ID of the owner of the instance.

* enable/disable root user

```go
resource "openstack_db_instance" "main" {
  name = "json_rack_instance"
  root_password = "${var.trove-user-root-password}"

  account_id = "${var.openstack-account-id}"
  configuration = "${openstack_db_config.main.id}"
  instance = "${openstack_instance.main}"
  flavor = "${openstack_flavor.main.id}"
  #or, flavor = "1"
  #replica_of = "${openstack_db_instance.main.id}"
  volume_size = "2"
}

resource "openstack_db_database" "users" {
  name = "testingdb"
  character_set = "utf8"
  collate = "utf8_general_ci"
  instance = "${openstack_db_instance.main}" # (optional)
}

resource "openstack_db_user" "main" {
  username = "dbuser1"
  password = "password"
  databases = [
    "${openstack_db_database.users.name}"
  ]
  instance = "${openstack_db_instance.main}"
}

```

Attributes:

https://developer.openstack.org/api-ref/database/#show-database-instance-details

* `created` => "2014-10-30T12:30:00"
* `datastore_type` => "mysql"
* `datastore_version` => "5.5"
* `flavor_id` => "1"
* `hostname` => "e09ad9a3f73309469cf1f43d11e79549caf9acf2.troveexampledb.com"
* `id` => "44b277eb-39be-4921-be31-3d61b43651d7"
* `link` => "https://troveapi.org/instances/44b277eb-39be-4921-be31-3d61b43651d7",
* `name` => `"json_rack_instance"`
* `region` => "RegionOne"
* `status` => "ACTIVE"
* `updated` => "2014-10-30T12:30:00"
* `volume_size` => 2
* `volume_used` => 0.16



### Trove Database

Command: `openstack_db_database`

https://developer.openstack.org/api-ref/database/#create-database

Arguments:
* characterSet (Optional), body, string, A set of symbols and encodings. Default is utf8. For information about supported character sets and collations, see Character Sets and Collations in MySQL.
* collate (Optional), body, string, A set of rules for comparing characters in a character set. Default is `utf8_general_ci`. For information about supported character sets and collations, see Character Sets and Collations in MySQL.
* name, body, string, Name of the configuration group you are creating.
* instanceId (Optional), path, string, The ID for the database instance.
* accountId (Optional), path, string, The account ID of the owner of the instance.

```go
resource "openstack_db_database" "users" {
  name = "testingdb"
  character_set = "utf8"
  collate = "utf8_general_ci"
  instance = "${openstack_db_instance.main}" # (optional)
  account_id = "${var.openstack-account-id}" # (optional)
}
```

Attributes:

* `character_set` => "utf8"
* `collate` => `"utf8_general_ci"`
* `name` => "testingdb"


### Trove User

Command : `openstack_db_user`

https://developer.openstack.org/api-ref/database/#create-user

Arguments:
* username, Name of the user for the database.
* password, User password for database access.
* (optional) database, name, Name of the database(s) that the user can access. You can specify one or more database names.

```go
resource "openstack_db_user" "main" {
  username = "dbuser1"
  password = "password"
  databases = [
    "databaseA",
    "${openstack_db_database.main.name}"
  ]
}
```

Attributes:

https://developer.openstack.org/api-ref/database/#list-database-instance-users

* `databases (array)` => `[ "anotherdb1", "anotherdb2" ]`
* `host` => "%"
* `username` => "dbuser1"


### Trove Configuration

Command `openstack_db_config_group`

Arguments:

* datastore, body, string, Data store assigned to the configuration group. Required if you did not configure the default data store.
* values, body, string, Dictionary that lists configuration parameter names and associated values.
* name, body, string, Name of the configuration group you are creating.
* accountId (Optional), path, string, The account ID of the owner of the instance.

https://developer.openstack.org/api-ref/database/#create-configuration-group

```go
resource "openstack_db_config_group" "main" {
  name = "group1"

  datastore = "${openstack_db_datastore.main.id}"
  # or
  # datastore = "type:mysql"

  configuration {
    name = "sync_binlog"
    value = "1"
  }

  configuration {
    name = "connect_timeout"
    value = 17
  }
}
```

Attributes:

https://developer.openstack.org/api-ref/database/#show-configuration-group-details

* `updated` => `2015-07-01T16:38:27`
* `name` => `group1`
* `created` => `2015-07-01T16:38:27`
* `datastore_version_name` => `5.6`
* `id` => `2aa51628-5c42-4086-8682-137caffd2ba6`
* `datastore_version_id` => `2dc7faa0-efff-4c2b-8cff-bcd949c518a5`

