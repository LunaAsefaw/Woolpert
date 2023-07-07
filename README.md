![Image Tag](https://img.shields.io/badge/Staging%20Image%20Tag:-0.0.1--6d140d0-blue.svg)
![Jenkins Status](https://img.shields.io/badge/Staging%20Jenkins%20Build%20Status:-SUCCESS-green.svg)

# Woolpert

The project is a GeoNode deployment with a core focus on managing infrastructure data.

## Repository Structure

1) Styles - This will store all the `SLD`, `CSS` styles that will be used by GeoServer.
2) Scripts - All scripts that can be run after deployment or during initialisation of the cluster.
3) Deployment - This stores all files and instructions relation to the deployment. The actual
Kubernetes manifest will not reside here.

# Populating Default GeoNode

1) Download the prepared data from the client.
2) Load the data into QGIS.
3) Explore the data and use the [Data QA Workbench](https://plugins.qgis.org/plugins/dataset_qa_workbench/) to 
load the data into a PostgreSQL database.
4) Use the script

    ```bash
    python3 deployment/populate_geoserver.py
    ```

    to populate the GeoServer instance with layers you have registered in the database.
    **Note** The script assumes you have installed a couple of python packages:

    ```bash
    pip3 install geoserver-rest
    pip3 install requests
    ```

5) Login to the GeoNode instance and navigate to the admin interface or access the page [Admin commands](https://S{SITE_URL}/admin/management_commands_http/) directly.
6) Setup a management job to update the layers in the GeoNode so that they can be visible in GeoNode.
![management_command](images/management_command_job.png)
7) Run the job and then inspect the layers in GeoNode.
