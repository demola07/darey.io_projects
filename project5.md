# PROJECT 5: CLIENT-SERVER IMPLEMENTATION using a MYSQL Relatonal Database Management System

## Install the `mysql-client` on the Client Machine

- `sudo apt install mysql-client`

  ![client_connection](./project5_images//mysql-client.JPG)

---

## Install `mysql-server` on the Server machine

- `sudo apt install mysql-server`

- To check if `mysql` service is enables, run `systemctl is-enabled myqsl`

- if `mysql` service is not enabled, run `systemctl enable mysql`

- `sudo mysql`

  ![server_connection](./project5_images//mysl-server.JPG)

  ### Create user on the Server machine

  - `CREATE USER '<username>'@'%' IDENTIFIED WITH mysql_native_password BY '<password>';`

  ### Create Database

  - `create DATABASE <database_name>;`

  ### Grant Privileges

  - `GRANT PRIVILEGE ON <database_name>.* TO '<username>'@'%' WITH GRANT OPTION;`

  ### Reloads the grant tables

  - `FLUSH PRIVILEGES`

    ![create_database](./project5_images//database_creation.JPG)

    ![edit_myqsld.cnf](./project5_images//edit_mysqld.cnf_file.JPG)

    ![edit_inbound_rules](./project5_images//edit_inbound_rules.JPG)

  ***

## Connect to Server machine from client

- `sudo mysql -u <username> -h <server_private_ip> -p`

  ![connect_to_server_from_client](./project5_images//connect_to_server_from_client.JPG)
