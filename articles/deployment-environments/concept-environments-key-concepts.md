---
title: Azure Deployment Environments key concepts
description: Learn the key concepts behind Azure Deployment Environments.
ms.service: deployment-environments
ms.custom: ignite-2022
ms.topic: conceptual
ms.author: rosemalcolm
author: RoseHJM
ms.date: 10/12/2022
---

# Key concepts for new Azure Deployment Environments Preview users

Learn about the key concepts and components of Azure Deployment Environments Preview. This knowledge can help you more effectively deploy environments for your scenarios.

> [!IMPORTANT]
> Azure Deployment Environments is currently in preview. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Dev centers

A dev center is a collection of projects that require similar settings. Dev centers enable development infrastructure (dev infra) managers to:

- Use catalogs to manage infrastructure as code (IaC) templates that are available to the projects.
- Use environment types to configure the types of environments that development teams can create.

## Projects

A project is the point of access for the development team. When you associate a project with a dev center, all the settings for the dev center are automatically applied to the project. 

Each project can be associated with only one dev center. Dev infra admins can configure environments for a project by specifying which environment types are appropriate for the development team.

## Environments

An environment is a collection of Azure resources on which your application is deployed. For example, to deploy a web application, you might create an environment that consists of [Azure App Service](../app-service/overview.md), [Azure Key Vault](../key-vault/general/basic-concepts.md), [Azure Cosmos DB](../cosmos-db/introduction.md), and a [storage account](../storage/common/storage-account-overview.md). An environment could consist of both Azure platform as a service (PaaS) and infrastructure as a service (IaaS) resources such as an Azure Kubernetes Service (AKS) cluster, virtual machines, and databases.

## Identities

in Azure Deployment Environments, you use [managed identities](../active-directory/managed-identities-azure-resources/overview.md) to provide elevation-of-privilege capabilities. Identities can help you provide self-serve capabilities to your development teams without giving them access to the target subscriptions in which the Azure resources are created. 

The managed identity that's attached to the dev center needs to be granted appropriate access to connect to the catalogs. You should grant owner access to the target deployment subscriptions that are configured at the project level. The Azure Deployment Environments service will use the specific managed identity to perform the deployment on behalf of the developer.

## Dev center environment types

You can define the types of environments that development teams can create: for example, dev, test, sandbox, pre-production, or production. Azure Deployment Environments provides the flexibility to name the environment types according to the nomenclature that your enterprise uses. You can configure settings for various environment types based on the specific needs of the development teams.

## Project environment types 

Project environment types are a subset of the environment types that you configure for the dev center. They help you preconfigure the types of environments that specific development teams can create. You can configure the target subscription in which Azure resources are created per project and per environment type. 

Project environment types allow you to automatically apply the right set of policies on environments and help abstract the Azure governance-related concepts from your development teams. The service also provides the flexibility to preconfigure:

- The [managed identity](concept-environments-key-concepts.md#identities) that will be used to perform the deployment.
- The access levels that the development teams will get after a specific environment is created.

## Catalogs

Catalogs help you provide a set of curated IaC templates for your development teams to create environments. You can attach either a [GitHub repository](https://docs.github.com/repositories/creating-and-managing-repositories/about-repositories) or an [Azure DevOps Services repository](/azure/devops/repos/get-started/what-is-repos) as a catalog. 

Deployment environments scan the specified folder of the repository to find [catalog items](#catalog-items). The environments then make those catalog items available to all the projects associated with the dev center.

## Catalog items

A catalog item is a combination of an IaC template and a manifest file. The template defines the environment, and the manifest provides metadata about the template. Your development teams will use the items that you provide in the catalog to create environments in Azure.

> [!NOTE]
> During public preview, Azure Deployment Environments uses Azure Resource Manager (ARM) templates.

## ARM templates

[ARM templates](../azure-resource-manager/templates/overview.md) help you implement the IaC for your Azure solutions by defining the infrastructure and configuration for your project, the resources to deploy, and the properties of those resources.

To learn about the structure of an ARM template, the sections of a template, and the properties that are available in those sections, see [Understand the structure and syntax of Azure Resource Manager templates](../azure-resource-manager/templates/syntax.md).

## Next steps

[Quickstart: Create and configure a dev center](./quickstart-create-and-configure-devcenter.md)
