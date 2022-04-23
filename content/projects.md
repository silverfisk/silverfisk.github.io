+++
title = "Projects"
slug = "projects"
thumbnail = "images/tn.png"
description = "projects"
+++

## Prometheus monitored freezer

{{% portfolio image="/img/prom_freezer.png" alt="Prometheus Freezer" %}}

## Monitor a freezer with Prometheus / Grafana

As a preparation of increased food prices, freezed food will be a more and more important asset to monitor.
I use a cheap ESP8266 NodeMCU unit with a Dallas DS18B20 temperature sensor to monitor the freezer.

[It uses PlatformIO to become a WiFi enabled device](https://github.com/silverfisk/esp8266_prometheus_exporter_for_ds18b20_temperature_sensors) that is scraped by Prometheus and visualized in Grafana. Prometheus alarms can then monitor and page you if the freezer loos it's temperature.
{{% /portfolio %}}