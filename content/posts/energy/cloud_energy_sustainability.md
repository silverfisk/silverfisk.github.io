---
title: "Energy, prosperity and mindset of sustainability in the cloud"
date: 2022-12-02T13:50:07+02:00
tags: ["sustainability", "Cloud", "karpenter"]
categories: ["Energy"]
description: "Energy, prosperity and mindset opf sustainability in the cloud"
---

Energy is a key factor for human prosperity. It is essential for providing the power needed for transportation, heating and cooling, lighting, and other daily activities. It also plays a crucial role in supporting economic growth and development, as it is needed to run factories, power plants, other infrastructure, and services.

There is a wealth of data that supports the importance of energy for human prosperity. For example, the World Bank reports that access to modern energy services is a key factor in reducing poverty and promoting economic growth. The International Energy Agency (IEA) also finds that improving access to modern energy can have a major impact on health, education, and other indicators of human development.

Additionally, studies have shown that increasing access to modern energy can have a positive impact on a variety of economic and social indicators, including gross domestic product (GDP), employment, and life expectancy.

Overall, the data clearly shows that energy is essential for human prosperity, and that access to modern, reliable energy is a key factor in promoting economic growth and development.

So we cannot remove energy, because that would also remove the capacity for human prosperity. But we all need to understand that this energy is precious. Here I will describe how this mindset can be used to improve delivery of cloud based work loads.


## Regional emmision factors

Regional emission factors are used to calculate the carbon emissions associated with the generation of electricity in a specific region. These factors take into account the mix of energy sources used in the region, such as coal, natural gas, renewable energy, etc. By using regional emission factors, you can more accurately estimate the carbon emissions of your electricity-consuming tasks, and choose the region with the lowest emissions for your tasks.

For example, the website [electricitymaps.com](https://electricitymaps.com) provides a map that shows the carbon intensity of electricity generation in different regions around the world. You can use this map to compare the emissions of different regions and choose the one with the lowest emissions for your tasks. 

In addition to choosing the most efficient region, you can also use regional emission factors to compare the emissions of different energy sources within a region. For example, you may find that using renewable energy sources in a region has lower emissions than using fossil fuels. This information can help you make more sustainable choices for your electricity-consuming tasks.

In our case our constraint is an AWS bound kubernetes cluster in Europe, so the regional emmision factors map should be taken into consideration when directing your workload to a suitable [AWS Region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/).

## Infrastructure as code (IaC) is a key sustainability factor

Infrastructure as code (IaC) automation refers to the practice of using code to manage and configure the infrastructure for a software system, rather than manually configuring the infrastructure using a graphical user interface (GUI) or other tools. IaC allows you to define your infrastructure in a version-controlled text file, which can be automatically reconsiliated into your infrastructure.

One benefit of using IaC automation is that it makes it easier to change the region of your infrastructure. When you use IaC, you can define the region of your infrastructure by changing a parameter, and then use the code to create or update your infrastructure in another AWS region. This means that you can change the region simply by modifying the code, and then run the code to update your infrastructure.

This is much easier, less error prone and provides better quality than manually changing your infrastructure using the AWS GUI or other tools. With IaC, you can make the change quickly and easily, and you can also track and audit the change using the version control system. Additionally, using IaC automation can help ensure that your infrastructure is consistent and well-defined, which can improve reliability and reduce the risk of errors.

## Rightscaling

Rightsizing can be challenging in the cloud, as workloads and usage patterns can change dynamically and unpredictably. However, using tools and techniques such as [karpenter](https://karpenter.sh/), it is possible to continuously monitor and adjust the cloud environment to ensure optimal performance, environmental footprint and cost.

Karpenter is a tool for managing and optimizing Kubernetes clusters on Amazon Web Services (AWS). It allows you to easily provision and manage your clusters, and to optimize their performance and cost.

One of the key features of Karpenter is its ability to help you rightsize your clusters and use spot instances. This can help you avoid overprovisioning or underprovisioning, which can lead to wasted resources or poor performance.

Karpenter can also help you use spot instances to save cost on AWS. Spot instances are spare compute capacity that AWS offers at a discounted price, but which can be terminated by AWS if the capacity is needed elsewhere. Karpenter can help you use spot instances in your clusters by automatically replacing terminated instances with new ones, and by providing tools for managing the availability and performance of your spot instances.


----
**Recognition**

This article was inspired by [GaiaGen](https://gaiagen.eu/recommendations).
