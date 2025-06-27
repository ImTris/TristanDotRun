---
title: "Dev Config"
date: 2025-06-26
layout: "page"
---

Below is a batch script that uses winget to install all my core development tools. Run this script as Administrator in Command Prompt or Windows Terminal:

```batch
:: Windows Dev Environment Setup Script

winget install --id Docker.DockerDesktop -e --accept-source-agreements --accept-package-agreements
winget install --id Git.Git -e --accept-source-agreements --accept-package-agreements
winget install --id Notepad++.Notepad++ -e --accept-source-agreements --accept-package-agreements
winget install --id Zen-Team.Zen-Browser -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.Azure.FunctionsCoreTools -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.Azure.StorageExplorer -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.AzureCLI -e --accept-source-agreements --accept-package-agreements
winget install --id OpenJS.NodeJS.LTS -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.PowerShell -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.VisualStudio.2022.Professional -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.VisualStudio.2019.BuildTools -e --accept-source-agreements --accept-package-agreements
winget install --id mRemoteNG.mRemoteNG -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.SQLServerManagementStudio -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.DotNet.DesktopRuntime.8 -e --accept-source-agreements --accept-package-agreements
winget install --id Dapr.CLI -e --accept-source-agreements --accept-package-agreements
winget install --id JetBrains.Toolbox -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.AzureDataStudio -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.VisualStudioCode -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.Bicep -e --accept-source-agreements --accept-package-agreements
winget install --id Microsoft.PowerToys -e --accept-source-agreements --accept-package-agreements

echo All packages processed. Please review any prompts above.
pause
```

- Make sure you have [winget](https://learn.microsoft.com/en-us/windows/package-manager/winget/) installed (included by default on Windows 10/11).
- Some packages may require manual steps or additional configuration after installation.

---