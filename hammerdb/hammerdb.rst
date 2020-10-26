.. _hammerdb:

------------------------------------------------
Performance Testing of MSSQL using HammerDB
------------------------------------------------

The this part of the lab you will install the HammerDB tool and use it to benchmark MSSQL database performance on the two VMs


Lab Agenda
^^^^^^^^^^^

#. Test database performance on MS SQL database configured without following Best Practices - here all the database files are located in a single OS drive

#. Test database performance on MS SQL database configured following Best Practices
   
Lab 1 - none BPG install
^^^^^^^^^^^^^^^^^^^^^^^^

#. Log in to the VM using Remote Desktop Client/Console into your **none-BPG** SQL server

#. Download the HammerDB setup binaries on your VM from `here <http://10.42.194.11/workshop_staging/HammerDB/HammerDB-3.3-Win-x86-64-Setup.exe>`_. (Copy link address)

#. When asked, click **Save**

#. Click **Run** to start the installation of HammerDB

#. Install HammerDB using the instuctions `here <https://www.hammerdb.com/docs/ch01s04.html#d0e166>`_ and make sure to install HammerDB in ``C:`` drive (default).

   .. note::
      If the installation URL doesn't work. Install the **exe** file as you would install any normal windows package. It is as simple as clicking **Next**.

#. Click the Finish in the last screen of the installer to start HammerDB

   .. figure:: images/1.png

Database Test 1 (without best practices on SQL)
***********************************************

Let's simulate a scenario where best practices for MS SQL databases are not followed. In this screnario the data and log files for a SQL database is in the same drive.

#. Double click on **SQL**

   .. figure:: images/dblclicksql.png

#. Select **SQL-Server** and **TPC-C** options and click on **OK**

   .. figure:: images/selsqltpc-c.png

#. Expand Schema Build and Double click on Options.

#. Change **Sql Server Database** name to **ntnxdb** as that is the database we have created earlier.

#. Change number of warehouses to 150.

#. Change virtual users to build schema to 16.

#. Click on **OK**

   .. figure:: images/warehousevirtualusers.png

#. Double click on **Build** option. click on Yes, data will start building.

   .. figure:: images/dblclickbuild.png

#. Click on **Start Transaction Counter** and observer transactions.

   .. figure:: images/starttrncnt.png

   .. figure:: images/trncnt.png

#. Wait until the data gets generated. This generates up to 15GB of data.

   .. note::
      It may take from 30-45 minutes for data population. You may start to prepare the next VM by repeating steps 1-10 of Lab 1. Once you are done with threse steps for the other VM, continue step #11 below for the NONE-BPG VM.

#. Once the data is generated (The Status column will all have Green Ticks),click on Destroy Virtual Users.

   .. figure:: images/destroyvirtusers.png

#. Double click on **Driver Script > Options**. Make sure **SQL Server Database** name is **ntnxdb** (the database you created in the previous few steps).

#. Select "TPC-C driver script" as **Timed Driver Script**.

#. Leave rest of them as-is and select **OK**.

   .. note::
      **Optional:** You can also try using the option **Keying and thinking time** for making the IOPS more intensive.

   .. figure:: images/drvscript.png

#. Double click on **Load**

#. Go to **Virtual users** and click on **Options**.

#. Make sure **Virtual users** in the popped-up window is 17 and click **OK**

#. Double click on **Create** and then double click on **Run** operations.\

   .. figure:: images/setvirtusers.png

#. While IO is getting generated, click on **Transactions Counter** and note the **TPM**. (Start the TPM counter if not already started)

   .. figure:: images/multitpm.png

#. Take screenshots and send TPM results to prospective customers or use it for your own reference.

   .. note::
      Please note that the test used here are using heavy I/O. Consider changing them in your own test to suit customers workloads.

#. Also check the **I/O Metrics** in Prism Element to see if you can observe I/O patterns, latencies, SSD/HDD usage and block sizes of files used by the VM you are running HammerDB tests on.

   .. figure:: images/vmiopattern.png



Lab 2 - BPG install
^^^^^^^^^^^^^^^^^^^

In this section you will run the same HammerDB Test on a MS SQL Server that has the BPG for database implemented. At the end you will be able to compare the results.

Database Test 2 (with best practices on SQL)
********************************************

#. Log in to the VM using Remote Desktop Client/Console into your **BPG** SQL server

#. Download the HammerDB setup binaries on your VM from `here <http://10.42.194.11/workshop_staging/HammerDB/HammerDB-3.3-Win-x86-64-Setup.exe>`_. (Copy link address)

#. When asked, click **Save**

#. Click **Run** to start the installation of HammerDB

#. Install HammerDB using the instuctions `here <https://www.hammerdb.com/docs/ch01s04.html#d0e166>`_ and make sure to install HammerDB in ``C:`` drive (default).

   .. note::
      If the installation URL doesn't work. Install the **exe** file as you would install any normal windows package. It is as simple as clicking **Next**.

#. Click the Finish in the last screen of the installer to start HammerDB

   .. figure:: images/1.png

#. Repeat steps 1 - 21 from Lab 1, but now for your BPG installed MSSQL server

#. Take screenshots and compare the TPM differences.

------

Takeaways
^^^^^^^^^^

#. HammerDB gives you a way to test DB performance with dummy data that it generates

   .. figure:: images/none-bpg.png

   vs.

   .. figure:: images/bpg.png

#. HammerDB is free and easy to use

#. Following best practices is the key to SQL DB Performance

#. Always right-size DB and DB Servers (do not over-provision or under-provision)

#. Don;t just use Nutanix Move to migrate a MS SQL Server to AHV before checking the BPG against the MS SQL Server

#. Introduce performance benchmarking to your customers as much as possible. It will make your life easier
