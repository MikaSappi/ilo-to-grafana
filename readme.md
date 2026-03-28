# iLO server visibility in Grafana

This repository contains the JSON configuration file for Grafana Cloud, as well as the InfluxDB configuration files you run on any server that can connect to iLO's IP.

Telegraf has some dependencies, please look at `sudo journalctl -f -u telegraf.service`. It will tell.

## `output.conf`
This file contains the information you need to push data to Grafana Cloud.

### Get credentials:
`https://grafana.com/` -> `My account` (top right) -> On your stack, choose `Details` -> `InfluxDB Connectivity` -> `Configure` -> copy your URL and username and create an API key and keep it safe.

`urls`: your Grafana Cloud stack's url, suffixed by `/api/v1/push/influx`. For example, if your stack is located in Stockholm, you would use `https://prometheus-prod-39-prod-eu-north-0.grafana.net/api/v1/push/influx`.

`skip_database_creation`: `true`, not relevant for this use case.

`username`: your Grafana username, but because it's Grafana, it's not really your username, but an arbitrary number string you got when you navigated to the path shown before urls.

`password`: again, it's Grafana. It's not a password at all, but an API key you got in the first step.

## `ilo.conf`
You shouldn't care about anything else than what's in the first block.

iLO's SNMP settings are found in `Management` -> `SNMP Settings`.

### iLO
Create an SNMPv3 user. Remember what you write in there, since you need the data when setting up `ilo.conf`. Set Authentication Protocol to `SHA`.

`agents`: set your **iLO** IP, suffixed by the port `161`.

`interval`: how often to scrape.

`sec_name`: whatever you set for Security Name in iLO.

`auth_protocol`: SHA.

`auth_password`: whatever you set in iLO as Authentication Passphrase.

`priv_password`: whatever you set in iLO as Privacy Passphrase.

Example:

```
[[inputs.snmp]]
  name_override = "ilo_esxi_snmp"
  agents = [
    "192.168.100.9:161"
  ]
  interval = "15s"
  version = 3
  community = "public"
  sec_name = "ilo_flux"
  auth_protocol = "SHA"
  auth_password = "developer"
  sec_level = "authPriv"
  context_name = ""
  priv_protocol = "AES"
  priv_password = "developer"
```

## Grafana Cloud
Then navigate to your stack url. For example, if your stack name was `kitten`, you'd go to `kitten.grafana.net`.

### Add data source
Navigate to `Connections` -> `Data sources` -> `Add new data source`, choose `Prometheus`.

VERY IMPORTANT: give the data source a proper name so you don't have to go through every `Prometheus-n` datasource when you run into trouble in the `UID` step.

#### URL
For `Prometheus server URL`, set the URL you're pushing data to, and by the help of a crystal ball or the nearest oracle, know that the URL you're pulling from just happens to be `https://prometheus-prod-39-prod-eu-north-0.grafana.net/api/prom` (if you're pushing to `eu-north` for example).

Then, because this is Grafana, you need to manually enable `metrics:read`. Go back to `grafana.com`, under `My account`, find `Access policies`, find your influxDB policy, press edit and add `metrics:read`.

#### Authentication
Select `Basic Authentication`, `Username` is your arbitrary number string, not your actual username. It's Grafana after all! `Password` would be your API key out of unknown reasons.

#### Save & Test
This should tell you that everything is fine, but usually it doesn't. If it says `401`, you need to check the access policies from the URL section. If it's `404`, you probably have the wrong URL, or don't have the correct path. Remember, it's not the same path you push into because it's Grafana.

#### UID
After it's saved, open the data source again, and from the browser URL bar copy the UID, such as `sdfgfsth343ka`.

### Import the dashboard
Under `Dashboard`, select `New` and `Import`. There, upload the JSON in the upload section. Give the dashboard a nice name, such as Jason.

You won't see data yet. Open the dashboard if not open yet, and in the top panel enter the `UID` you copied recently. For `iLO Host IP` set the iLO LOCAL IP.

## Have fun
PS: I don't like Grafana.
