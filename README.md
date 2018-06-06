# Azure Policy Service Sample
Azure policy Service can be used to implement rules that helps organizations stay compliant when deploying and configuring resources in Azure. This sample implements a rule that ensures that no Compute resources in Azure, like Virtual Machines, are deployed without having mandatory tags included in the Provisioning request. The tag names used are 'Cost Center Name' to be able to attribute the charge to, and the 'Service Name' of the Application that would be hosted in the Virtual Machine.

Using an Azure Policy Service Policy ensures that these rules are honored irrespective of how the resource is provisioned, be it using ARM Template, Azure Portal,  CLI, or using the REST API.

The Json document (*VMTagsPolicy.json*) representing the Policy definition is available in this GitHub repository accompanying this article.
The steps to define a Policy and assign it are documented here https://docs.microsoft.com/en-us/azure/azure-policy/create-manage-policy#implement-a-new-custom-policy and here https://docs.microsoft.com/en-us/azure/azure-policy/assign-policy-definition.

![GitHub Logo](/images/PolicyDefinition.png = 250x)

In the Policy rule defined above, if either of the Tags are not specified in the request, the Provisioning request gets denied. The tag values are also validated to ensure that they are in the list of allowed Cost Center and Service Name Lists.

![GitHub Logo](/images/PolicyParams.png =100x250)

The tag values are parameterized, and allowed values for Cost Center and Service Names bound to a predetermined set of values in the JSON definition.
In this example, the Policy is assigned to a specific Resource group in the current Azure Subscription, so that the Policy gets applied only to this scope.

The Policy is now assigned with the allowed values for Cost Centers as "Cost Center 1" and "Cost Center 2". See screenshot below.

![GitHub Logo](/images/policy_assignment.PNG)

## Validating this Policy by provisioning a VM using different options

1) Using CLI

az vm create  --resource-group azpolicyrg --name azpolicyvm1 --image UbuntuLTS --admin-username onevmadmin --admin-password Pass@word123 --debug

![GitHub Logo](/images/vm_cli_debug1.PNG)

The request above fails since the tags were missing in the request. 

The request below fails since the values set for the tags did not conform to the allowed values specified in the Policy assignment defined in the previous steps
az vm create  --resource-group azpolicyrg --name azpolicyvm1 --image UbuntuLTS --admin-username onevmadmin --admin-password Pass@word123 --tags CostCenter="Cost Center 2" ServiceName="Service 1" --debug

The request below includes all the mandatory tags and the allowed values as set in the Policy definition, hence it succeeds and the VM gets provisioned.
az vm create  --resource-group azpolicyrg --name azpolicyvm1 --image UbuntuLTS --admin-username onevmadmin --admin-password Pass@word123 --tags CostCenter="Cost Center 2" ServiceName="Service 1" --debug

2) Using an ARM Template
The ARM template (*SimpleVmJson.json*) used here is uploaded to the GitHub Repository referred to in this article
Selecting a wrong value for the Cost Center Code ('Cost Center 3' selected in the ARM Template is not from among the list of allowed values in the Policy Assignment created in the previous steps), fails the resource provisioning request. See screenshot below

![GitHub Logo](/images/ArmTemplate1.png)

3) Using the Azure portal to create a VM will not succeed, since the wizard does not provide an option to specify tags. However, when  a user edits the tags in a VM that already exists, the Policy validation kicks in and ensures that any changes that violate the policy are disallowed.
In the screen shot below, deleting the 'Cost' Center' tag and selecting 'save' errors out citing the Policy violation

![GitHub Logo](/images/PortalEditTags.PNG)

While in the Policy definition the rule action is set to 'Deny' when the validation fails, and the VM provisioning fails, setting the rule action to 'audit' could be used instead to ensure that the provisioning requests succeeds, but the violations are written to audit and surfaces in the compliance dashboard. An organization could take corrective action manually, and at their convenience.

Scenario 2:
Azure Storage now provides the option to associate a Vnet Service endpoint to it, that ensures that only services deployed in the Subnet would have access to the Storage account. This helps to lock down access to it from Services that originate from the internet.

The Policy definition below implements this rule, whereby only requests to provision a Storage account that have a VNET Service endpoint configured would be permitted, else the action is set to 'deny' the request. See screenshot below for the Policy Definition. The Policy definition file, *StorageSecurityCompliance.json* is available in the GitHub Location accompanying this article

![GitHub Logo](/images/StorageSecurity.PNG)



