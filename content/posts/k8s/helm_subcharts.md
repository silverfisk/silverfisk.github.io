


+++ 
date = 2022-12-02T09:36:00+02:00
title = "When to extend a helm chart and when to use a subchart"
slug = "" 
tags = ["k8s", "Helm", "Subcharts"]
categories = ["kubernetes"]
thumbnail = ""
description = "When to extend a helm chart and when to use a subchart"
+++


## Overview
----

A Helm chart is a package of Kubernetes resources that can be managed and deployed as a unit. It is defined using YAML files and follows a specific directory structure, with a Chart.yaml file that contains metadata about the chart, and one or more templates that define the resources to be deployed. Helm charts can be used to deploy applications, services, or infrastructure on Kubernetes clusters.

A Helm subchart is a reusable, modular component of a Helm chart that can be defined and managed independently. It is defined using the same directory structure and file formats as a regular chart, but it is included and managed as part of a parent chart. A subchart can have its own dependencies, templates, values, and release cycle, and can be shared and reused across different charts. Subcharts allow you to create more modular and maintainable Helm charts, and improve the organization and reliability of your Kubernetes deployments.


### The case for subcharts

A Helm subchart is better than extending a regular Helm chart in several ways:

1. Reusability and modularity: A subchart can be used in multiple charts and shared between different teams or projects. It can be updated and maintained independently, without affecting the rest of the chart. This makes the chart more maintainable, scalable, and easier to understand and modify.
1. Separation of concerns: A subchart allows you to separate the different concerns and responsibilities of the chart into distinct, manageable components. This can improve the design and organization of the chart, and make it easier to understand and work with.
1. Dependency management: A subchart can have its own dependencies and requirements, which can be managed independently from the parent chart. This allows you to manage the dependencies of the subchart more effectively, and avoid conflicts or version mismatches with the parent chart.
1. Version control and release management: A subchart can have its own version and release cycle, which can be managed independently from the parent chart. This allows you to manage the versions and releases of the subchart more effectively, and avoid disruptions or inconsistencies with the parent chart.
1. Testing and validation: A subchart can be tested and validated independently from the parent chart, using a different set of test cases and criteria. This allows you to ensure the quality and reliability of the subchart more effectively, and avoid introducing bugs or regressions into the parent chart.

 
### When is it better to just extend a chart?

Adding a Helm subchart can add some complexity to a chart however, as it introduces additional components and dependencies that need to be managed and maintained. Therefore, it is not always a good idea to use a subchart, and it may be more appropriate to use a regular chart in some cases:

1. Simple charts: If your chart is small and simple, with few components and dependencies, using a subchart may add unnecessary complexity and overhead. In this case, it may be better to use a regular chart that is easier to manage and maintain.
1. Single-team or single-project charts: If your chart is used only by a single team or project, and is not shared or reused by other teams or projects, using a subchart may not provide any benefits or advantages. In this case, it may be better to use a regular chart that is easier to understand and work with.
1. Infrequently updated charts: If the chart is not updated or modified frequently, and does not require a complex or flexible release cycle, using a subchart may not be necessary. In this case, it may be better to use a regular chart that is easier to deploy and manage.
1. Overall, using a Helm subchart can be a good idea when it provides benefits or advantages that outweigh the added complexity and overhead. However, in some cases, it may be more appropriate to use a regular chart instead.


## Summary
----

* A Helm chart is a package of Kubernetes resources that can be deployed as a unit on a Kubernetes cluster.
* A Helm subchart is a reusable, modular component of a Helm chart that can be defined and managed independently.
* Using a Helm subchart can provide benefits such as reusability, modularity, separation of concerns, dependency management, version control, and testing.
* However, using a Helm subchart can also add complexity and overhead, and may not be appropriate in some cases such as simple, single-team, infrequently updated, or untested charts.
* Overall, using a Helm subchart can be a good idea when it provides benefits that outweigh the added complexity and overhead.
