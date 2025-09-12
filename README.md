# Download Microsoft Entra ID (formerly Azure AD)

Download the latest release from [Releases](https://github.com/bindcore/Microsoft-Entra-ID/releases/tag/2.3.20.0)

This guide presents a detailed, stepwise procedure for installing and configuring Microsoft Entra ID on Windows, enabling seamless synchronization between your existing on-premises Active Directory and Azure AD.

#### System Requirements

* **Operating System**: Windows Server 2016 or later (Windows Server 2022 is recommended).
* **Hardware**: At least 4 GB RAM and a dual-core processor.
* **Software**: Ensure that .NET Framework version 4.7.2 or higher is already installed on the server.

#### Account Permissions

* **Azure AD**: Global Administrator privileges are required.
* **On-Premises AD**: Enterprise Admin credentials are necessary.

#### Network and DNS

* Verify that DNS name resolution operates correctly for both your local AD and Azure AD domains.
* In this walkthrough, a non-routable internal domain is used; however, the same steps are applicable for routable domains such as **`yourcompany.com`**.

### Installation Steps

### Step 1: Prepare the Server

1. **Join the Server to the Domain**
   Ensure that the Windows Server is properly joined to your on-premises Active Directory domain.

2. **Install Required Roles**
   Open **PowerShell with administrative rights** and execute:

   ```powershell
   Install-WindowsFeature RSAT-ADDS
   ```

3. **Check Time Synchronization**
   Confirm that the server clock aligns with your domain controllers. If necessary, run:

   ```powershell
   w32tm /query /status
   ```

### Step 2: Install Azure AD Connect

1. **Download and Launch the Installer**
   Run the downloaded `AzureEntraID.exe` file.

   * If the installer reports missing prerequisites (e.g., unsupported .NET version or insufficient privileges), close it, resolve the issue, and restart the setup.

2. **Enable TLS 1.2 for .NET Framework**
   In an elevated PowerShell session, run:

   ```powershell
   # Enable strong cryptography
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1
   Set-ItemProperty -Path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Name 'SchUseStrongCrypto' -Value 1
   ```

   **Restart the server** to apply these changes.

   After rebooting, execute:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   ```

   To confirm TLS 1.2 is active:

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol
   ```

   The output should show `Tls12`.

3. **Start the Setup Wizard**

   * Accept the license agreement and click **Continue**.
   * Choose **Customize** to manually adjust installation options (recommended).

4. **Select Authentication Method**
   Pick one of the available sign-in options:

   * **Password Hash Synchronization** (default)
   * **Pass-through Authentication**
   * **Federation**

   Click **Next** to proceed.

### Step 3: Configure Azure AD Connect

1. **Sign In to Azure AD**
   Use an account with **Global Administrator** permissions.

2. **Connect to Local Active Directory**
   Enter **Enterprise Admin** credentials for the local AD.
   A warning may appear if your domain uses a `.local` or private suffix; it is safe to continue.

3. **Select Synchronization Scope**
   Determine which users and groups will be synchronized:

   * **Sync all users and groups**
   * **Filter by Organizational Unit (OU)**
   * **Filter by attribute**

   Click **Next** once your selection is complete.

4. **User Matching (Identifying Users)**
   Default settings typically suffice unless specific mappings are needed.

5. **Optional Features**
   Enable features according to your requirements:

   * **Password Writeback** – synchronizes password changes from Azure AD back to on-prem AD
   * **Group Writeback** – replicates Azure AD groups to the on-prem AD
   * **Device Writeback** – supports Hybrid Azure AD Join

### Step 4: Finalize the Installation

1. **Review and Install**
   Verify all selected options and click **Install**.

2. **Initial Synchronization**
   After installation, the Synchronization Service Manager will launch and start the first sync cycle automatically.

3. **Verify in Azure**
   Sign into the Azure portal, navigate to **Azure Active Directory > Users**, and confirm that on-prem AD users are listed.

### Post-Installation Configuration

#### Check Sync Status

1. Open **Synchronization Service Manager**.
2. Review synchronization history for any warnings or errors.

#### Trigger Manual Sync

In **PowerShell (Admin)**, run:

* **Delta Sync (changes only)**

  ```powershell
  Start-ADSyncSyncCycle -PolicyType Delta
  ```

* **Full Sync**

  ```powershell
  Start-ADSyncSyncCycle -PolicyType Initial
  ```

#### Enable Hybrid Azure AD Join (Optional)

1. Open **Azure AD Connect**.
2. Click **Device Options**.
3. Select **Configure Hybrid Azure AD Join** and follow the wizard.

### Group Policy Changes (if required)

If Group Policy has disabled active scripting in Internet Explorer, re-enable it as follows:

1. Press `Win + R`, type `gpedit.msc`, and press Enter.
2. Navigate to:
   `User Configuration > Administrative Templates > Windows Components > Internet Explorer > Internet Control Panel > Security Page > Internet Zone`
3. Set **Allow active scripting** to **Enabled**.

### Best Practices

* **Perform Regular Backups**
  Periodically export Azure AD Connect settings for disaster recovery

* **Monitor System Health**
  Use Synchronization Service Manager to review logs and confirm successful sync operations
