# Script:

```
# Import required module
Import-Module PnP.PowerShell

# Retrieve variables from Automation Account
$baseUrl = Get-AutomationVariable -Name "BaseUrl"
$sitePrefix = Get-AutomationVariable -Name "SitePrefix"
$testSitesNumber = Get-AutomationVariable -Name "TestSitesNumber"
$templatePath = Get-AutomationVariable -Name "TemplatePath"
$clientId = Get-AutomationVariable -Name "ClientId"
$clientSecret = Get-AutomationVariable -Name "ClientSecret"
$tenantId = Get-AutomationVariable -Name "TenantId"
$numberOfSites = Get-AutomationVariable -Name "NumberOfSites"

# Function to create and apply template to a site
function Process-Site {
    param (
        [string]$siteUrl,
        [string]$templatePath,
        [string]$clientId,
        [string]$clientSecret,
        [string]$tenantId
    )

    try {
        # Extract the site name from the URL
        $siteName = $siteUrl.Split("/")[-1]
        $siteTitle = "Site for $siteName"

        # Log the current processing site
        Write-Output "Processing site: $siteUrl"

        # Authenticate to SharePoint
        Connect-PnPOnline -Url $baseUrl -ClientId $clientId -ClientSecret $clientSecret -Tenant $tenantId

        # Create the site
        $newSite = New-PnPSite -Type CommunicationSite -Title $siteTitle -Url $siteUrl -Owner $clientId -ErrorAction Stop
        Write-Output "Site creation response: $($newSite | ConvertTo-Json)"

        # Wait for a short period to ensure the site is fully provisioned
        Start-Sleep -Seconds 30

        # Connect to the new site to apply the template
        Connect-PnPOnline -Url $siteUrl -ClientId $clientId -ClientSecret $clientSecret -Tenant $tenantId

        # Apply the site template
        Invoke-PnPSiteTemplate -Path $templatePath -ErrorAction Stop
        Write-Output "Successfully applied template to $siteUrl"
    } catch {
        Write-Output "Failed to create or apply template to site: $siteUrl. Error: $_"
    }
}
```

```
# Generate site URLs dynamically for testing
$siteUrls = 1..$testSitesNumber | ForEach-Object { "$baseUrl$sitePrefix$_" }

# Loop through each site URL and process it
foreach ($siteUrl in $siteUrls) {
    Process-Site -siteUrl $siteUrl -templatePath $templatePath -clientId $clientId -clientSecret $clientSecret -tenantId $tenantId
}
```

# Steps to Execute:

## Upload the Script to Azure Automation:

Go to your Azure Automation account.
Create a new runbook and paste the script above.
Save and publish the runbook.

## Run the Runbook:

Start the runbook and monitor its execution.
Check the output for any errors and make sure sites are being created and templates applied correctly.

## Important Notes:

- Ensure that the ClientId, ClientSecret, and TenantId are correctly configured in your Azure Automation account variables.
- The script is designed to use the minimal approach and should generate logs indicating what went wrong.
- Adjust the number of sites to be created by modifying the TestSitesNumber variable.
