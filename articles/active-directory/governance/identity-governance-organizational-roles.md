---
title: Govern access with an organizational role model
description: Microsoft Entra Identity Governance allows you to model organizational roles using access packages, so you can migrate your existing role definitions to entitlement management.
services: active-directory
documentationcenter: ''
author: markwahl-msft
manager: amycolannino
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 12/1/2022
ms.author: mwahl
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
---

# Govern access by migrating an organizational role model to Microsoft Entra Identity Governance

Role-based access control (RBAC) provides a framework for classifying users and IT resources. This framework allows you to make explicit their relationship and the access rights that are appropriate according to that classification. For example, by assigning to a user attributes that specify the users job title and project assignments, the user can be granted access to tools needed for the user's job and data that the user needs to contribute to a particular project. When the user assumes a different job and different project assignments, changing the attributes that specify the user's job title and projects automatically blocks access to the resources only required for the users previous position.

In Azure AD, you can use role models in several ways to manage access at scale through identity governance.

 * You can use access packages to represent organizational roles in your organization, such as "sales representative". An access package representing that organizational role would include all the access rights that a sales representative might typically need, across multiple resources.
 * Applications [can define their own roles](../develop/howto-add-app-roles-in-azure-ad-apps.md). For example, if you had a sales application, and that application included the app role "salesperson", you could then [include that role in an access package](entitlement-management-access-package-resources.md).
 * You can use roles for [delegating administrative access](entitlement-management-delegate.md).  If you have a catalog for all the access packages needed by sales, you could assign someone to be responsible for that catalog, by assigning them a catalog-specific role.

This article discusses how to model organizational roles, using entitlement management access packages, so you can migrate your role definitions to Azure AD to enforce access.

## Migrating an organizational role model

The following table illustrates how concepts in organizational role definitions you might be familiar with in other products correspond to capabilities in entitlement management.

