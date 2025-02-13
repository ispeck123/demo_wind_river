1. Prepare & Deploy mysql image:

    A. Pull the docker image 'mysql:8.0' from DockerHub.
        Use command:
            docker pull mysql:8.0
    
    B. Configure the below details in 'kubevirt-mysql.yaml' file.
        o. MySQL Version
        o. Database Name
        o. User Credentials:
                - Root User
                - Root Password
                - Database User
                - User Password
                - Ports
        o. Storage: emptyDir (Data will be lost if the pod is restarted)

    C. Make sure your Kubernetes cluster is active.
        Use command:
            kubectl cluster-info
        If using Minikube, start it with:
            minikube start

    D. Apply the MySQL Deployment File.
        Run the following command to deploy the MySQL pod and service:
            kubectl apply -f kubevirt-mysql.yaml
    E. Verify Pod Status.
        Use command:
            kubectl get pods

    
2. Create the Database Schema:

    A. Install MySQL Workbench.

    B. Open the MySQL Workbench application and Create a MySQL Connections.
        Use your MySQL credentials:
            o. Username
            o. Password
            o. Host
            o. Port
    
    C. Create the database schema from the MySQL dump file 'POWER_AUTOMATION_DUMP.sql'.

3. Configure the YAML files within model folder.

    A. Configure 'intrution.yaml', 'people_count.yaml', 'ppe.yaml':

        o. Configure the values of the environmet variables (DB_HOST, DB_PORT, DB_USER, DB_PASSWORD, DB_SCHEMA, pipeline_id) under 'env' section. 
        Note that, the pipeline_id you are using in environment variable, should be in database. Otherwise the model will not work or you will not get any alert.

        o. Configure the hostPaths of the volume named 'image-output' under 'volumes' section. Give a path of your host machine, where you want the alert images to be saved.
    
    B. Configure 'alert_schedular.yaml':

        o. Configure the values of the environment variables (database_host, database_port, database_user, database_pass, database_schema)

4. Deploy services:

    A. Pull the below images from DockerHub:
        o. ispeck1/windriver_esentinel_frontend:v1
        o. ispeck1/esentinel_windriver_backend:v1
        o. ispeck1/windriver_streaming_backend:v1
        o. ispeck1/windriver_pc:v1
        o. ispeck1/windriver_ppe:v1
        o. ispeck1/windriver_intrution:v1
        o. ispeck1/windriver-alertschedular:v1
    
    B. Create a JSON file named 'appsettings.json' with the content below.
        Configure the "ConnectionStrings", "Auth" and "SMTPServer" with your credentials:
            {
                "Logging": {
                    "LogLevel": {
                    "Default": "Information",
                    "Microsoft": "Warning",
                    "Microsoft.Hosting.Lifetime": "Information"
                    }
                },
                "AllowedHosts": "*",
                "ConnectionStrings": {
                    "Default": "server=127.0.0.1;port=3306;database=schema;uid=username;pwd=password;persist security info=False;sslMode=None;"
                },
                "Auth": {
                    "Key": "key",
                    "SysadminHash": "SysadminHash"
                },
                "SMTPServer": {
                    "DisplayName": "CRL Notification",
                    "Address": "smtp.gmail.com",
                    "Email": "email_id",
                    "Username": "username",
                    "Password": "password",
                    "Port": 587,
                    "UsingSSL": true
                },
                "StreamingServerURL": "/stream-ws",
                "ImageRootPath": "/app/Images/",
                "FaceImageRootPath": "/app/FaceCrop"
            }

    C. Apply the YAML File 'kubevirt-deployment.yaml'.
        Run the following command to deploy all three virtual machines and the ConfigMap:
            kubectl apply -f kubevirt-deployment.yaml

    D. Verify the VM Deployment
        Use command:
            kubectl get vms