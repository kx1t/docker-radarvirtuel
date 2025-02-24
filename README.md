# Docker-RadarVirtuel

<!-- TOC -->

- [Docker-RadarVirtuel](#docker-radarvirtuel)
  - [What is it?](#what-is-it)
  - [Quick Start Guide](#quick-start-guide)
  - [Detailed Instructions](#detailed-instructions)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Parameters for `docker-radarvirtuel`](#environment-parameters-for-docker-radarvirtuel)
  - [MLAT Configuration](#mlat-configuration)
  - [Timezone configuration](#timezone-configuration)
  - [Starting, stopping, upgrading, and monitoring the container](#starting-stopping-upgrading-and-monitoring-the-container)
  - [Troubleshooting](#troubleshooting)
  - [Further help](#further-help)
  - [OWNERSHIP AND LICENSE](#ownership-and-license)

<!-- /TOC -->

## What is it?

This application is a feeder service that takes RAW/AVR ADSB data from a service like `dump1090[-fa]`, `readsb`, and `tar1090`, and feeds this data to `adsbnetwork`'s RadarVirtuel data aggregator. It works similar to other data connectors from the @Mikenye family of ADSB tools.

RadarVirtuel can be reached at:

- <http://www.radarvirtuel.com/>

Before you install this container, you need a **feeder key**. To request one, email <support@adsbnetwork.com> with the following information:

- Your name
- The Lat/Lon and nearest airport of your station
- Your Raspberry Pi model (or other hardware if not Raspberry Pi)
- Mention that you will feed using a Docker container.

## Quick Start Guide

These instructions are for people who are deploying on Raspberry Pi (3B+ or 4) with Raspberry OS or Ubuntu, and who already have a running `Docker` and `Docker-compose` setup with `readsb`, `dump1090-fa`, or a similar ADSB decoder running. That decoder can run inside a container or directly on the host.

You should also have received a Feeder Key as per the section above.

With these 4 simple steps, you should be up and running in 5 minutes or less. If you need more detailed instructions, please continue reading the next few sections of this README.

1. Download the [`docker-compose.yml`](docker-compose.yml) example file and add it to your existing `docker-compose.yml`, or run it stand-alone.
2. In `docker-compose.yml`, please check the `${HOSTNAME}` variable:
   - if you are using dockerized `readsb` or `tar1090` in the same docker-compose stack, then you can put in the name of the target container. Example: `SOURCE_HOST=readsb:30002`
   - if you are using `dump1090[-fa]`, `readsb`, or `tar1090` WITHOUT docker or in a different docker stack:
     - if it's on the local machine -- use the default value of `${HOSTNAME}:30002`
     - if it's on a different machine -- use the hostname or IP address of the target machine, for example `SOURCE_HOST=192.168.1.10:30002`
     - (Note - NEVER put "127.0.0.1" as the IP address - this won't work!)
3. Make sure that you add the following parameters to your `.env` file, in the same directory as `docker-compose.yml`. You may already have some of these params in that file, in which case it's not necessary to duplicate them. You should have received your feeder key value from <support@adsbnetwork.com>.

```config
RV_FEEDER_KEY=xxxx:123456789ABCDEF
FEEDER_LAT=12.345678
FEEDER_LONG=6.7890123
FEEDER_ALT_M=12.3
```

4. Restart your container stack with `docker-compose up -d` and you're in business. Monitor `docker logs -f radarvirtuel` to check for any errors.

## Detailed Instructions

## Prerequisites

1. The use of this connector service assumes that you already have a working ADS-B station setup

- Ensure you enabled RAW (=AVR) and BEAST data output on the application that actively processes your ADS-B data.
- Your ADS-B station can be on the same machine as this application, or on a different machine.
- Similarly, it doesn't matter if you are using a Containerized or non-Containerized setup.

2. The use of this connector also assumes that you have installed `Docker` and `Docker-compose` on the machine you want to run `RadarVirtuel` on.

- For a good overview on Docker, installation, etc., please read Mike Nye's excellent [gitbook](https://mikenye.gitbook.io/ads-b/).
- If you need to install `Docker` and/or `Docker-compose` on your machine, [go here](https://github.com/sdr-enthusiasts/docker-install) for a handy script that will take care of it for you

3. Last, you will need to get a `FEEDER_KEY` to identify your station to RadarVirtuel. See the "What is it?" section above for instructions on how to get this key.

## Installation

For a stand-alone installation on a machine with an existing ADS-B receiver, you can simply do this:

```bash
sudo mkdir -p /opt/adsb && sudo chmod a+rwx /opt/adsb && cd /opt/adsb
wget https://raw.githubusercontent.com/kx1t/docker-radarvirtuel/main/docker-compose.yml
```

Then, edit the `docker-compose.yml` file and the `.env` file in the same directory using the instructions above.

To add `RadarVirtuel` to an existing Docker Stack, simply copy and paste the relevant sections into your existing `docker-compose.yml` and `.env` files.

## Environment Parameters for `docker-radarvirtuel`

| Parameter         | Definition                                                                                                                                                                                                                                   | Value                                                                                                           |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `FEEDER_KEY`      | This key is provided by RadarVirtuel and is your PRIVATE KEY. Do no share this with anyone else.                                                                                                                                             | `[icao]:[private_key]`                                                                                          |
| `SOURCE_HOST`     | host and port number of your ADSB receiver. When running stand-alone on your local machine, this should be `${HOSTNAME}`. The value after the `:` is the port number to the RAW or AVR service on the target machine, most probably `30002`. | `${HOSTNAME}:30002`                                                                                             |
| `RV_SERVER`       | The hostname and the port of the RadarVirtuel server. You should NOT set this parameter unless specifically instructed. Note - this value may be overwritten/automatically updated if you are in a [predefined zone in North America](https://github.com/user-attachments/assets/c59b6026-3cc0-44ef-92b7-eec886555564) or [Europe](https://github.com/user-attachments/assets/1d76b061-cdfa-45f5-b531-b6030402e29a) | `mg22.adsbnetwork.com:50050`                                                                                     |
| `VERBOSE`         | Write verbose messages to the log                                                                                                                                                                                                            | `OFF` (default) / `ON`                                                                                          |
| `MLAT_SERVER`     | RV MLAT server address and port - do not change unless instructed to                                                                                                                                                                         | mlat.adsbnetwork.com:50000                                                                                      |
| `MLAT_HOST`       | This is the same hostname as for SOURCE_HOST, but now using port 30005                                                                                                                                                                       | `${HOSTNAME}:30005`                                                                                             |
| `MLAT_INPUT_TYPE` | Optional param to define MLAT Input Type - do not change unless instructed to                                                                                                                                                                | `auto` (default) / `dump1090` / `beast` / `radarcape_12mhz` / `radarcape_gps` / `radarcape` / `sbs` / `avrmlat` |
| `MLAT_RESULTS`    | Optional param to define how to make the MLAT results available. Value is used with the `--results` option of `mlat-client`. For valid options, do `docker exec -t radarvirtuel mlat-client --help`                                          | `beast,listen,30105` (default)                                                                                  |
| `LAT`             | This is your station latitude (used with MLAT)                                                                                                                                                                                               |                                                                                                                 |
| `LON`             | This is your station longitude (used with MLAT)                                                                                                                                                                                              |                                                                                                                 |
| `ALT`             | This is your antenna altitude above the ellipsoid (MSL). Use "ft" for feet or "m" for meters                                                                                                                                                 |                                                                                                                 |
| `ENABLE_MLAT`     | Enable MLAT                                                                                                                                                                                                                                  | `true` (default) / `false`                                                                                      |

## MLAT Configuration

By default, MLAT is switched ON in the container. Please make sure to configure a valid `MLAT_HOST`, `LAT`, `LON`, and `ALT` in your `docker-compose.yml` setup. If you want to switch off MLAT, simply set `ENABLE_MLAT` to false.

## Timezone configuration

- The default timezone setting for the container mimics the host machine's timezone. Sometimes, it is desired to run the container in UTC instead.
- To run the container in UTC, comment out the following lines (using `#`) in `docker-compose.yml`:

```yaml
#    volumes:
#      - "/etc/localtime:/etc/localtime:ro"
#      - "/etc/timezone:/etc/timezone:ro"
```

## Starting, stopping, upgrading, and monitoring the container

To start the container for the first time:

- `pushd /opt/adsb && docker-compose up -d && popd`

To restart the container:

- `docker restart radarvirtuel`

To stop the container:

- `pushd /opt/adsb && docker-compose down && popd` <-- this stops all containers in the stack
- `docker stop radarvirtuel` <-- this stops only RadarVirtuel

To download and deploy a new version of the container, if one exists:

- `pushd /opt/adsb && docker-compose pull && docker-compose up -d && popd`

To monitor the logs of the RadarVirtuel container:

- `docker logs radarvirtuel` <-- shows all logs for RadarVirtuel in the buffer (warning - can be very long!)
- `docker logs -f radarvirtuel` <-- shows the last few logs and waits for any new log entries, abort with CTRL-C

## Troubleshooting

- If things don't appear to work as they should, set `VERBOSE=ON` in your `docker-compose.yml` and restart the container. Then monitor `docker logs -f radarvirtuel` to see what is going on
- If it complains about your `SOURCE_HOST`:
  - Make sure your data source can be reached from inside the container. `SOURCE_HOST` should be set to:
    - if it's a container within the same stack: the container name, for example `SOURCE_HOST=readsb:30002`
    - if it's a separate container or a stand-alone installation on the same machine, `SOURCE_HOST` must be set to `SOURCE_HOST=${HOSTNAME}:30002`
    - If it's a separate container on a different machine, you should use the full hostname or IP address of that machine. For example `SOURCE_HOST=192.168.1.10:30002`.
  - Make sure that your `readsb` or `dump1090[-fa]` installation is providing RAW (AVR) data:
    - Stand-alone `dump1090` variants must use the `--net-ro-port 30002` command line parameter
    - Containerized `readsb` or `readsb-protobuf` should pass the following variable in the `environment:` section of `docker-compose.yml`: `READSB_NET_RAW_OUTPUT_PORT=30002`
- If the logs complain about not being able to reach the RadarVirtuel server:
  - Make sure your machine is connected to the internet. Try this command to check connectivity:
    `netcat -u -z -v mg2.adsbnetwork.com 50050`
- If it complains about errors in your `docker-compose.yml` file
  - SPACING IS IMPORTANT. Spacing is important. There, I said it. Check that each line has the correct indentation
  - Sometimes, there are problems with newline characters when you import from a Windows machine. In that case, you can easily convert your file to Unix format. There are multiple ways to do this, one of them is:
    `cat docker-compose.yml | tr -dc '[:print:]\n' >/tmp/tmp.tmp && sudo mv -f /tmp/tmp.tmp docker-compose.yml`

## Further help

- For help with credentials and service outages, please email <support@adsbnetwork.com>
- For help with the Docker Container and related issues, please contact kx1t on this Discord channel: <https://discord.gg/m42azbZydy>

## OWNERSHIP AND LICENSE

RADARVIRTUEL is owned by, and copyright by AdsbNetwork and by Laurent Duval. All rights reserved.
Note that parts of the code and scripts included with this package are NOT covered by an Open Source license, and may only be distributed and used with express permission from AdsbNetwork. Contact <support@adsbnetwork.com> for more information.

Any modifications to the existing AdsbNetwork scripts and programs, and any additional scripts are provided by `kx1t` under the MIT License as included with this package.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
