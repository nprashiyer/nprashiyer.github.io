### App Service Environment

An App Service Environment is an Azure App Service feature that provides a fully isolated and dedicated environment for running App Service apps securely at high scale. An App Service Environment is a single-tenant deployment of Azure App Service that runs on your virtual network.
<br>

Applications are hosted in App Service plans, which are created in an App Service Environment. An App Service plan is essentially a provisioning profile for an application host. As you scale out your App Service plan, you create more application hosts with all the apps in that App Service plan on each host. A single App Service Environment v3 can have up to 200 total App Service plan instances across all the App Service plans combined. A single App Service Isolated v2 (Iv2) plan can have up to 100 instances by itself.

When you're deploying onto dedicated hardware (hosts), you're limited in scaling across all App Service plans to the number of cores in this type of environment. An App Service Environment that's deployed on dedicated hosts has 132 vCores available. I1v2 uses two vCores, I2v2 uses four vCores, and I3v2 uses eight vCores per instance.

Support zone redundancy now

**VNet Integration**
The App Service Environment feature is a deployment of Azure App Service into a single subnet on a virtual network. When you deploy an app into an App Service Environment, the app is exposed on the inbound address that's assigned to the App Service Environment. If your App Service Environment is deployed with an internal virtual IP (VIP) address, the inbound address for all the apps will be an address in the App Service Environment subnet. If your App Service Environment is deployed with an external VIP address, the inbound address will be an internet-addressable address, and your apps will be in a public Domain Name System.

ILB - domain - `contoso.appserviceenvironment.net` <br>
External VIP - domain - `contoso.p.azurewebsites.net` <brs>

**Application Gateway Integration With ASE**