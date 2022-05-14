---
title: "Prometheus monitoring of BME280 sensor readings"
date: 2022-05-14T19:07:07+02:00
tags: "observability, prometheus, grafana"
categories: "Grafana"
description: "Prometheus monitoring of BME280 sensor readings"
---

I wanted to be able to monitor the environment in my RV, so this weekend I created the [esp32_prometheus_exporter_BME280](https://github.com/silverfisk/esp32_prometheus_exporter_BME280).

My plan later is to monitor it and send alerts with LoraWan so I know my cat is OK I want to leave it in the vehicle. With LoraWan I should be able to get notified without cellular coverage.

For now it's just a regular wifi based sensor since I don't have the lora hardware yet.

Having earlier experience of DHT11 sensors I can say they're not worth your time. This BME280 sensor is much better, and not very expensive. 

## Hardware

The hardware I used is no longer for sale, but I tried to find similar devices from other places. You should be able to make the same setup with this hardware.

| Device | Price |
|--| -- |
| [BMP280 sensor module](https://www.aliexpress.com/item/32784210368.html?spm=a2g0o.productlist.0.0.65802e97vsRnc2&algo_pvid=a7d4a0c5-bb64-41e4-93d0-a5443a1b649d&algo_exp_id=a7d4a0c5-bb64-41e4-93d0-a5443a1b649d-5&pdp_ext_f=%7B%22sku_id%22%3A%2267085122438%22%7D&pdp_npi=2%40dis%21SEK%21%2111.75%21%21%21%21%21%402100bddb16525579776323038e881a%2167085122438%21sea) | €1,10 |
| [ESP32 Devkit v1](https://www.aliexpress.com/item/32847027609.html?spm=a2g0o.store_pc_groupList.8148356.27.37cf63a5G1T0ex&pdp_npi=2%40dis%21EUR%21%E2%82%AC%204%2C75%21%E2%82%AC%204%2C52%21%21%21%21%21%402100bdcf16525584039303986ee56b%21%21sh) | € 4,53 |

These devices have unsoldered pins. I personally prefer this since I can make the footprint smaller when soldering the components together.