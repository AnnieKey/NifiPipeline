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

> Nifi pipeline: general view.

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

### GetFile processor
Creates FlowFiles from files in a directory. NiFi will ignore files it doesn't have at least read permissions for.

> Just add path to input directory.

![GetFile](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/get_file_conf.png)

### UpdateAttribute_AddSchemaName processor
Updates the Attributes for a FlowFile by using the Attribute Expression Language and/or deletes the attributes based on a regular expression.

> Add attribute schema name, which will be configured later.

![UpdateAttribute_AddSchemaName](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/update_attr_schema.png)

### ConvertRecord_CSVtoJSON processor
Converts records from one data format to another using configured Record Reader and Record Write Controller Services. The Reader and Writer must be configured with "matching" schemas. By this, we mean the schemas must have the same field names. The types of the fields do not have to be the same if a field value can be coerced from one type to another. For instance, if the input schema has a field named "balance" of type double, the output schema can have a field named "balance" with a type of string, double, or float. If any field is present in the input that is not present in the output, the field will be left out of the output. If any field is specified in the output schema but is not present in the input data/schema, then the field will not be present in the output or will have a null value, depending on the writer.

> Create and add Record Reader and Record Writer.

![ConvertRecord_CSVtoJSON](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/convert_record_config.png)

> Go to the Controller services configuration (press arrow) and enable services.

![Controllerserviser_config](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/controllerserviser_config.png)

> Configure AuroSchemaRegistry adding "users" field name and its value:
>{
>"name": "users",
> "namespace": "nifi",
>"type":"record",
>"fields":
> [
>  {"name": "id", "type": "int"},
>  {"name": "first_name", "type": "string"},
>  {"name": "last_name", "type": "string"},
>  {"name": "email", "type": "string"},
>  {"name": > "gender", "type": "string"},
>  {"name": "phone", "type": "string"}
> ]
>}

![Controllerserviser_config](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/AuroSchemaRegistry_config.png)

### SplitRecord processor
Splits up an input FlowFile that is in a record-oriented data format into multiple smaller FlowFiles.

> Create Record Reader and Record Writer.
> Add number of records to split into (10).
> And enable them in Controller Services configuration (as in previous one processor).

![SplitRecord](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/split_record_config.png)

### UpdateAttribute_AddJSONExpansion processor
Updates the Attributes for a FlowFile by using the Attribute Expression Language and/or deletes the attributes based on a regular expression.

> Add a "filename" attribute with value ${uuid}.json.
> uuid - is for unique name of every splitted flowfile.

![UpdateAttribute_AddJSONExpansion](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/update_attr_addJSONexpan.png)

### PutFile processor
Writes the contents of a FlowFile to the local file system.

> Just add path to output directory.

![PutFile](https://raw.githubusercontent.com/AnnieKey/NifiPipeline/master/Screenshots/put_file_config.png)

Results
---------------------

You can see it in inputData and outputData directories.












