# Azure Sandbox creation step by step guide

A sandbox is an isolated environment where you can test and experiment without affecting other environments, like production, development, or user acceptance testing (UAT) environments. Conduct proof of concepts (POCs) with Azure resources in a controlled environment. Each sandbox has its own Azure subscription, and Azure policies control the subscription. The policies are applied at the sandbox management group level, and the management group inherits policies from the hierarchy above it. Depending on its purpose, an individual or a team can use a sandbox.  This guide assumes you are creating the subscriptions under an Enterprise Agreement.  If you use a CSP (now called MCA) or Pay as you go, the process will be slightly different and will require you to ask your MCA provider to create the subscriptions for you.

![Azure Sandbox Architecture](/sandpit/images/single-use-case-sandbox.png)

Each of the steps outlined manually below can be automated, including the process for requesting and decommissioning sandbox environments.  To improve the process it is recommended to focus on one of these steps at a time, and then automate the process.  This will allow you to build up the automation over time, and improve your capability to scale and reliably repeat the process to provision and decommission the sandbox.

## Things to know before you start

Before you begin this process you will need to have the following information:

### Subscription Name
Most organisations will have a naming convention for subscriptions.  For sandbox environments you may want to use *sbx-projectname* or *sbx-username* as the naming convention.  This will allow you to easily identify the sandbox subscriptions in the Azure Portal. [Azure Best Practices - Resource Naming](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming#example-names-general)

### Cost Center
Who is going to be funding this sandbox?  Is it a specific project, or is it a centralised cost?  This will determine who the subscription owner is, and who will be responsible for the cost of the sandbox.

### Subscription Owner
Who is going to be responsible for the subscription?  This person will be responsible for the cost of the subscription, and will be the owner of the subscription.  They will be able to create resources, assign roles and manage the subscription.  This person should be a member of the project team, and should be the person who requested the sandbox.

### Billing Account and Enrollment Account
For most organisations you will only have one of these so it will be easy to determine which one to use.  If you have multiple billing accounts, you will need to determine which billing account the sandbox will be billed to.  This will determine which enrollment account you will need to use.  Enrollment accounts may relate to different tenants so ensure you are selecting the correct one.

### Time-box \ Expiration Date
How long will the sandbox be required for?  This will determine the expiration date of the sandbox.  It is recommended to set an expiration date on the sandbox to ensure that it is decommissioned when it is no longer required.  This will ensure that the sandbox is not left running and incurring costs when it is no longer required.

### Roles and Rights
Ensure the person performing the steps below has the correct roles assigned, or available to them.  The following roles are required to perform the steps below:

- [Account Owner on EA to create subscriptions](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/create-enterprise-subscription#permission-required-to-create-azure-subscriptions)
- [Owner on Management Group to assign policy](https://docs.microsoft.com/en-us/azure/governance/management-groups/overview#management-group-access)

## Create a sandbox management group
To create a management group structure based on the CAF, follow these steps:

1. In the Azure Portal in the search bar type Management and click on Management Groups
2. Select the location where you want to create the management group.  This should be outside the main management group tree as per the example in the [Cloud Adoption Framework](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/considerations/sandbox-environments)
3. Click Create
4. In the Management Group ID field type *mg-sandboxes* and in the Management group display name field type *Sandboxes*
5. Click Submit

![Management Group Creation](/sandpit/images/managementgroup.png)

Note: The Cloud Adoption Framework (CAF) provides a set of best practices and guidance for adopting Azure. It helps organizations align their cloud adoption efforts with business goals and establish a solid foundation for successful cloud adoption.

For more information on the CAF and creating a management group structure, refer to the official Microsoft documentation [aka.ms/azurecaf](https://aka.ms/azurecaf)

## Create policy guardrails for the sandbox

Once the management group is created, policies need to be applied to ensure that specific resources and configurations are enforced.  This is accomplished by creating an [Azure Policy Initiative](https://learn.microsoft.com/en-us/azure/governance/policy/concepts/initiative-definition-structure), adding in policies required to enforce the guardrails on your sandbox, and then assigning the initiative to the management group.  By enforcing these rules at the management group level, all subscriptions created under the management group will inherit the policies.  Users who have access to the subscription can then create resources within the guardrails but will have no ability to change the policies, regardless of their role.

For this sample we will create an initiative with 3 specific policies:

- [Deny ExpressRoute and Virtual WAN](https://www.azadvertizer.net/azpolicyadvertizer/6c112d4e-5bc7-47ae-a041-ea2d9dccd749.html)
- [Deny Peering](https://www.azadvertizer.net/azpolicyadvertizer/Deny-VNET-Peer-Cross-Sub.html)
- [Configure Azure activity logs](https://www.azadvertizer.net/azpolicyadvertizer/Deny-VNET-Peer-Cross-Sub.html)

These policies can be added to over time as more guardrails are required.

### Create a Deny Express Route and VWan policy

1. In the Azure Portal in the search bar type Policy and click on Policy
2. Click on Definitions
3. Click on + Policy Definition
4. In the Definition Location click on the ellipsis and click on the Sandboxes management group then click Select
5. In the Policy definition name field type *Deny ExpressRoute and Virtual WAN*
6. In the Description field type *This policy is used to enforce the prevention of creation of specific resources by policy as a guardrail for Sandbox environments.*
7. In the Category field type *Sandbox Guardrail*
8. Open the deny_er_vwan.json file in this repo, and copy the contents into the Policy rule field
9. Click Save

![Policy Creation](/sandpit/images/policy.png)

### Create a Deny Peering

1. Click on + Policy Definition
2. In the Definition Location click on the ellipsis and click on the Sandboxes management group then click Select
3. In the Policy definition name field type *Deny Vnet Peering*
4. In the Description field type *This policy is used to enforce the prevention of creation of specific resources by policy as a guardrail for Sandbox environments.*
5. In the Category field click Use existing and select *Sandbox Guardrail*
6. Open the deny_vnet_peering.json file in this repo, and copy the contents into the Policy rule field
7. Click Save

### Create the Policy Initiative

1. Click on + Initiative Definition
2. In the Initiative Location click on the ellipsis and click on the Sandboxes management group then click Select
3. In the Name field type *Sandbox Guardrail Initiative*
4. In the Description field type *This policy initiative is used to enforce the prevention of creation of specific resources by policy as a guardrail for Sandbox environments.*
5. In the Category field click Use existing and select *Sandbox Guardrail*
6. Click Next
7. Click Add Policy Definition(s)
8. In the search bar type *Deny*.  Check the box next to both Deny expressroute and Deny Vnet Peering and click Add
7. Click Add Policy Definition(s)
8. In the search bar type *Configure Azure activity logs*.  Check the box next to Configure Azure activity logs and click Add
9. Click Next till you get to the Policy Parameters screen
10. Click on Value(s) dropdown for the Deny ExpressRoute and Virtual WAN policy
11. Type *wan* and then check the box next to virtualWans
12. Replace *wan* with *express* and check the boxes next to expressRouteCircuits, expressRouteGateways and expressRoutePorts
13. The Values field should now have 4 selected
14. On the line for the Configure Azure Activity Logs click the ellipsis at the far right end
15. In the Subscription field select the subscription you want to send the logs to and in the workspaces field select the workspace, select your core logging workspace.
16. Click Select
17. Click Next
18. Click Create

### Assign the Policy Initiative to the Management Group

1. Click Assign Initiative
2. In the Management Group field select the Sandboxes management group
3. Ensure Policy Enforcement is set to Enabled
4. Click Review and Create
5. Click Create

## Create subscription

To ensure that the right configuration, roles and tags are applied to the subscription, it is recommended to create the subscription using the same process every time.  This is a great process to start with when m oving to scripting using ARM templates and automation as it makes it easier to repeatably and reliably create the subscription every time.  For this example we will use the portal.

### Add a subscription

1. In the Azure Portal in the search bar type Subscriptions and click on Subscriptions
2. Click on + Add
3. In the Subscription Name field type the name preselected for this subscription - for example *sbx-<your initials>*
4. In the Billing Account and Enrollment Account fields select account this subscription will be billed to
5. In the Offer Type field select Enterprise Dev\Test
6. Click Next
7. In the Subscription Directory click the default directory for your tenant.  This is where the identities live that will be used to access the subscription.
8. In the Management Group field select the Sandboxes management group
9. In the Subscription Owner field type the name of the person who will be responsible for the subscription i.e. billtest999621@contoso.com
10. Click Next
11. In the Tags screen, click the first Name field and type CostCenter.  In the Value field type the cost center this subscription will be billed to.  For example *123456*
12. Click the next Name field and type ExpirationDate.  In the Value field type the date the subscription will expire.  For example *31/12/2021*
13. Click Next
14. Click Review and Create

## Budget Alerts

To ensure that the sandbox does not incur unexpected costs, a budget alert should be created.  This will send an email to the specified email address when the budget is reached.  The budget can be set to a specific amount or to a predicted amount based on current trend.  You can choose to set the budget when you create the subscription or after the subscription is created.  Predicting the budget can be difficult as it can be hard to determine what the costs will be based on services.  Its best to set a budget that your business feels comfortable spending on a prototype or proof of concept for a month, then managing that as the POC progresses.  For this example, we will be using $1000 as the estimated monthly cost.

### Create a Budget Alert

1. In the Azure Portal in the search bar type Budgets and click on Budgets
2. Ensure Scope: is set to the newly created subscription.  If it is not, click change, click through the tree of management groups until you get to the subscription under the Sandboxes management group and click Select
3. Click on Budgets in the left hand menu
4. Click on + Add
5. In the Name field type *SandboxBudget*
6. In the Amount field type *1000*
7. Click Next
8. In the Type field select Actual
9. In % of budget type 75
10. In the next row in the Type field select Forecasted
11. In % of budget type 100
12. In the Type field select Actual
13. In % of budget type 100
14. In the Alert recipients field type the email address you want the alert to go to.  Its recommended you add both the Sandpit requestor and someone in operations responsible for cost management to this list.
15. Click Create

![Budget Creation](/sandpit/images/budget-creation.png)

## Check user access

The creation process is complete.  Ensure the user who requested the subscription can access the subscription.  If they cannot, ensure they are added to the subscription as a user or owner.

## Monitor Expiration

This can be done manually by running the following KQL query in Azure Resource Graph Explorer:

1. In the Azure portal in the search bar type Resource Graph Explorer and click on Resource Graph Explorer
2. Copy the following code and paste it in the Query window

```kusto
resourcecontainers
| where type == "microsoft.resources/subscriptions"
| project SubscriptionId = id, SubscriptionName = name, SubscriptionTags = tags
| where SubscriptionTags.ExpirationDate != ""
| extend ExpirationDateStr = tostring(SubscriptionTags.ExpirationDate)
| project SubscriptionId, SubscriptionName, ExpirationDateStr
```	

3. Click Run query

This will list all subscriptions with an expiration date.  This could be added to an Azure Dashboard or extended using a function to check each day and email you when a subscription is about to expire.

## Subscription Cleanup

Once the expiration date is reached, the subscription should be decommissioned.  Remember to remove any data from the subscription that you want to keep before decommissioning the subscription as once it is gone, it is gone for good.

The cancellation can be done manually by following the steps below:

1. In the Azure Portal in the search bar type Subscriptions and click on Subscriptions
2. Click on the subscription you want to decommission
3. Click on Cancel Subscription
4. Select a reason for cancellation and tick the box Turn off Resources
5. Click on Cancel Subscription

![Subscription Cancellation](/sandpit/images/cancel-subscription-final.png)

This will set the subscription state to cancelled and all billing will cease. The subscription will be deleted automatically after 90 days, but if you wish to hasten that process you can [delete it manually 15 minutes after cancelling it.](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/cancel-azure-subscription#delete-subscriptions)

It is recommended to have a separate management group called Decommissioned and any subscriptions no longer required should be moved to this management group.  You can then apply a set of policies to restrict access to it, but to retain it for the 90 days just in case you need to recover it.  This ensures you retain the subscription without being charged.

## Automation

The following articles give steps on how to automate the manual processes above:

Creation of subscriptions - https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement?tabs=rest
Approval process using teams - https://learn.microsoft.com/en-us/power-automate/teams/native-approvals-in-teams
Dashboarding for operations teams - https://learn.microsoft.com/en-us/azure/azure-portal/azure-portal-dashboards
Purview for data security - https://learn.microsoft.com/en-us/purview/concept-best-practices-accounts

## Anti-patterns

There are areas of sandbox management and provisioning that are provide a trap for the unwary, especially as teams transition from heritage on premises style environments to public cloud.  The following are some of the common anti-patterns that should be avoided when creating sandbox environments.

### Responsibility
Ensure everyone in the process responsible for data, security or decommissioning is aware of their responsibilities and has bandwidth and capability to deliver them.  For users of sandbox environments, they are responsible for Cost and Data – they must be aware of their consumption and any data used must not be sensitive or classified data.  If the use case requires sensitive data, a more secure or controlled environment should be used.  For operations staff, they are responsible for monitoring cost and expiration dates on sandbox environments, and take action when appropriate.  They must have the dashboards, scripts and other tools alongside the technical skills to perform this task.

### Platform at once
Start the process of enabling sandbox environments by using manual process if automation capability does not exist today.  Make part of running and operating the environment a chance to start creating the automation of parts of the process.  This enables the operations team to get hands on experience with the tools and environments they are supporting, and drives process improvement over time without impeding innovation.  Start with an MVP that achieves the goal of enabling innovation and break each area into sections that can be improved on over time.  Automation capability patterns built within this process can easily be leveraged elsewhere in the business to drive process improvement at scale.

### Manage Service Deny Lists
In Azure Policy Service block lists should reflect a risk of massive cost or security, but there will always be times when they are required for proving a concept or testing a design pattern.  It is easier to manage if these block lists are kept to a minimum and there is the ability to lift them on a case by case basis.  This can be achieved by grouping the block lists into small groups in policy so individual services can be enabled without enabling the entire block list.  For example Network, Data, Identity and so on.

### Regional Lock-in
Many new services and capabilities do not get deployed to all regions initially, so policies enforcing specific regional deployments will stifle innovation.  Its useful to understand the reason for regional restriction policies and many of these stem from data residency risk amelioration.  For sandbox environments, as the data they consume is not sensitive data, these rules do not apply.

### Approval and Change
Architecture Review, Change Control and Release processes may be blockers to a streamlined development process so must evolve to meet the demands of an DevOps style project.  Ensure you identify early what the process for sign off and deployment are, include those teams in the process and use this as an opportunity to evolve internal process to a more modern framework.  This may slow down delivery however so balance the needs of process improvement against your time to value for this project.