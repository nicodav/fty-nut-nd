# fty-nut

fty-nut is a family of agents responsible for 42ITy interaction with NUT (see
[http://www.networkupstools.org]) including both collection of device data
and configuration of NUT to monitor new devices as assets are created.

## To build fty-nut project run:

```bash
./autogen.sh
./configure
make
make check # to run self-test
```
Compilation of fty-nut creates two binaries _fty-nut_ and _fty-nut-configurator_, which are run by systemd service.

## How to run

To run fty-nut project:

* from within the source tree, run:

```bash
./src/fty-nut --mapping-file <path_to_mapping_file> --state-file <path_to_state_file>
./src/fty-nut --mapping-file /usr/share/fty-nut/mapping.conf --state-file /var/lib/fty/fty-nut/state_file

./src/fty-nut-configurator
```

* from an installed base, using systemd, run:

```bash
systemctl start fty-nut
systemctl start fty-nut-configurator
```

### Configuration file

To configure fty-nut, a two configuration files exist: _fty-nut.cfg_ and _fty-nut-configurator.cfg_.
Both contain standard configuration directives, under the server sections. Additional parameter

* fty-nut.cfg
  * polling_interval - polling interval in seconds. Default value: 30 s

### Mapping file
Mapping between NUT and fty-nut is saved in:

```
/usr/share/fty-nut/mapping.conf
```

### State File
State files are located in

```
/var/lib/fty/fty-nut/state_file  (fty-nut)
/var/lib/fty/fty-autoconfig/state (fty-nut-configurator)

```

## Architecture

### Overview

fty-nut is composed of three actors:

* fty_nut_server - server actor
* alert_actor - actor handling device alerts and thresholds coming from NUT
* sensor_actor - actor handling sensor measurements coming from NUT.

fty-nut-configurator is composed of 1 actor:

* fty_nut_configurator_server - server actor which configures nut-server (upsd) based on results from nut scanner

## Protocols

### Publishing Metrics

* sensor_actor produces metrics from sensors connected to power devices on FTY_PROTO_STREAM_METRICS_SENSOR.

```
stream=_METRICS_SENSOR
sender=agent-nut-sensor
subject=humidity.0@epdu-77
D: 17-11-02 12:18:16 FTY_PROTO_METRIC:
D: 17-11-02 12:18:16     aux=
D: 17-11-02 12:18:16         port=0
D: 17-11-02 12:18:16     time=1509625096
D: 17-11-02 12:18:16     ttl=60
D: 17-11-02 12:18:16     type='humidity.0'
D: 17-11-02 12:18:16     name='epdu-77'
D: 17-11-02 12:18:16     value='37.60'
D: 17-11-02 12:18:16     unit='%'
```

```
stream=_METRICS_SENSOR
sender=agent-nut-sensor
subject=status.GPI1.0@epdu-76
D: 17-11-13 15:21:57 FTY_PROTO_METRIC:
D: 17-11-13 15:21:57     aux=
D: 17-11-13 15:21:57         sname=sensorgpio-81
D: 17-11-13 15:21:57         port=0
D: 17-11-13 15:21:57         ext-port=1
D: 17-11-13 15:21:57     time=1510586517
D: 17-11-13 15:21:57     ttl=60
D: 17-11-13 15:21:57     type='status.GPI1.0'
D: 17-11-13 15:21:57     name='epdu-76'
D: 17-11-13 15:21:57     value='closed'
D: 17-11-13 15:21:57     unit=''
```
alerts for sensors are managed by fty-alert-engine (environmental sensors) and fty-alert-flexible (GPI sensors)

* fty_nut_server produces metrics on FTY_PROTO_STREAM_METRICS.

```
stream=METRICS
sender=fty-nut
subject=status.outlet.2@ups-52
D: 17-11-13 12:53:21 FTY_PROTO_METRIC:
D: 17-11-13 12:53:21     aux=
D: 17-11-13 12:53:21     time=1510577601
D: 17-11-13 12:53:21     ttl=60
D: 17-11-13 12:53:21     type='status.outlet.2'
D: 17-11-13 12:53:21     name='ups-52'
D: 17-11-13 12:53:21     value='42'
D: 17-11-13 12:53:21     unit=''
```

### Publishing Alerts

* alert_actor produces metrics on FTY_PROTO_STREAM_ALERT_SYS.

```
stream=_ALERTS_SYS
sender=bios-nut-alert
subject=outlet.group.1.voltage@epdu-54/OKG@epdu-54
D: 17-11-13 15:05:57 FTY_PROTO_ALERT:
D: 17-11-13 15:05:57     aux=
D: 17-11-13 15:05:57     time=1510560758
D: 17-11-13 15:05:57     ttl=90
D: 17-11-13 15:05:57     rule='outlet.group.1.voltage@epdu-54'
D: 17-11-13 15:05:57     name='epdu-54'
D: 17-11-13 15:05:57     state='RESOLVED'
D: 17-11-13 15:05:57     severity='OK'
D: 17-11-13 15:05:57     description='outlet.group.1.voltage is resolved'
D: 17-11-13 15:05:57     action=''
```

### Consuming Assets

* fty_nut_server, sensor_actor and alert_actor listen on FTY_PROTO_STREAM_ASSETS stream.

### Mailbox Requests
No mailbox commands implemented
