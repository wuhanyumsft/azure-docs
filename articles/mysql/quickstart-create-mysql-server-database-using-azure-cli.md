---
title: 'Quickstart: Create an Azure Database for MySQL server - Azure CLI'
description: This quickstart describes how to use the Azure CLI to create an Azure Database for MySQL server in an Azure resource group.
services: mysql
author: ajlam
ms.author: andrela
manager: kfile
editor: jasonwhowell
ms.service: mysql-database
ms.devlang: azure-cli
ms.topic: quickstart
ms.date: 04/01/2018
ms.custom: mvc
---

# Create an Azure Database for MySQL server using Azure CLI
This quickstart describes how to use the Azure CLI to create an Azure Database for MySQL server in an Azure resource group in about five minutes. The Azure CLI is used to create and manage Azure resources from the command line or in scripts.

If you don't have an Azure subscription, create a [free](https://azure.microsoft.com/free/) account before you begin.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

If you choose to install and use the CLI locally, this article requires that you are running the Azure CLI version 2.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI 2.0]( /cli/azure/install-azure-cli). 

If you have multiple subscriptions, choose the appropriate subscription in which the resource exists or is billed for. Select a specific subscription ID under your account using [az account set](/cli/azure/account#az_account_set) command.
```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## Create a resource group
Create an [Azure resource group](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) using the [az group create](/cli/azure/group#az_group_create) command. A resource group is a logical container into which Azure resources are deployed and managed as a group.

The following example creates a resource group named `myresourcegroup` in the `westus` location.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## Create an Azure Database for MySQL server
Create an Azure Database for MySQL server with the **[az mysql server create](/cli/azure/mysql/server#az_mysql_server_create)** command. A server can manage multiple databases. Typically, a separate database is used for each project or for each user.

The following example creates a server in West US named `mydemoserver` in your resource group `myresourcegroup` with server admin login `myadmin`. This is a **Gen 4** **General Purpose** server with 2 **vCores**. The name of a server maps to DNS name and is thus required to be globally unique in Azure. Substitute the `<server_admin_password>` with your own value.
```azurecli-interactive
az mysql server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen4_2 --version 5.7
```

## Configure firewall rule
Create an Azure Database for MySQL server-level firewall rule using the **[az mysql server firewall-rule create](/cli/azure/mysql/server/firewall-rule#az_mysql_server_firewall_rule_create)** command. A server-level firewall rule allows an external application, such as the **mysql.exe** command-line tool or MySQL Workbench to connect to your server through the Azure MySQL service firewall. 

The following example creates a firewall rule for a predefined address range, which in this example is the entire possible range of IP addresses.

```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowYourIP --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
```
Allowing all IP addresses is not secure. This example is provided for simplicity, but in a real-world scenario, you need to know the precise IP address ranges to add for your applications and users. 

> [!NOTE]
> Connections to Azure Database for MySQL communicate over port 3306. If you try to connect from within a corporate network, outbound traffic over port 3306 might not be allowed. If this is the case, you can't connect to your server unless your IT department opens port 3306.
> 


## Configure SSL settings
By default, SSL connections between your server and client applications are enforced. This default ensures security of "in-motion" data by encrypting the data stream over the internet. To make this quick start easier, disable SSL connections for your server. Disabling SSL is not recommended for production servers. For more information, see [Configure SSL connectivity in your application to securely connect to Azure Database for MySQL](./howto-configure-ssl.md).

The following example disables enforcing SSL on your MySQL server.
 
 ```azurecli-interactive
 az mysql server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled
 ```

## Get the connection information

To connect to your server, you need to provide host information and access credentials.

```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

The result is in JSON format. Make a note of the **fullyQualifiedDomainName** and **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen4",
    "name": "GP_Gen4_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## Connect to the server using the mysql.exe command-line tool
Connect to your server using the **mysql.exe** command-line tool. You can download MySQL from [here](https://dev.mysql.com/downloads/) and install it on your computer. Instead you may also click the **Try It** button on code samples, or the  `>_` button from the upper right toolbar in the Azure portal, and launch the **Azure Cloud Shell**.

Type the next commands: 

1. Connect to the server using **mysql** command-line tool:
```azurecli-interactive
 mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

2. View server status:
```sql
 mysql> status
```
If everything goes well, the command-line tool should output the following text:

```dos
C:\Users\>mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
Enter password: ***********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 65512
Server version: 5.6.26.0 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> status
--------------
mysql  Ver 14.14 Distrib 5.6.35, for Win64 (x86_64)

Connection id:          65512
Current database:
Current user:           myadmin@116.230.243.143
SSL:                    Not in use
Using delimiter:        ;
Server version:         5.6.26.0 MySQL Community Server (GPL)
Protocol version:       10
Connection:             mydemoserver.mysql.database.azure.com via TCP/IP
Server characterset:    latin1
Db     characterset:    latin1
Client characterset:    gbk
Conn.  characterset:    gbk
TCP port:               3306
Uptime:                 2 days 9 hours 47 min 20 sec

Threads: 4  Questions: 34833  Slow queries: 2  Opens: 84  Flush tables: 4  Open tables: 1  Queries per second avg: 0.167
--------------

mysql>
```

> [!TIP]
> For additional commands, see [MySQL 5.7 Reference Manual - Chapter 4.5.1](https://dev.mysql.com/doc/refman/5.7/en/mysql.html).

## Connect to the server using the MySQL Workbench GUI tool
1.	Launch the MySQL Workbench application on your client computer. You can download and install MySQL Workbench from [here](https://dev.mysql.com/downloads/workbench/).

2.	In the **Setup New Connection** dialog box, enter the following information on **Parameters** tab:

   ![setup new connection](./media/quickstart-create-mysql-server-database-using-azure-cli/setup-new-connection.png)

| **Setting** | **Suggested Value** | **Description** |
|---|---|---|
|	Connection Name | My Connection | Specify a label for this connection (this can be anything) |
| Connection Method | choose Standard (TCP/IP) | Use TCP/IP protocol to connect to Azure Database for MySQL> |
| Hostname | mydemoserver.mysql.database.azure.com | Server name you previously noted. |
| Port | 3306 | The default port for MySQL is used. |
| Username | myadmin@mydemoserver | The server admin login you previously noted. |
| Password | **** | Use the admin account password you configured earlier. |

Click **Test Connection** to test if all parameters are correctly configured.
Now, you can click the connection to successfully connect to the server.

## Clean up resources
If you don't need these resources for another quickstart/tutorial, you can delete them by doing the following command: 

```azurecli-interactive
az group delete --name myresourcegroup
```

If you would just like to delete the one newly created server, you can run **[az mysql server delete](/cli/azure/mysql/server#az_mysql_server_delete)** command.
```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## Next steps

> [!div class="nextstepaction"]
> [Design a MySQL Database with Azure CLI](./tutorial-design-database-using-cli.md)
