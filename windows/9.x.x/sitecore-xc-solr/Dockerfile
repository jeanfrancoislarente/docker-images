# escape=`
ARG BUILD_IMAGE
ARG BASE_IMAGE
ARG ASSETS_IMAGE

FROM $ASSETS_IMAGE as assets
FROM $BUILD_IMAGE as builder

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG SOLR_VERSION
ARG CORE_NAMES
ARG ASSETS_USE_WDP

COPY --from=assets ["${ASSETS_USE_WDP}", "C:\\temp\\packages\\"]

# expand selected wdp into installation directory
RUN Expand-Archive -Path 'C:\\temp\\packages\\*.zip' -DestinationPath 'C:\\temp';

RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; `
    Invoke-WebRequest -Uri ('https://archive.apache.org/dist/lucene/solr/{0}/solr-{0}.zip' -f $env:SOLR_VERSION) -OutFile 'C:\\solr.zip'; `
    Expand-Archive -Path 'C:\\solr.zip' -DestinationPath 'C:\\temp'; `
    Move-Item -Path 'C:\\temp\\solr-*' -Destination 'C:\\solr';

COPY managed-schema.* C:\\temp\\

RUN New-Item -Path 'C:\\clean' -ItemType Directory | Out-Null; `
    Copy-Item -Path 'C:\\solr\\server\\solr\\*' -Destination 'C:\\clean' -Force -Recurse; `
    $env:CORE_NAMES -split ',' | ForEach-Object { `
        $name = $_.Trim(); `
        $schema = @{$true='C:\\temp\\managed-schema.xdb';$false='C:\\temp\\managed-schema.default'}[$name -like '*xdb*']; `
        Copy-Item -Path 'C:\\clean\\configsets\\_default\\conf' -Destination ('C:\\clean\\{0}\\conf' -f $name) -Recurse -Force; `
        Copy-Item -Path $schema -Destination ('C:\\clean\\{0}\\conf\\managed-schema' -f $name); `
        New-Item -Path ('C:\\clean\\{0}\\core.properties' -f $name) -ItemType File | Out-Null; `
        New-Item -Path ('C:\\clean\\{0}\\data' -f $name) -ItemType Directory | Out-Null; `
    }; `
    # CatalogItems
    $name = 'sitecoreCatalogItems'; `
    $schema = 'C:\\temp\\SolrSchemas\\CatalogItems\\managed-schema'; `
    Copy-Item -Path 'C:\\clean\\configsets\\_default\\conf' -Destination ('C:\\clean\\{0}\\conf' -f $name) -Recurse -Force; `
    Copy-Item -Path $schema -Destination ('C:\\clean\\{0}\\conf\\managed-schema' -f $name); `
    New-Item -Path ('C:\\clean\\{0}\\core.properties' -f $name) -ItemType File | Out-Null; `
    New-Item -Path ('C:\\clean\\{0}\\data' -f $name) -ItemType Directory | Out-Null; `
    # Customers
    $name = 'sitecoreCustomers'; `
    $schema = 'C:\\temp\\SolrSchemas\\Customers\\managed-schema'; `
    Copy-Item -Path 'C:\\clean\\configsets\\_default\\conf' -Destination ('C:\\clean\\{0}\\conf' -f $name) -Recurse -Force; `
    Copy-Item -Path $schema -Destination ('C:\\clean\\{0}\\conf\\managed-schema' -f $name); `
    New-Item -Path ('C:\\clean\\{0}\\core.properties' -f $name) -ItemType File | Out-Null; `
    New-Item -Path ('C:\\clean\\{0}\\data' -f $name) -ItemType Directory | Out-Null; `
    # Orders
    $name = 'sitecoreOrders'; `
    $schema = 'C:\\temp\\SolrSchemas\\Orders\\managed-schema'; `
    Copy-Item -Path 'C:\\clean\\configsets\\_default\\conf' -Destination ('C:\\clean\\{0}\\conf' -f $name) -Recurse -Force; `
    Copy-Item -Path $schema -Destination ('C:\\clean\\{0}\\conf\\managed-schema' -f $name); `
    New-Item -Path ('C:\\clean\\{0}\\core.properties' -f $name) -ItemType File | Out-Null; `
    New-Item -Path ('C:\\clean\\{0}\\data' -f $name) -ItemType Directory | Out-Null; `
    # Clean up
    Remove-Item -Path 'C:\\clean\\README.txt'; `
    Remove-Item -Path 'C:\\clean\\configsets' -Recurse;


FROM $BASE_IMAGE

USER ContainerAdministrator

COPY --from=builder ["C:\\solr", "C:\\solr"]
COPY --from=builder ["C:\\clean", "C:\\clean"]
COPY --from=builder ["C:\\windows\\system32\\find.exe", "C:\\windows\\system32\\"]

RUN MKDIR c:\\data

ENV SOLR_HOME=c:/data

EXPOSE 8983

COPY Boot.cmd .

CMD Boot.cmd c:\\solr 8983 c:\\clean c:\\data