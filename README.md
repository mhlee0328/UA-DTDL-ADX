# Introduction UA-DTDL-ADX

This project involves creating a datasource extension that maps OPC UA and Modbus with Azure Digital Twin Definition Language (DTDL). The OPC UA example used in this project is based on the umati information model. The goal of this project is to enable communication between OPC UA and Modbus devices with Azure Digital Twin, allowing for the creation of digital twins and their corresponding models. The project will be hosted on GitHub for easy access and collaboration.



# Installation

```powershell
git clone https://github.com/mhlee0328/UA-DTDL-ADX.git
```

The following environment variables **must** be defined in the docker-compose.yml:

* **AzureAd__Domain** - Your Azure Domain
* **AzureAd__TenantId** - Your TenantId
* **AzureAd__ClientId**  - Your Azure client ID of UA Cloud Twin. A client ID can be created through AAD app registration in the Azure portal under Azure Active Directory -> Overview -> Add -> App Registration
* **AzureAd__ClientSecret**- the Azure client secret of UA Cloud Twin. A client secret can be added after AAD app registration under Add a certificate or secret -> New client secret

To successfully connect to an Azure Digital Twins service instance, the above AAD app registration must be assigned to the Azure Digital Twins Data Owner role.

****

```yaml
version: '3.9'
services:
  db:
    container_name: umati-db
    image: mhlee0328/itri_mmsl_adx_umati_mariadb:0.0.1
    mem_limit: 5G
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: 123456
    ports:
      - 5053:3306
    volumes:
      - ./ContainerVolume/db:/var/lib/mysql
    networks:
      - umati_stack_nw
 


  api:
    container_name: umati-api
    image: mhlee0328/itri_mmsl_adx_umati_api:0.0.1
    mem_limit: 5G
    restart: unless-stopped
    environment:
      - ConnectionStrings__AdtLayoutDataBase=server=db;port=3306;database=adt_layout;user=root;password=123456
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=123456
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ./Certs/https:/https:ro
    networks:
      - umati_stack_nw
      
 
  web:
    container_name: umati-web
    image: mhlee0328/itri_mmsl_adx_umati_web:0.0.1
    mem_limit: 5G
    restart: unless-stopped
    ports:
      - 80:80
      - 443:443
    environment:
      - AzureAd__Instance=https://login.microsoftonline.com/
      - AzureAd__Domain=mhlee0328hotmail.onmicrosoft.com
      - AzureAd__TenantId=51557692-759d-47e4-aee6-2b7f8321ab76
      - AzureAd__ClientId=460982e1-2ba0-425b-9f5d-452ec1cee957
      - AzureAd__ClientSecret=peQ8Q~kCTnZIHDHH93g4vSvBEruUVg63.J4cHb9i
      - AzureAd__CallbackPath=/signin-oidc
      - Queue__Host=queue
      - Queue__Port=5672
      - Queue__Username=admin
      - Queue__Password=admin
      - Queue__SensorQueueName=sensor
      - Queue__CommandQueueName=command
      - BackendUrl=https://api/
      - OfflineMode__Enabled=false
      - EnableFactorySimulator=false
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=123456
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ./Certs/https:/https:ro
    networks:
      - umati_stack_nw
     

     
networks:
  umati_stack_nw:
    driver: bridge
```



# Usage

Run it on a Docker-enabled computer via:

    docker-compose up

Alternatively, you can run it in a Docker-enabled web application in the Cloud.

Then point your web browser to <http://yourIPAddress>