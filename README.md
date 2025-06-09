# Azure Basic to Standard LB Upgrade

## Overview
This is the process for upgrading a Basic Public IP Address and Basic Load Balancer to Standard SKU versions. Output has been included to show a successful migration process. The process will use the AzureBasicLoadBalancerUpgrade PowerShell module.

## Before getting started
Make sure to review [Upgrade from Basic to Standard with PowerShell](https://learn.microsoft.com/en-us/azure/load-balancer/upgrade-basic-standard-with-powershell) document for a list of Common Questions, Unsupported Scenarios, Installation, Pre and Post-Migration steps, and module usage. The information below is from this document. 
 
>[!Important]
> - **Disclaimer:** This is for demonstration purposes only. I am not responsible for any issues you may run into during this process.
> - Again, review Microsoft's public process document before completing this process. [Upgrade from Basic to Standard with PowerShell](https://learn.microsoft.com/en-us/azure/load-balancer/upgrade-basic-standard-with-powershell)
> - Open an [Azure Support Ticket](https://learn.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request) if have any questions or if you're experiencing any issues.

>[!NOTE]
> - The backend is being migrated from a Basic Load Balancer to a Standard Load Balancer during the upgrade process.
> - This process ***WILL*** cause downtime. Plan accordingly.


## Module Installation
```PowerShell
Install-Module -Name AzureBasicLoadBalancerUpgrade -Scope CurrentUser -Repository PSGallery -Force
```

## Use the Module
Ensure you're using the expected Azure Sub ID
```PowerShell
Get-AzSubscription

   TenantId: 11111111-1111-1111-1111-111111111111

Name                                  Id                                   State
----                                  --                                   -----
MySubscriptionName                    1111111-1111-1111-1111-111111111111 Enabled
MySubscriptionName2                   1111111-1111-1111-1111-111111111112 Enabled
```

Set the correct Subscription
```PowerShell
Select-AzSubscription -Subscription 1111111-1111-1111-1111-111111111112
```

## Example Resource Ouput
Two Basic SKU resources will be converted in this example. A dynamic public ip named basic-lb-public-ip-test and a load balancer named basic-lb-test2.
```PowerShell
get-AzPublicIPAddress -Name basic-lb-public-ip-test -ResourceGroupName Lab

ResourceGroupName Name                    Location  PublicIpAllocationMethod IpAddress     PublicIpAddressVersion IdleTimeoutInMinutes ProvisioningState Sku
----------------- ----                    --------  ------------------------ ---------     ---------------------- -------------------- ----------------- ---
Lab               basic-lb-public-ip-test centralus Dynamic                  1.1.1.1 IPv4                   4                    Succeeded         "Basic"

get-azLoadBalancer -Name basic-lb-test2 -ResourceGroupName Lab         

ResourceGroupName Name           Location  ProvisioningState Sku Name
----------------- ----           --------  ----------------- --------
Lab               basic-lb-test2 centralus Succeeded         Basic
```

## Validation
Run validation to ensure the basic LB can be upgraded.
```PowerShell
Start-AzBasicLoadBalancerUpgrade -ResourceGroupName Lab -BasicLoadBalancerName basic-lb-test2 -validateScenarioOnly:$true
WARNING: Migration causes downtime for the application(s) using the Basic Load Balancer--usually a few minutes--see https://aka.ms/BasicLBMigrateDowntime.
2025-06-05T18:16:27+00 [Information]:############################## Initializing Start-AzBasicLoadBalancerUpgrade ##############################
2025-06-05T18:16:27+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] PowerShell Version: 7.5.0
2025-06-05T18:16:27+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] AzureBasicLoadBalancerUpgrade Version: 2.4.24
2025-06-05T18:16:27+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Checking that user is signed in to Azure PowerShell
2025-06-05T18:16:27+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] User is signed in to Azure with account 'XXXXXXX', subscription 'MySubscriptionName2' selected
2025-06-05T18:16:27+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Loading Azure Resources                         
2025-06-05T18:16:28+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Basic Load Balancer 'basic-lb-test2' in Resource Group 'Lab' loaded
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Verifying if Load Balancer basic-lb-test2 is valid for migration
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Verifying source load balancer SKU
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Source load balancer SKU is type Basic
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Determining whether basic load balancer is used by an AKS cluster
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Checking backend pool member types and that all backend pools are not empty
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] All backend pools members are virtualMachines!
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Checking that standard load balancer name 'basic-lb-test2'
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Load balancer resource 'basic-lb-test2' already exists. Checking if it is a Basic SKU for migration
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Load balancer resource 'basic-lb-test2' is a Basic Load Balancer. The same name will be re-used.
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Determining if LB is internal or external based on FrontEndIPConfiguration[0]'s IP configuration
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] FrontEndIPConfiguiration[0] is assigned a public IP address '/subscriptions/1111111-1111-1111-1111-111111111112/resourceGroups/Lab/providers/Microsoft.Network/publicIPAddresses/basic-lb-public-ip-test', so this LB is External
2025-06-05T18:16:28+00 [Information]:[Test-SupportedMigrationScenario] Determining if there is a frontend IPV6 configuration
2025-06-05T18:16:29+00 [Information]:[Test-SupportedMigrationScenario] Load balancer does not have a frontend IPV6 configuration
2025-06-05T18:16:30+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend pools contain members which are members of another load balancer's backend pools...
2025-06-05T18:16:30+00 [Information]:[Test-SupportedMigrationScenario] All VM load balancer associations are with the Basic LB(s) to be migrated.
2025-06-05T18:16:30+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend VMs have public IPs...       
2025-06-05T18:16:32+00 [Information]:[Test-SupportedMigrationScenario] Checking if load balancing rule has floating IP enabled and associated IP config is not primary...
2025-06-05T18:16:32+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend VMs are part of an Availability Set and if there are other members of the same availability set which are not part of the load balancer backend pool...                                                                                                                   
2025-06-05T18:16:32+00 [Information]:[Test-SupportedMigrationScenario] No Availability Sets found                       
2025-06-05T18:16:33+00 [Information]:[Test-SupportedMigrationScenario] Detected migration scenario: {"VMSSInstancesHavePublicIPs":false,"VMsHavePublicIPs":false,"ExternalOrInternal":"External","BackendType":"VM","SkipOutboundRuleCreationMultiBE":false}                                                                                                         
2025-06-05T18:16:33+00 [Information]:[Test-SupportedMigrationScenario] Load Balancer 'basic-lb-test2' is valid for migration
2025-06-05T18:16:33+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Scenario validation completed, exiting because -validateScenarioOnly was specified
```
The last two lines confirm the check has completed and that basic-lb-test2 a valid migration target.

## Starting the migration
The migration can be started by removing the -validateScenarioOnly:$true switch from the last command.

>[!NOTE]
> The duration of the upgrade process will depend on several factors.
> My upgrade only took a couple minutes but my configuration was very basic. 

The process will output two lines with JSON file names. The file starting with 'State' can be used for post-validation. The ARMTemplate file is an ARM template of the Basic SKU deployment. You can keep this in case you need to deploy the Basic SKU LB again before the retirement date - September 30, 2025.
```PowerShell
JSON backup Basic Load Balancer to file /home/USER/State_LB_basic-lb-test2_Lab_20250605T1825432837.json Completed
[Information]:[BackupBasicLoadBalancer] Completed export Basic Load Balancer ARM template to path '/home/USER/ARMTemplate_basic-lb-test2_Lab_20250605T1825432837.json'...
```

```PowerShell
Start-AzBasicLoadBalancerUpgrade -ResourceGroupName Lab -BasicLoadBalancerName basic-lb-test2
2025-06-05T18:25:37+00 [Information]:############################## Initializing Start-AzBasicLoadBalancerUpgrade ##############################
2025-06-05T18:25:37+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] PowerShell Version: 7.5.0
2025-06-05T18:25:37+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] AzureBasicLoadBalancerUpgrade Version: 2.4.24
2025-06-05T18:25:37+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Checking that user is signed in to Azure PowerShell
2025-06-05T18:25:37+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] User is signed in to Azure with account 'XXXXXXX', subscription 'MySubscriptionName2' selected
2025-06-05T18:25:37+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Loading Azure Resources                         
2025-06-05T18:25:38+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Basic Load Balancer 'basic-lb-test2' in Resource Group 'Lab' loaded
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Verifying if Load Balancer basic-lb-test2 is valid for migration
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Verifying source load balancer SKU
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Source load balancer SKU is type Basic
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Determining whether basic load balancer is used by an AKS cluster
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Checking backend pool member types and that all backend pools are not empty
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] All backend pools members are virtualMachines!
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Checking that standard load balancer name 'basic-lb-test2'
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Load balancer resource 'basic-lb-test2' already exists. Checking if it is a Basic SKU for migration
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Load balancer resource 'basic-lb-test2' is a Basic Load Balancer. The same name will be re-used.
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Determining if LB is internal or external based on FrontEndIPConfiguration[0]'s IP configuration
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] FrontEndIPConfiguiration[0] is assigned a public IP address '/subscriptions/1111111-1111-1111-1111-111111111112/resourceGroups/Lab/providers/Microsoft.Network/publicIPAddresses/basic-lb-public-ip-test', so this LB is External
2025-06-05T18:25:38+00 [Information]:[Test-SupportedMigrationScenario] Determining if there is a frontend IPV6 configuration
2025-06-05T18:25:39+00 [Information]:[Test-SupportedMigrationScenario] Load balancer does not have a frontend IPV6 configuration
2025-06-05T18:25:39+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend pools contain members which are members of another load balancer's backend pools...
2025-06-05T18:25:40+00 [Information]:[Test-SupportedMigrationScenario] All VM load balancer associations are with the Basic LB(s) to be migrated.
2025-06-05T18:25:40+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend VMs have public IPs...       
2025-06-05T18:25:42+00 [Information]:[Test-SupportedMigrationScenario] Checking if load balancing rule has floating IP enabled and associated IP config is not primary...
2025-06-05T18:25:42+00 [Information]:[Test-SupportedMigrationScenario] Checking if backend VMs are part of an Availability Set and if there are other members of the same availability set which are not part of the load balancer backend pool...                            
2025-06-05T18:25:42+00 [Information]:[Test-SupportedMigrationScenario] No Availability Sets found                       
2025-06-05T18:25:42+00 [Information]:[Test-SupportedMigrationScenario] Detected migration scenario: {"VMSSInstancesHavePublicIPs":false,"VMsHavePublicIPs":false,"ExternalOrInternal":"External","BackendType":"VM","SkipOutboundRuleCreationMultiBE":false}
2025-06-05T18:25:42+00 [Information]:[Test-SupportedMigrationScenario] Load Balancer 'basic-lb-test2' is valid for migration
2025-06-05T18:25:43+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Preparing for migration by backing up and deleteing the basic LB(s)
2025-06-05T18:25:43+00 [Information]:[LBMigrationPrep] Preparing load balancer 'basic-lb-test2' for migration           
2025-06-05T18:25:43+00 [Information]:[BackupBasicLoadBalancer] Initiating Backup of Basic Load Balancer Configurations to path '/home/USER'
2025-06-05T18:25:43+00 [Information]:[BackupBasicLoadBalancer] JSON backup Basic Load Balancer to file /home/USER/State_LB_basic-lb-test2_Lab_20250605T1825432837.json Completed
2025-06-05T18:25:43+00 [Information]:[BackupBasicLoadBalancer] Exporting Basic Load Balancer ARM template to path '/home/USER'...
2025-06-05T18:25:47+00 [Information]:[BackupBasicLoadBalancer] Completed export Basic Load Balancer ARM template to path '/home/USER/ARMTemplate_basic-lb-test2_Lab_20250605T1825432837.json'...
2025-06-05T18:25:47+00 [Information]:[LBPublicIPToStatic] Changing public IP addresses to static (if necessary)         
2025-06-05T18:25:48+00 [Information]:[LBPublicIPToStatic] 'basic-lb-public-ip-test' ('1.1.1.1') was using Dynamic IP, changing to Static IP allocation method.
2025-06-05T18:25:51+00 [Information]:[LBPublicIPToStatic] Completed the migration of 'basic-lb-public-ip-test' ('1.1.1.1') from Basic SKU and/or dynamic to static
2025-06-05T18:25:51+00 [Information]:[LBPublicIPToStatic] Public Frontend Migration Completed                           
2025-06-05T18:25:52+00 [Information]:[RemoveBasicLoadBalancer] Removing Basic Loadbalancer basic-lb-test2 from Resource Group Lab
2025-06-05T18:26:34+00 [Information]:[RemoveBasicLoadBalancer] Removal of Basic Loadbalancer basic-lb-test2 Completed   
2025-06-05T18:26:34+00 [Information]:[LBMigrationPrep] Completed preparing load balancer 'basic-lb-test2' for migration 
2025-06-05T18:26:34+00 [Information]:[Start-AzBasicLoadBalancerUpgrade] Starting migration for Basic Load Balancer 'basic-lb-test2' in Resource Group 'Lab'
2025-06-05T18:26:34+00 [Information]:[PublicLBMigrationVM] Public Load Balancer with VM backend detected. Initiating Public Load Balancer Migration
2025-06-05T18:26:34+00 [Information]:[UpgradeVMPublicIP] Starting upgrade of Public IP SKUs for VMs associated with the Basic Load Balancer basic-lb-test2
2025-06-05T18:26:34+00 [Information]:[UpgradeVMPublicIP] Querying Resource Graph for PIPs associated with VMs in the backend pool(s)...
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Found '0' Public IPs associcated with VMs in the Basic LB's backend pool
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Waiting for '0' PIP allocation method change jobs to complete before starting upgrade of Public IP SKUs
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Waiting for all '0' NIC detach jobs to complete before starting upgrade of Public IP SKUs
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Waiting for all '0' PIP SKU upgrade jobs to complete
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Waiting for '0' PIP reattach to NIC jobs to complete...
2025-06-05T18:26:35+00 [Information]:[UpgradeVMPublicIP] Completed upgrade of VM Public IP SKUs
2025-06-05T18:26:35+00 [Information]:[_CreateStandardLoadBalancer] Initiating Standard Load Balancer Creation           
2025-06-05T18:26:37+00 [Information]:[_CreateStandardLoadBalancer] Standard Load Balancer basic-lb-test2 created successfully
2025-06-05T18:26:37+00 [Information]:[PublicFEMigration] Initiating Public Frontend Migration                           
2025-06-05T18:26:38+00 [Information]:[PublicFEMigration] 'basic-lb-public-ip-test' ('1.1.1.1') is using Basic SKU, changing Standard SKU.
2025-06-05T18:26:41+00 [Information]:[PublicFEMigration] Completed the migration of 'basic-lb-public-ip-test' ('1.1.1.1') from Basic SKU and/or dynamic to static
2025-06-05T18:26:41+00 [Information]:[PublicFEMigration] Saving Standard Load Balancer basic-lb-test2                   
2025-06-05T18:26:48+00 [Information]:[PublicFEMigration] Public Frontend Migration Completed                            
2025-06-05T18:26:48+00 [Information]:[AddLoadBalancerBackendAddressPool] Adding BackendAddressPool basic-lb-test2bepool 
2025-06-05T18:26:48+00 [Information]:[AddLoadBalancerBackendAddressPool] Saving added BackendAddressPool to Standard Load Balancer basic-lb-test2
2025-06-05T18:26:50+00 [Information]:[ProbesMigration] Initiating Probes Migration                                      
2025-06-05T18:26:50+00 [Information]:[ProbesMigration] Adding Probe basic-lb-test-hp-port22 to Standard Load Balancer   
2025-06-05T18:26:50+00 [Information]:[ProbesMigration] Saving Standard Load Balancer basic-lb-test2                     
2025-06-05T18:26:53+00 [Information]:[ProbesMigration] Probes Migration Completed                                       
2025-06-05T18:26:53+00 [Information]:[LoadBalancingRulesMigration] Initiating LoadBalancing Rules Migration             
2025-06-05T18:26:53+00 [Information]:[LoadBalancingRulesMigration] Adding LoadBalancing Rule basic-lb-test-rule to Standard Load Balancer
2025-06-05T18:26:54+00 [Information]:[LoadBalancingRulesMigration] Saving Standard Load Balancer basic-lb-test2         
2025-06-05T18:26:57+00 [Information]:[LoadBalancingRulesMigration] LoadBalancing Rules Migration Completed              
2025-06-05T18:26:57+00 [Information]:[OutboundRulesCreation] Initiating Outbound Rules Creation                         
2025-06-05T18:26:57+00 [Information]:[OutboundRulesCreation] Adding Outbound Rule basic-lb-test2bepool to Standard Load Balancer
2025-06-05T18:26:58+00 [Information]:[OutboundRulesCreation] Saving Standard Load Balancer basic-lb-test2               
2025-06-05T18:27:00+00 [Information]:[OutboundRulesCreation] Outbound Rules Creation Completed                          
2025-06-05T18:27:00+00 [Information]:[NatRulesMigration] Initiating Nat Rules Migration                                 
2025-06-05T18:27:01+00 [Information]:[NatRulesMigration] Saving Standard Load Balancer basic-lb-test2                   
2025-06-05T18:27:01+00 [Information]:[NatRulesMigration] No NAT Rules to migrate. Skipping save.                        
2025-06-05T18:27:01+00 [Information]:[Start-NatPoolToNatRuleMigration] Starting NAT Pool to NAT Rule migration...       
2025-06-05T18:27:01+00 [Information]:[Start-NatPoolToNatRuleMigration] Load Balancer 'basic-lb-test2' does not have any Inbound NAT Pools to migrate
2025-06-05T18:27:01+00 [Information]:[NsgCreationVM] Initiating NSG Creation for VMs                                    
2025-06-05T18:27:01+00 [Information]:[NsgCreationVM] Looping all VMs in the backend pool of the Load Balancer           
2025-06-05T18:27:01+00 [Information]:[NsgCreationVM] Querying Resource Graph for current NIC and Subnet NSG associations...
2025-06-05T18:27:01+00 [Information]:[NsgCreationVM] Found '1' subnets associcated with VMs in the Basic LB's backend pool.
2025-06-05T18:27:02+00 [Information]:[NsgCreationVM] Checking NSGs to update or create based on Resource Graph data...  
2025-06-05T18:27:02+00 [Information]:[NsgCreationVM] Checking for existing NSGs at the subnet (it is assumed if an NSG exists, it allows LB traffic already)
2025-06-05T18:27:02+00 [Information]:[NsgCreationVM] NSG found on subnet '', NSG: ''                                    
2025-06-05T18:27:02+00 [Information]:[NsgCreationVM] Updating existing NSGs with new security rules for the LB...       
WARNING: 2025-06-05T18:27:02+00 [Warning]:[NsgCreationVM] Updating exising NSGs is not implemented; ensure your NSG '/subscriptions/1111111-1111-1111-1111-111111111112/resourceGroups/Lab/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd' has rules to allow traffic from the Load Balancer!                                                       
2025-06-05T18:27:02+00 [Information]:[NsgCreationVM] NSG Creation Completed                                             
2025-06-05T18:27:02+00 [Information]:[BackendPoolMigrationVM] Initiating Backend Pool Migration                         
2025-06-05T18:27:02+00 [Information]:[BackendPoolMigrationVM] Adding original VMs to the new Standard Load Balancer backend pools
2025-06-05T18:27:02+00 [Information]:[BackendPoolMigrationVM] Adding ipconfigs on NIC 'basic-vm-test-01739' to backend pools
2025-06-05T18:27:02+00 [Information]:[BackendPoolMigrationVM] Getting NIC '/subscriptions/1111111-1111-1111-1111-111111111112/resourceGroups/Lab/providers/Microsoft.Network/networkInterfaces/basic-vm-test-01739'
2025-06-05T18:27:03+00 [Information]:[BackendPoolMigrationVM] Getting IP Config 'ipconfig1' on NIC 'basic-vm-test-01739'
2025-06-05T18:27:03+00 [Information]:[BackendPoolMigrationVM] Waiting for all '1' NIC backend pool association jobs to complete
2025-06-05T18:27:47+00 [Information]:[BackendPoolMigrationVM] Backend Pool Migration Completed                          
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Initiating Validation of Migration for basic LB 'basic-lb-test2' to standard LB 'basic-lb-test2')'
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Backend type: VM                                               
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Standard Load Balancer exists                                  
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Standard Load Balancer is Standard SKU                         
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Standard Load Balancer has the same number of frontend IPs ('1') as the Basic Load Balancer ('1')
2025-06-05T18:27:47+00 [Information]:[ValidateMigration] Standard Load Balancer has the same number of backend pools plus the count added for NAT Pool migration ('') as the Basic Load Balancer ('1')
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer has the same number of load balancing rules ('1') as the Basic Load Balancer ('1')
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer has the same number of health probes ('1') as the Basic Load Balancer ('1')
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer has the expected number of NAT Rules ('0') when NAT Pools are migrated to NAT Rules (one per NAT Pool plus original NAT Rules). Standard load balancer has: '0'
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer's NAT Rules have the same backend IP configurations count as Basic Load Balancer's NAT rules
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer has no inbound NAT pools--migration was configured to upgrade them to NAT Rules
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer has outbound rules ('1')                
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] External Standard Load Balancer has Basic Load Balancer public IP address '/subscriptions/1111111-1111-1111-1111-111111111112/resourceGroups/Lab/providers/Microsoft.Network/publicIPAddresses/basic-lb-public-ip-test'
2025-06-05T18:27:48+00 [Information]:[ValidateMigration] Standard Load Balancer pool 'basic-lb-test2bepool' has the same membership as Basic Load Balancer pool 'basic-lb-test2bepool'
2025-06-05T18:27:49+00 [Information]:[ValidateMigration] All VM NICs have NSGs                                          
2025-06-05T18:27:49+00 [Information]:############################## Migration Completed ############################## 
```

## Post-Migration Check
The migration completed successfully. Check the resources using the same GET PS commands we originally ran also confirms the SKU is now Standard for both resources. Some may have noticed that basic-lb-public-ip-test was originally Dynamically allocated but it shows as Static now. The allocation method conversion is handled during migration process.

```PowerShell
get-AzPublicIPAddress -Name basic-lb-public-ip-test -ResourceGroupName Lab                                               

ResourceGroupName Name                    Location  PublicIpAllocationMethod IpAddress     PublicIpAddressVersion IdleTimeoutInMinutes ProvisioningState Sku
----------------- ----                    --------  ------------------------ ---------     ---------------------- -------------------- ----------------- ---
Lab               basic-lb-public-ip-test centralus Static                   1.1.1.1 IPv4                   4                    Succeeded         "Standard"

get-azLoadBalancer -Name basic-lb-test2 -ResourceGroupName Lab                                                           

ResourceGroupName Name           Location  ProvisioningState Sku Name
----------------- ----           --------  ----------------- --------
Lab               basic-lb-test2 centralus Succeeded         Standard
```

>[!NOTE]
> The resources used in this example have been deleted.
