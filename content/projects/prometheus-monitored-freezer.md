+++
title = "Monitor your freezer with Prometheus and Grafana"
slug = "projects"
description = "projects"
date = "2022-04-20"
+++

## Prometheus monitored freezer

{{% portfolio image="/img/esp01-usb-programmer.png" alt="Prometheus Freezer" %}}

## Monitor a freezer with Prometheus and Grafana

As a preparation of increased food prices, freezed food will be a more and more important asset to monitor.
I use a cheap ESP8266 NodeMCU unit with a Dallas DS18B20 temperature sensor to monitor the freezer.

[It uses PlatformIO to become a WiFi enabled device](https://github.com/silverfisk/esp8266_prometheus_exporter_for_ds18b20_temperature_sensors) that is scraped by Prometheus and visualized in Grafana. Prometheus alarms can then monitor and page you if the freezer loose it's temperature.
{{% /portfolio %}}


