# Runespreader

A python runescape trading application

## Installing

### From PyPI

```
pip install runeascend-platform
```

### From Project

```
make create-dev
```

## Running the applets

I recommend setting up systemd services (ideally [user services](https://wiki.archlinux.org/title/Systemd/User) - *make sure to enable linger!*) for all of the apps like such:

OSRS Discord Bot:
```
[Unit]
Description= OSRS Bot
[Service]
User=<your_user_here>
ExecStart= /bin/bash -c "source /home/<your_user_here>/.venvs/runespreader/bin/activate && discord_bot"
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

Scraping script to populate clickhouse:
```
[Unit]
Description= OSRS GE Archiver
[Service]
User=<your_user_here>
ExecStart= /bin/bash -c "source /home/<your_user_here>/.venvs/runespreader/bin/activate && archiver"
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```

TradeSeeker - The simple opportunity finder bot
```
[Unit]
Description= OSRS GE Market Maker
[Service]
User=charles
ExecStart= /bin/bash -c "source /home/<your_user_here>/.venvs/runespreader/bin/activate && trade_seeker"
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target
```
Kafka Publisher - Distribute aggregated data to downstream consumers
```
[Unit]
Description= OSRS Publisher
[Service]
User=<your_user_here>
ExecStart= /bin/bash -c "source /home/<your_user_here>/.venvs/runespreader/bin/activate && publisher"
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target 
```
Runewriter - Kafka Consumer that writes to clickhouse
```
[Unit]
Description= OSRS Kafka Consumer
[Service]
User=<your_user_here>
WorkingDirectory=/home/<your_user_here>/tmp # There is a directory emitted from faust while running so you need to specify a location
ExecStart= /bin/bash -c "source /home/<your_user_here>/.venvs/runespreader/bin/activate && consumer worker -l info"
Restart=on-failure
RestartSec=5s
[Install]
WantedBy=multi-user.target 
```


## Setting up clickhouse

Installing and configuring [clickhouse](https://clickhouse.com/docs/en/install#quick-install)

Migrations can be found in migrations/ and are managed by [clickhouse-migrations](https://github.com/zifter/clickhouse-migrations)

### Tables

#### OSRS API Data

- rs_buys
- rs_sells

#### Runespreaders Published Messages

 - osrs_hf_opp
 - osrs_mf_opp
 - osrs_mkt_data
 - osrs_sweeps

 #### Runevault/Runesavant Order Tracking
 
 - osrs_savant_cancel_events
 - osrs_savant_fill_events
 - osrs_savant_order_events
 - osrs_savant_orders


## Using grafana for visualization

I have a public instance that I can share with anyone interested, but feel free to point a grafana clickhouse datasource at your instance and then use the `grafana-dashboard.json` file to import. The discord bot in its excerpt for linking graphs assumes that you have the same public IP that is running the discord bot and the grafana server

## Setting up Redpanda

To install redpanda follow this [guide](https://docs.redpanda.com/current/deploy/deployment-option/self-hosted/manual/production/production-deployment/)

I recommend using redpanda console to interact with you environment, the instruction are included above

message schemas are updated in `schemas/` (except for osrs-ref-data)

```
osrs-fills: Successful execution {price, symbol_id, account_username, buy/sell, position_open_time, position_close_time}

```

