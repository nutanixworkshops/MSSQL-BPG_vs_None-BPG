.. _deploy_mssql:

--------------------------------------
Deploying and preparing the MS SQL VMs
--------------------------------------

As we are going to see the difference between a BPG (Best Practice Guide) setup and a none BPG installation of MSSQL, this is the same for the other databases, we need to install two Microsoft SQL 2016 servers running Microsoft Windows 2016 Server installations.

Clone Existing VM
+++++++++++++++++

For the sake of time, we are going to clone the existing **MSSQL 2016** VM two times.

#. Select your assigned **MSSQL 2016** VM and click the **Clone** option
#. Changed the number of clones to 2 and provide the name of the VM to have your initials in the name
#. Update vCPU and RAM to have **8 vCPUs, 1 Core**, and **8GB of RAM** for the SQL servers

   .. figure:: images/18.png

#. Click the **Save** button
#. Rename the first clone to include **none-BPG** in its name. Example: **XY-MSSQL-none-BPG**
#. Rename the second clone to include **BPG** in its name. Example: **XY-MSSQL-BPG**

MSSQL VM for None BPG
++++++++++++++++++++++++

#. Select your **XY-MSSQL-none-BPG** VM and click Power on

#. Log in to your  **XY-MSSQL-none-BPG** (**Cancel** Shutdown Event Tracker):

   - **Username** - Administrator
   - **Password** - Nutanix/4u

Proceed to the next VM.

MSSQL for VM BPG
++++++++++++++++
#. Select your **XY-MSSQL-BPG** VM and click Update

#. Add the following 7 disks (storage container is Default)

   .. danger:: 

      Make sure to create the drives in this order, or the rest of the installation will FAIL
   
   #. Disk, 300 GB
   #. Disk, 300 GB
   #. Disk, 100 GB
   #. Disk, 100 GB
   #. Disk, 250 GB
   #. Disk, 150 GB
   #. Disk, 200 GB
   

#. Save the VM's update

#. Power on the VM

#. Log in to your  **XY-MSSQL-BPG** (**Cancel** Shutdown Event Tracker):

   - **Username** - Administrator
   - **Password** - Nutanix/4u

Configuring the two database servers
++++++++++++++++++++++++++++++++++++

Per server, run the below steps. We have tried to automate as much as we can so it doesn't become to tedious....

#. To configure the MSSQL database and the server itself, download the following zip file: https://raw.githubusercontent.com/wessenstam/MSSQL-BPG_vs_None-BPG/main/files/NTNX-HammerDB.zip in your Windows Server. The scripts in the zip file will set the following parameters:
   
   - Server platform:
     
     - If needed, this is for the BPG VM, creates the extra drives and formats them
     - Disables firewall to all profiles
     - Disables Windows updates
     - Disables IE security checks
     - Sets Windows power options to High Performance
     - Install .NET Framework Core
     - Configures automatic logon as administrator to the VM
     - Creates the needed directories
     - Reboots VM

   - MSSQL Server:
     
     - Install two important flags
     - Run SQL commands to:
       
       #. Making sure that the Instant File Initialization is enabled ((https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-instant-file-initialization?view=sql-server-ver15#:~:text=In%20SQL%20Server%2C%20instant%20file,files%20cannot%20be%20initialized%20instantaneously)
       #. Change the tempdb files for the SQL server
       #. Increase the size of the Log file
       #. Increase the SQL Memory assignment
       #. Change the default paths for the SQL server
       #. Set the DB Recovery mode


#. Unpack the downloaded zip file and move the directory (**NTNX-HammerDB**) to the root of **c:** using the Windows Explorer.

#. Run **Powershell -ExecutionPolicy ByPass** as the script has not been signed and therefore the Windows Server platform allows the execution of the script.

#. Run **cd c:\\NTNX-HammerDB**  in the Powershell Window

#. Run **./configure.ps1** to have the script run. The server will reboot at the end of the script.

#. Log back into the server via RDP 

#. Run the Following SQL scripts by double clicking them in the Windows Explorer. 

   .. note::
   
      This will open the MSSSQL Management Studio. In the Connect Screen, select the **.** for the server and Windows authentication and click the **Connect** button. Make sure to wait till each step has been run before moving on with the next step. Some steps may take up to two minutes!!!

   #. **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\0\0_Check_Instant_File_Initialization.sql** 
   #. Clik on the Execute button to have the script run. This checks if the Instant File Initialization has been configured. the result should be *We have instant file initialization*.
   #. Via **File -> Open -> File...** in the MSSQL Management Studio screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\01-ChangeTempDB.sql** and hit the Execute button to change the TempDB of the SQL server.
   #. Open a Microsoft command line (CMD) and run **net stop "SQL SERVER (MSSQLSERVER)" && net start "SQL SERVER (MSSQLSERVER)"** to restart the SQL server

      .. figure:: images/21.png

   #. In the MSSMS Click on the **+** Sign next to databases till you see **tempdb** under *System Databases*
   #. Right Click the **tempdb** and select **Properties** and select **Files**. 
   #. Remove **all** lines that have **C:\\Program Files\\...** in the name by selecting them and click the **Remove** button. This will also remove the physical files. The script in step 3 has added the other needed tempdb files.

      .. figure:: images/19.png

      .. note::
         The screenshot is from the none-BPG VM! For the BPG VM you will see F, G, H, I and L drives.

   #. Click on the **Ok** button to close the window
   #. Click **File -> Open -> File...** in the MSSQL Management Studio (MSSMS) screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\02-IncreaseTempDB_logs.sql** and hit the Execute button to extend the TemDB log file
   #. Click **File -> Open -> File...** in the MSSMS screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\03-SQL_set_memory.sql** and hit the Execute button to increase the SQL assigned memory
   #. Click **File -> Open -> File...** in the MSSMS screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\04-SQL-ChangeDefaultPaths.sql** and hit the Execute button to make sure the SQL server is using the newly created files
   #. Reboot the server to make sure the system is clean.
   #. Log back in to the Server using a RDP session
   #. Open the MSSMS select the **.** for the server and Windows authentication and hit **Connect**
   #. Click **File -> Open -> File...** in the MSSMS screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\create-ntnxdb\\Create-SQL2014-ntnxdb.sql** and and hit the Execute button to create the needed Database for use with the HammerDB application

      .. figure:: images/20.png

   #. Click **File -> Open -> File...** in the MSSMS screen and open **C:\\NTNX-HammerDB\\TSQL_Scripts\\All_versions\\05-Set-DB-RecoveryMode_benchmark.sql** and hit the Execute button to set the backup/recovery mode of the Database

#. Your Servers are now ready be to used with HammerDB.