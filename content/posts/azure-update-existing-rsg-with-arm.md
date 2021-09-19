---
title: 'Azure - Update Existing RSG with ARM Template'
date: 2018-08-10T14:03:00.000-07:00
draft: false
url: /2018/08/azure-update-existing-rsg-with-arm.html
---

Do you ever find yourself face this kind the situation: You are told to provision new resources with ARM templates to an existing resource group that already has VMs and vNets built and running. How can you add new subnets and VMs to the resource group without breaking those ones already there?

Unlike AWS Cloud Formation Templates, Microsoft ARM Templates do not provide “Update” option for past deployments. In order to modify the existing environment, the usual option is to make the change through CLI, PowerShell or Azure portal. But what if you somehow have to make the change through ARM template?

One way to do this is to create a template that enlists all existing resources and then add changes to it.

Below is a lab setup I created through portal. Assume there is a requirement to create another subnet in the same vNet.

[![Image](https://lh3.googleusercontent.com/-zdY2MTJ220A/W239ez5FVKI/AAAAAAAAKLQ/yTVg_CazjHcnfOQA1ylzMe0PWan193rEgCHMYCw/Image_thumb4?imgmax=800 "Image")](https://lh3.googleusercontent.com/-63puMomjfFE/W239dxOUQPI/AAAAAAAAKLM/wy8Xlu2W7nMa7IhORRljqQafijdYU2OBACHMYCw/s1600-h/Image14)

First, we use Azure Resource Group’s **Automation script** feature to generate a ARM template based on the existing resources.

[![image](https://lh3.googleusercontent.com/-TIDpoUfy6OY/W239hV643pI/AAAAAAAAKLY/eP0zvNzZGpQfxiDSzNm2MsVNquhdkGMFgCHMYCw/image_thumb1?imgmax=800 "image")](https://lh3.googleusercontent.com/-ZlAqXtDik30/W239f7bMuWI/AAAAAAAAKLU/vsPKFi8lkgEL8Wxmb9_o6mF_J5YxY0HnACHMYCw/s1600-h/image5)

**Reformat and Cleanup ARM Template**

Next, download the whole deployment package as a ZIP file and Extract to local folder.

Open the folder in **Visual Studio Code**. The only files we will be changing are “template.json” and “parameters.json”.

Enable GIT for source control.

[![image](https://lh3.googleusercontent.com/-Q0WCl3WdZ3M/W25j0Qo0evI/AAAAAAAAKMc/w_y5_Fv8KUs-9zDcQFy7sWYviM3WLCxAACHMYCw/image_thumb%255B1%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-IqpXxAuatBM/W25jzbWJFVI/AAAAAAAAKMY/AYXp86Im0G8TB6udPbMXfaLxxMx-CHArgCHMYCw/s1600-h/image%255B5%255D)

Because all we want is to update existing resource, we will need to keep all the parameter values same as the resources already created. To do that, we convert all the parameters to variables.

In **parameters.json** file, cut all the parameters to a new file.

The parameters are list in the format below. Conveniently it has its current value appended in the parameter name. So first, we move this value into the “value” field.

```
"virtualMachines\_proxyvm1\_name": {  
            "value": null  
         },  

```

After the change, the parameter looks like this:

```
"virtualMachines\_name": {  
             "value": proxyvm1  
         },  

```

Next, remove “{}” and “value:” from the new file. (VS Tip: use Ctrl + F2 to edit multiple phrases)

```
"virtualMachines\_name": "proxyvm1",  
"virtualNetworks\_name": "tomlabVNET1",  
"networkInterfaces\_name": "proxyVMNic",  
"networkSecurityGroups\_name": "tomlabLinuxSG",  
"publicIPAddresses\_name": "tomlabpubdns1940326763",  
"storageAccounts\_name": "visuatomlabprox072316000",  
"virtualMachines\_id": "/subscriptions/911c1a11-0db1-11a1-b011-fa1a1e01ce11/resourceGroups/tomlab-kr/providers/Microsoft.Compute/disks/proxyvm1\_OsDisk\_1\_11a11f1c0c11111db1fcab111aa11cd1"  

```

Paste the contents from the new file to Template.json Variables section.

We now need to remove the parameters from the “parameters{ }” section in **template.json**, as they are replaced with variables.

[![Image](https://lh3.googleusercontent.com/-IQyVaBXfywI/W25kszHkiZI/AAAAAAAAKNI/gbO1ElZZgPUty0Ga6J5tCD2cI3OYdwHyQCHMYCw/Image_thumb%255B2%255D?imgmax=800 "Image")](https://lh3.googleusercontent.com/-P47t7-ksYhU/W25kr0maVfI/AAAAAAAAKNE/779qRbJrqf0HyknVxs0PqBaiLD-4hn5VgCHMYCw/s1600-h/Image%255B8%255D)

We also need to replace the parameters in the “resources{ }” section with “variables” and their changed names as well. This is the most tedious part of the conversion. I found **VS Code**’s multi-selection feature is quite handy to do this job.

To further clean the template, remove “comments” and “provisioningState” lines.

[![image](https://lh3.googleusercontent.com/-FISZ9aVu0tI/W25kuRweBcI/AAAAAAAAKNQ/yYTbssZeFVMh9ITkPGSBXafw0oCDYBFrQCHMYCw/image_thumb%255B3%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-hsDBwuC07Z0/W25ktnEPmlI/AAAAAAAAKNM/_RkiZUCD0jwRHTXTNHFe_dRTKrH1CyrwQCHMYCw/s1600-h/image%255B11%255D)

[![image](https://lh3.googleusercontent.com/-xTPHL709uvY/W25kvyG--TI/AAAAAAAAKNY/DTbm7JrtPX45zQ7btda0dV7xrkjSPVhqQCHMYCw/image_thumb%255B4%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-SHFZX6JYl7o/W25kvBMyabI/AAAAAAAAKNU/j3kVUfcbuOkuJJTiKA8DWvSrG0BDZfoZgCHMYCw/s1600-h/image%255B14%255D)

After the modification your template should have all parameters replaced with variables.

**Validate the ARM Template**

Use Azure CLI to validate the template (you will need to sign in first with az login). Since there is no parameter used, we don’t need to specify the parameter file location.

```
az group deployment validate \--resource\-group tomlab\-kr \--template\-file .\\template.json   

```

Once confirm the template is all valid, we can now add the new resource, in this case a new subnet to this template.

The new subnet is added as a property of **virtualNetwork.**

```
"subnets": \[  
                    {  
                        "name": "proxysubnet",  
                        "properties": {  
                            "addressPrefix": "192.168.188.0/24"  
                        }  
                    },  
                    {  
                        "name": "secondSubnet",  
                        "properties": {  
                            "addressPrefix": "192.168.180.0/24"  
                        }  
                    }  
                \],  

```

Save the template and validate it again.

**Deploy the Template**

Now, let’s deploy the template with CLI.

```
az group deployment create \--resource\-group tomlab\-kr \--template\-file .\\template.json  

```

If all goes well, you should see the CLI returns the template in JSON.

Check in Azure portal, under the Resource Group –> Deployments.

[![image](https://lh3.googleusercontent.com/-_jKVwTdWNbk/W25j2IUIJZI/AAAAAAAAKMk/ZtmuISbs1WcSMW5Hh5D7p5AiqKo4CORFwCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-RKnAZRXLQyc/W25j1IC_9qI/AAAAAAAAKMg/nT62XvIltZgYJONxmkp4RETkAonis0ijQCHMYCw/s1600-h/image%255B2%255D)

Open Azure Portal and select **Virtual networks**, click **Diagram.** We can see the new subnet “secondSubnet” is now added. All existing resources are still intact.

[![Image](https://lh3.googleusercontent.com/-o_eJbQCWydY/W25j4cy2zVI/AAAAAAAAKMs/2Ksmrd4j7u09CNvmnZczHyc9GPMhWwtqQCHMYCw/Image_thumb4%255B1%255D?imgmax=800 "Image")](https://lh3.googleusercontent.com/-D_mBWuW-E-g/W25j240kNLI/AAAAAAAAKMo/nU5pFhTfVDc6ukEpVooCna1Ynk6nXuJugCHMYCw/s1600-h/Image14)