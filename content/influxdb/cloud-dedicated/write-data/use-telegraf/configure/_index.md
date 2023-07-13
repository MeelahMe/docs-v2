---
title: Configure Telegraf for InfluxDB
seotitle: Configure Telegraf to write data to InfluxDB
description: >
  Telegraf is a plugin-based agent with plugins that are enabled and configured in
  your Telegraf configuration file (`telegraf.conf`).
  Update existing or create new Telegraf configurations to use the `influxdb_v2`
  output plugin to write to InfluxDB.
  Start Telegraf using the custom configuration.
menu:
  influxdb_cloud_dedicated:
    name: Configure Telegraf
    parent: Use Telegraf
weight: 101
influxdb/cloud-dedicated/tags: [telegraf]
related:
  - /{{< latest "telegraf" >}}/plugins/
alt_engine: /influxdb/cloud/write-data/no-code/use-telegraf/manual-config/
aliases:
  - /influxdb/cloud-dedicated/write-data/use-telegraf/manual-config/
---

Use the Telegraf `influxdb_v2` output plugin to collect and write metrics to
{{< cloud-name >}}.
Learn how to enable the plugin in new and
existing Telegraf configurations,
and then start Telegraf using the custom configuration file.

{{% note %}}
_View the [requirements](/influxdb/cloud-dedicated/write-data/use-telegraf#requirements)
for using Telegraf with {{< cloud-name >}}._
{{% /note %}}

<!-- TOC -->

- [Configure Telegraf input and output plugins](#configure-telegraf-input-and-output-plugins)
  - [Add Telegraf plugins](#add-telegraf-plugins)
  - [Enable and configure the InfluxDB v2 output plugin](#enable-and-configure-the-influxdb-v2-output-plugin)
      - [urls](#urls)
      - [token](#token)
      - [organization](#organization)
      - [bucket](#bucket)
      - [Write to InfluxDB v1.x and {{< cloud-name >}}](#write-to-influxdb-v1x-and--cloud-name-)
  - [Other Telegraf configuration options](#other-telegraf-configuration-options)
- [Start Telegraf](#start-telegraf)

<!-- /TOC -->

## Configure Telegraf input and output plugins

Configure Telegraf input and output plugins in the Telegraf configuration file (typically named `telegraf.conf`).
Input plugins collect metrics.
Output plugins define destinations where metrics are sent.

This guide assumes you followed [Setup instructions](/influxdb/cloud-dedicated/get-started/setup/) in the Get Started tutorial
to set up InfluxDB and [configure authentication credentials](/influxdb/cloud-dedicated/get-started/setup/?t=Telegraf).

### Add Telegraf plugins

To add any of the available [Telegraf plugins](/{{< latest "telegraf" >}}/plugins/), follow the steps below.

1.  Find the plugin you want to enable from the complete list of available
    [Telegraf plugins](/{{< latest "telegraf" >}}/plugins/).
2.  Click **View** to the right of the plugin name to open the plugin page on GitHub.
    For example, view the [MQTT plugin GitHub page](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/mqtt_consumer/README.md).
3.  Copy and paste the example configuration into your Telegraf configuration file
    (typically named `telegraf.conf`).

### Enable and configure the InfluxDB v2 output plugin

To send data to {{< cloud-name >}}, enable the
[`influxdb_v2` output plugin](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/influxdb_v2/README.md)
in the `telegraf.conf`.

{{% code-placeholders "DATABASE_NAME" %}}
```toml
[[outputs.influxdb_v2]]
  urls = ["https://cluster-id.influxdb.io"]
  # INFLUX_TOKEN is an environment variable you created for your database WRITE token

  token = "${INFLUX_TOKEN}"
  organization = ""
  bucket = "DATABASE_NAME"
```
{{% /code-placeholders %}}

Replace the following:

- **`DATABASE_NAME`**: the name of the InfluxDB [database](/influxdb/cloud-dedicated/admin/databases/) to write data to

The InfluxDB output plugin configuration contains the following options:

##### urls

An array of URL strings.
To write to {{% cloud-name %}}, include your {{% cloud-name %}} cluster URL using the HTTPS protocol:

```toml
["https://cluster-id.influxdb.io"]
```

##### token

Your {{% cloud-name %}} [database token](/influxdb/cloud-dedicated/admin/tokens/) with _write_ permission to the database.

In the examples, **`INFLUX_TOKEN`** is an environment variable assigned to a [database token](/influxdb/cloud-dedicated/admin/tokens/) that has _write_ permission to the database.

##### organization

For {{% cloud-name %}}, set this to an empty string (`""`).

##### bucket

The name of the {{% cloud-name %}} database to write data to.

{{% note %}}
##### Write to InfluxDB v1.x and {{< cloud-name >}}

If a Telegraf agent is already writing to an InfluxDB v1.x database,
enabling the InfluxDB v2 output plugin will write data to both v1.x and your {{< cloud-name >}} cluster.
{{% /note %}}

### Other Telegraf configuration options

`influx_uint_support`: supported in InfluxDB IOx.

For more plugin options, see [`influxdb`](https://github.com/influxdata/telegraf/blob/master/plugins/outputs/influxdb/README.md) on GitHub.

## Start Telegraf

Start the Telegraf service using the `--config` flag to specify the location of your `telegraf.conf`.

```sh
telegraf --config /path/to/custom/telegraf.conf
```