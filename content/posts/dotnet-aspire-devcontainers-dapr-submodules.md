---
title: "Modern .NET Development with Aspire, Dapr, TestContainers and DevContainers"
date: 2025-08-05
draft: true
---

## The Problem
When creating distributed applications one of the common issues is the complicated setup for local development. Often I've seen new developers spending 2-3 days getting their local environment running, wrestling with outdated documentation and version conflicts.

An example of what such an environment could contain:
- **Database**: Azure SQL Server, PostgreSQL, MongoDB, MySQL
- **Cache**: Redis
- **Message Broker**: RabbitMQ, Azure Service Bus
- **Backend APIs**: multiple ASP.NET Core services
- **Frontend**: multiple SPAs (Single Page Applications)

## The Solution

After working with .NET Aspire for nearly a year in our current team, we've transformed this experience. New developer onboarding now takes under an hour. Aspire orchestrates our entire distributed environment locally, while Dapr abstracts away the complexity of cloud-native patterns.

In this post, I'll share the tools that made this possible and provide a complete starter template for building production-ready .NET applications using Aspire, Dapr, and DevContainers.

> **Note:** Just want to get the starter template? Scroll to the bottom.
---

## The Tools

### .NET Aspire
.NET Aspire allows us to define resources and orchestrate them seamlessly in what they call **Run mode**, which is designed for local development and testing scenarios.

In production environments, we can deploy containerized applications in many different ways. .NET Aspire allows you to get quickly up and running in Azure Container Apps. Azure Container Apps is a serverless platform that allows you to maintain less infrastructure and save costs while running containerized applications. Azure Container Apps is a PaaS that has Kubernetes under the hood and provides built-in support for KEDA (Kubernetes Event-Driven Autoscaling) and Dapr (Distributed Application Runtime) out of the box.

### Dapr
Dapr makes it easier to build resilient, stateless and stateful microservice applications. Dapr uses the **sidecar architecture pattern**, where each of our containerized instances runs alongside a Dapr sidecar process that handles the complexities of distributed computing. Dapr has a number of building blocks, of which I think the following are the most important:

- **Service Invocation**: Enables secure, reliable service-to-service communication with automatic service discovery, load balancing, and retry policies
- **Pub/Sub Messaging**: Provides publish and subscribe capabilities with support for multiple message brokers, ensuring loose coupling between services
- **State Management**: Offers a consistent API for managing application state across different state stores, with features like transactions and encryption

By abstracting infrastructure resources behind consistent APIs, Dapr enables switching between local and cloud implementations without code changes. We can use RabbitMQ locally and Azure Service Bus in production.

### DevContainers
DevContainers provide a consistent, reproducible development environment by packaging your entire development setup into a container.

---

## References
- [.NET Aspire Documentation](https://docs.microsoft.com/en-us/dotnet/aspire/)
- [.NET Aspire Architecture Overview](https://docs.microsoft.com/en-us/dotnet/aspire/fundamentals/app-host-overview)
- [Azure Container Apps Overview](https://docs.microsoft.com/en-us/azure/container-apps/overview)
- [Dapr Documentation](https://docs.dapr.io/)
- [Dapr Sidecar Architecture](https://docs.dapr.io/concepts/dapr-services/sidecar/)
- [DevContainers Documentation](https://containers.dev/)


