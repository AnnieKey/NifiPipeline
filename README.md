Instruction to reproduce pipeline
==================================
General
-----------------------------------
Nifi pipeline that does next:
- Ingest datasource file;
- convert row into JSON;
- split data into batches of some amount of rows (to create more than one file out of original one);
- write these batches of JSON into files.

Nifi pipeline needs to be run whithin Docker container.

 ![Pipeline](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/pipeline.png)
 
 Сustomize Docker container.
----------------------------------
1. Run container.
Command: **docker run -d -P -h nifi -p 38080:8080 -p 38181:8181 --name nifi_latest  -v $HOME/docker/volumes/nifi:/var/lib/data apache/nifi:latest**.
Check result command: **docker ps**.
Result: *CONTAINER ID        IMAGE                COMMAND                 CREATED             STATUS              PORTS                                                                                                 NAMES
269091c48766        apache/nifi:latest   "../scripts/start.sh"   2 hours ago         Up 2 hours          0.0.0.0:38080->8080/tcp, 0.0.0.0:38181->8181/tcp, 0.0.0.0:32780->8443/tcp, 0.0.0.0:32779->10000/tcp   nifi_latest*.

> -v -- allows to set a directory, where all data will be saved even after deleting the container;
> $HOME/docker/volumes/nifi: -- the directory on local machine;
> /var/lib/data -- the directory whithin container.

2. Add permissions to read/write whithin container.
Command: **docker exec -it  -u root 269091c48766 chmod 777 .**.

> 269091c48766 -- container ID.

3. Execute the container.
Command: **docker exec -it  -u root 269091c48766 bash**.
Result: *root@nifi:/opt/nifi/nifi-current#*.

4. Create directories for saving data.
Command: **cd /var/lib/data/**, **mkdir inputData**, **mkdir outputData**.
Check result command: **ls**.
Result: *root@nifi:/var/lib/data# ls
inputData  outputData*
Exit from container mode: **exit**.

5. Put the file with input data into the directory inputData.
Command: **docker cp /home/anniekey/Downloads/MOCK_DATA.csv 269091c48766:/var/lib/data/inputData/MOCK_DATA.csv**.

> /home/anniekey/Downloads/MOCK_DATA.csv -- path to file.

 Сustomize processors
----------------------------------



