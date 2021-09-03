## Prerequisites

* An active [Azure subscription](https://azure.microsoft.com/en-us/free/).
* Sufficient access to create resources and register an application.

## Usage

1. **Copy** the PowerShell code snippet below.
```powershell
$uri = "https://raw.githubusercontent.com/tayganr/MCW-Azure-Synapse-Analytics-and-AI/master/scripts/preDeploymentScript.ps1"
Invoke-WebRequest $uri -OutFile "preDeploymentScript.ps1"
./preDeploymentScript.ps1
  ```
2. Navigate to the [Azure Portal](https://portal.azure.com), open the **Cloud Shell**.
![Azure Portal Cloud Shell](https://raw.githubusercontent.com/tayganr/purviewdemo/main/images/azure_portal_cloud_shell.png)

3. **Paste** the code snippet. Wait until the deployment is complete.