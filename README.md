# DMSPosgresMigration

Steps involved

1.Install posgress DB in Ubuntu server in onprem  and create databases
2.Create Azure  DB for Posgres
3. Migrate database from onprem- Azure using DMS

I have ubuntu machine created

![image](https://github.com/user-attachments/assets/26c7de49-bf13-4ec8-aa99-ed50b5e0e7c0)

I am connected to ubuntu server from my local machine

![image](https://github.com/user-attachments/assets/21a53577-ce37-46c2-b69b-4dca8061998f)

setting up posgress DB in ubuntu  by referring  https://www.postgresql.org/download/linux/ubuntu/


![image](https://github.com/user-attachments/assets/390e1466-7319-4e28-9250-616fd1b688d1)


![image](https://github.com/user-attachments/assets/7b081ed6-662a-4fef-afe8-29d78582910a)

We are completed posgress db setup


![image](https://github.com/user-attachments/assets/061d730b-42e8-4b6a-ba5a-122bd2b06746)

Connect to postgres using below commands 

![image](https://github.com/user-attachments/assets/6a0f1aa0-7826-4896-ace0-cb9869af3ec9)


Built in databases are listed here

![image](https://github.com/user-attachments/assets/3f25010c-65df-48e7-a857-e16effee2729)



creating new DB calle mydb

![image](https://github.com/user-attachments/assets/b9b67a1e-07b0-4870-8c1b-7c9228d34111)

postgres=# \c mydb
You are now connected to database "mydb" as user "postgres".

create table student and added few records

![image](https://github.com/user-attachments/assets/35814af7-dc25-4671-bbc1-1c735187cada)

Now I am trying to open this db in Pgamdin, before that we need to see config file 

![image](https://github.com/user-attachments/assets/5ed045b9-ab3d-4f50-9d38-1284867413ad)

edit file add host details

![image](https://github.com/user-attachments/assets/026d3e3f-fafc-439a-9825-f3dc584e2b83)

added my ip there. ( my laptop ip)

![image](https://github.com/user-attachments/assets/f2703e4b-9130-4cbc-a5db-86e9c4d0e9de)

![image](https://github.com/user-attachments/assets/0290fb82-f731-4874-b322-9b01b4b8c4cc)

changes from here


![image](https://github.com/user-attachments/assets/bbe36c9a-1625-4a3f-82ad-8b01a9a41217)



Creating  new DB in Azure to migrated DB we setup in ubuntu

![image](https://github.com/user-attachments/assets/cd21a5fb-02b4-4a12-9e63-c9c05229af40)


![image](https://github.com/user-attachments/assets/45f7c8c9-d8a2-449b-bfc5-a07152445171)

Now creating Same database mytest db in Azure DB



![image](https://github.com/user-attachments/assets/6799cf7b-da0c-4939-b468-c7b5a51c3aa7)


Now connect to Ubuntu Postgres

$ ssh -i vmforpostgres_key.pem bijuthottathil@20.83.46.38

![image](https://github.com/user-attachments/assets/a7cf933e-6c55-4c33-8cc4-192f939a8a90)

edit config file and change required options
sudo nano /etc/postgresql/16/main/postgresql.conf
![image](https://github.com/user-attachments/assets/dc3a42c5-aab7-4fd9-9621-c5173d54a942)

![image](https://github.com/user-attachments/assets/fda6621a-9d42-4f86-9054-5c3a69c9dbe5)

Always restart postgress after any config changes

sudo systemctl restart postgresql

next you need to create a role in on prem db


create role db_migrtionuser with
login
superuser
createdb
createrole
inherit
replication
connection limit -1
password 'xxxxxx'

![image](https://github.com/user-attachments/assets/10a4c955-39b5-4a12-a565-c315d45271bf)


connect to sql and see 

![image](https://github.com/user-attachments/assets/89e82e81-8771-444d-83f1-4ebe3c124683)

provide below access too

grant postgres to db_migrtionuser

sudo nano /etc/postgresql/16/main/pg_hba.conf

![image](https://github.com/user-attachments/assets/68494dee-cefa-4f32-aea6-844f239ae4ab)

new entry added

![image](https://github.com/user-attachments/assets/c7633823-82c8-45e4-a537-a57687c1882b)

now migrate sample schema from source DB to Azure DB  as given in https://learn.microsoft.com/en-us/azure/dms/tutorial-postgresql-azure-postgresql-online-portal

Schema is created after executing  pg_dump -O -h 20.83.46.38 -U postgres -d mytestdb -s > mytestdb_schema.sql
![image](https://github.com/user-attachments/assets/ca1a9d4b-2be2-4106-84ad-ee8385f78776)

![image](https://github.com/user-attachments/assets/28ebbe3a-2cd7-4016-9634-45abcbd4a12f)

![image](https://github.com/user-attachments/assets/42c9ce38-b236-4f2c-97c5-45019a3f366d)

![image](https://github.com/user-attachments/assets/42d60450-8607-489a-b257-b774d44899d9)



make sure to run  select pg_reload_conf();  to refresh all configuration changes

![image](https://github.com/user-attachments/assets/77e3a43f-c6be-4a69-8e6b-eecf20aad4d9)

Now run psql -h vmazurepostgres.postgres.database.azure.com  -U bijuthottathil -d mytestdb <  mytestdb_schema.sql

![image](https://github.com/user-attachments/assets/89f3c00c-40cc-42eb-9768-70925a5a0aa0)

Then Navigate to Azure Posgres db and see whether table is created or not

Yes. It worked. You can see student table is copied from onrprem db to azure postgres db

![image](https://github.com/user-attachments/assets/709f2088-6f10-4f33-aaa6-ea89c8d59a4e)

# Next step is create Database Migration Service in Azure


![image](https://github.com/user-attachments/assets/9dccae94-cea7-4115-a56d-99ed258538d2)


![image](https://github.com/user-attachments/assets/e5b8cfc0-0d79-4887-a1d7-227da4153897)




![image](https://github.com/user-attachments/assets/6fc6822d-2235-4210-a822-65d9b3f82615)  It can take 20 to 30 minutes
Now DMS Service is up

We need to create a project to proceed further

![image](https://github.com/user-attachments/assets/72cc2978-2dd2-470c-8ea7-82445a6f9166)

![image](https://github.com/user-attachments/assets/71a57e9f-b142-4f91-b1cf-87d6a876d5f3)

![image](https://github.com/user-attachments/assets/b1fe3287-19c1-4df6-87a2-4f23fd5442c5)

![image](https://github.com/user-attachments/assets/1b8e9d91-f247-4d9b-81e6-942f35a5e949)

I had postgres latest version  in Ubuntu, it was not supported by Azure DMS