| Concept in organizational role modeling | Representation in Entitlement Management |
| --- | --- |
| Delegated role management | [Delegate to catalog creators](entitlement-management-delegate-catalog.md) |
| Collection of permissions across one or more applications | [Create an access package with resource roles](entitlement-management-access-package-create.md) |
| Restrict duration of access a role provides | [Set an access package's policy lifecycle settings to have an expiration date](entitlement-management-access-package-lifecycle-policy.md) |
| Individual assignment to a role | [Create a direct assignment to an access package](entitlement-management-access-package-assignments.md#directly-assign-a-user) |
| Assignment of roles to users based on properties (such as their department) | [Establish automatic assignment to an access package](entitlement-management-access-package-auto-assignment-policy.md) |
| Users can request and be approved for a role | [Configure policy settings for who can request an access package](entitlement-management-access-package-request-policy.md) |
| Access recertification of role members | [Set recurring access review settings in an access package policy](entitlement-management-access-reviews-create.md) |
| Separation of duties between roles | [Define two or more access packages as incompatible](entitlement-management-access-package-incompatible.md)|

For example, an organization may have an existing organizational role model similar to the following table.

|Role Name|Permissions the role provides|Automatic assignment to the role|Request-based assignment to the role|Separation of duties checks|
|:--|-|-|-|-|
|*Salesperson*|Member of **Sales** Team|Yes|No|None|
|*Sales Solution Manager*|The permissions of *Salesperson*, and **Solution manager** app role in the Sales application|None|A salesperson can request, requires manager approval and quarterly review|Requestor cannot be a *Sales Account Manager*|
|*Sales Account Manager*|The permissions of *Salesperson*, and **Account manager** app role in the Sales application|None|A salesperson can request, requires manager approval and quarterly review|Request cannot be a *Sales Solution Manager*|
|*Sales Support*|Same permissions as a *Salesperson*|None|Any non-salesperson can request, requires manager approval and quarterly review|Requestor cannot be a *Salesperson*|

This could be represented in Entra Identity Governance as an access package catalog containing four access packages.

|Access package|Resource roles|Policies|Incompatible access packages|
|:--|--|--|--|
|*Salesperson*|Member of **Sales** Team|Auto-assignment||
|*Sales Solution Manager*|**Solution manager** app role in the Sales application|Request-based|*Sales Account Manager*|
|*Sales Account Manager*|**Account manager** app role in the Sales application|Request-based|*Sales Solution Manager*|
|*Sales Support*|Member of **Sales** Team|Request-based|*Salesperson*|

The next sections outline the process for migration, creating the Azure AD and Microsoft Entra Identity Governance artifacts to implement the equivalent access of an organizational role model.

### Connect apps whose permissions are referenced in the organizational roles to Azure AD

If your organizational roles are used to assign permissions that control access to non-Microsoft SaaS apps, on-premises apps or your own cloud apps, then you will need to connect your applications to Azure AD.

In order for an access package representing an organizational role to be able to refer to an application's roles as the permissions to include in the role, for an application that  has multiple roles and supports modern standards such as SCIM, you should [integrate the application with Azure AD](identity-governance-applications-integrate.md) and ensure that the application's roles are listed in the application manifest.

If the application only has a single role, then you should still [integrated the application with Azure AD](identity-governance-applications-integrate.md).  For applications that do not support SCIM, Azure AD can write users into an application's existing directory or SQL database, or add AD users into an AD group.

### Populate Azure AD schema used by apps and for user scoping rules in the organizational roles

If your role definitions include statements of the form "all users with these attribute values get assigned to the role automatically" or "users with these attribute values are allowed to request", then you will need to ensure those attributes are present in Azure AD.

You can [extend the Azure AD schema](../app-provisioning/user-provisioning-sync-attributes-for-mapping.md) and then populate those attributes either from on-premises AD, via Azure AD Connect, or from an HR system such as Workday or SuccessFactors.

### Create catalogs for delegation

If the ongoing maintenance of roles is delegated, then you can delegate the administration of access packages by [creating a catalog](entitlement-management-catalog-create.md ) for each part of the organization you will be delegating to.

If you have multiple catalogs to create, you can use a PowerShell script to [create each catalog](entitlement-management-catalog-create.md#create-a-catalog-with-powershell).

If you are not planning to delegate the administration of the access packages, then you can keep the access packages in a single catalog.

### Add resources to the catalogs

Now that you have the catalogs identified, then [add the applications, groups or sites](entitlement-management-catalog-create.md#add-resources-to-a-catalog) that will be included in the access packages representing the organization roles to the catalogs.

If you have many resources, you can use a PowerShell script to [add each resource to a catalog](entitlement-management-catalog-create.md#add-a-resource-to-a-catalog-with-powershell).

### Create access packages corresponding to organizational role definitions

Each organizational role definition can be represented with an [access package](entitlement-management-access-package-create.md) in that catalog.

You can use a PowerShell script to [create an access package in a catalog](entitlement-management-access-package-create.md#create-an-access-package-with-microsoft-powershell).

Once you've created an access package, then you'll link one or more of the roles of the resources in the catalog to the access package.  This represents the permissions of the organizational role.

In addition, you'll [create a policy for direct assignment](entitlement-management-access-package-request-policy.md#none-administrator-direct-assignments-only), as part of that access package that can be used to track the users who already have individual organizational role assignments.

### Create access package assignments for existing individual organizational role assignments

If some of your users already have organizational role memberships, that they would not receive via automatic assignment, then you should [create direct assignments](entitlement-management-access-package-assignments.md#directly-assign-a-user) for those users to the corresponding access packages.

If you have many users who need assignments, you can use a PowerShell script to [assign each user to an access package](entitlement-management-access-package-assignments.md#assign-a-user-to-an-access-package-with-powershell).  This would link the users to the direct assignment policy.

### Add policies to those access packages for auto assignment

If your organizational role definition includes a rule based on user's attributes to assign and remove access automatically based on those attributes, you can represent this using an [automatic assignment policy](entitlement-management-access-package-auto-assignment-policy.md). An access package can have at most one automatic assignment policy.

If you have many role definitions that each have a role definition, you can use a PowerShell script to [create each automatic assignment policy](entitlement-management-access-package-auto-assignment-policy.md#create-an-access-package-assignment-policy-through-powershell) in each access package.

### Set access packages as incompatible for separation of duties

If you have separation of duties constraints that prevent a user from taking on one organizational role when they already have another, then you can prevent the user from requesting access in entitlement management by [marking those access package combinations as incompatible](entitlement-management-access-package-incompatible.md).

For each access package that is to be marked as incompatible with another, you can use a PowerShell script to [configure access packages as incompatible](entitlement-management-access-package-incompatible.md#configure-incompatible-access-packages-through-microsoft-powershell).

### Add policies to access packages for users to be allowed to request

If users who do not already have an organizational role are allowed to request and be approved to take on a role, then you can also configure entitlement management to allow users to request an access package. You can [add additional policies to an access package](entitlement-management-access-package-request-policy.md#choose-between-one-or-multiple-policies), and in each policy specify which users can request and who must approve.

### Configure access reviews in access package assignment policies

If your organizational roles require regular review of their membership, you can [configure recurring access reviews](entitlement-management-access-reviews-create.md) in the request-based and direct assignment policies.

## Next steps

- [What is Microsoft Entra entitlement management?](entitlement-management-overview.md)
- [Define governance policies](identity-governance-applications-define.md)
- [Integrate an application with Azure AD](identity-governance-applications-integrate.md)
- [Deploy governance policies](identity-governance-applications-deploy.md)
