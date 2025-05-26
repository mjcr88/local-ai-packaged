Outline Documentation


Requirements
There are a few minimum services and dependencies that are required for for the Outline software to function correctly.
#Operating System
Outline should run on any Unix operating system that supports the software dependencies listed below. The docker image is based on Alpine Linux and operators have successfully deployed Outline on Ubuntu, Debian, and similar.
Outline is developed Unix-compatible operating systems only. It does not run on Microsoft Windows server, for instance.
#Supporting Software
PostgreSQL (v12+)
Redis (v4+)
#Hardware
#CPU
Outline will run on relatively low powered hardware. CPU requirements are dependent on the number of users and expected workload. Your workload is influenced by factors such as - but not limited to - how many users are online, how active your users are, whether they are viewing and typing in documents.
#Memory
A minimum of 512MB of memory is required, depending on the number of users 1GB or more is recommended.
#Other Dependencies
A dedicated domain or sub-domain to host Outline on (e.g. docs.yourcompany.com)
A compatible Authentication Provider
Installation methods
Outline can be installed in most GNU/Linux distributions, and within cloud providers such as AWS. We also offer a hosted version of Outline with a one-click signup at getoutline.com which enables you to get started in minutes.
#Requirements
Before you install Outline, be sure to review the system requirements which includes details about the minimum hardware, software, and database setup.
#Choose Installation
Method
Description
When to use
Docker
The dockerized application image
This is the recommend and supported way for self-hosting Outline.
From source
Build from scratch
If you are developing Outline or need to run on a platform that isn’t support by the official Docker images.
Terraform
Community maintained
Use to deploy on GCP. This method is community maintained.
Hosted
Outline hosts your install in the cloud
If you do not have the technical expertise or requirement to run your own installation and would like to get started in minutes.

Outline in the cloud is updated daily with the latest features and fixes and has an outstanding uptime record of
99.99%+.




Docker
Docker is the recommended way to run Outline – updated Docker images based on Alpine Linux are available on a monthly release cadence and are tested by the maintaining team.
#Configuration
Using this sample file as a reference create a docker.env file to contain the environment variables for your installation.
Be sure to include all of the values in the Required section and at least one authentication provider from the Optional section to have the application successfully start.
#Docker Compose
It is recommended to use Docker Compose to manage the various docker containers, if your Postgres, Redis, and storage are running in the cloud then you may skip this step and run the single Outline docker container directly.
Install Docker Compose
Create a docker-compose.yml file, an example configuration with all dependencies dockerized and environment variables kept in docker.env is as follows. If you are installing the enterprise edition the image name should be replaced with outlinewiki/outline-enterprise.

Note: It is recommended to pin the version of all images rather than relying on the latest tag so that you can remain in control of upgrades, e.g. image: outlinewiki/outline:0.82.0 , image: postgres:16
services:
  outline:
    image: docker.getoutline.com/outlinewiki/outline:latest
    env_file: ./docker.env
    expose:
      - "3000"
    volumes:
      - storage-data:/var/lib/outline/data
    depends_on:
      - postgres
      - redis

  redis:
    image: redis
    env_file: ./docker.env
    expose:
      - "6379"
    volumes:
      - ./redis.conf:/redis.conf
    command: ["redis-server", "/redis.conf"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 30s
      retries: 3

  postgres:
    image: postgres
    env_file: ./docker.env
    expose:
      - "5432"
    volumes:
      - database-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-d", "outline", "-U", "user"]
      interval: 30s
      timeout: 20s
      retries: 3
    environment:
      POSTGRES_USER: 'user'
      POSTGRES_PASSWORD: 'pass'
      POSTGRES_DB: 'outline'

  https-portal:
    image: steveltn/https-portal
    env_file: ./docker.env
    ports:
      - '80:80'
      - '443:443'
    links:
      - outline
    restart: always
    volumes:
      - https-portal-data:/var/lib/https-portal
    healthcheck:
      test: ["CMD", "service", "nginx", "status"]
      interval: 30s
      timeout: 20s
      retries: 3
    environment:
      DOMAINS: 'docs.mycompany.com -> http://outline:3000'
      STAGE: 'production'
      WEBSOCKET: 'true'
      CLIENT_MAX_BODY_SIZE: '0'

volumes:
  https-portal-data:
  storage-data:
  database-data:
#Database
Create the database with the following command:
docker compose run --rm outline yarn db:create --env=production-ssl-disabled
Migrate the new database to add needed tables, indexes, etc:
docker compose run --rm outline yarn db:migrate --env=production-ssl-disabled
If you setup your Redis or Postgres outside the docker container you could use additional flag --net="host" for docker. In this case you will be able to use localhost as host in your docker.env file for Redis and Postgres.
#Running
Make sure you are in the same directory as docker-compose.yml and start Outline:
docker compose up -d
Check the output carefully for errors such as connecting to your database or redis instance. To make your installation accessible to the outside world look at the SSL configuration documentation.
#Updating
To update Outline, follow these steps:
Backup the database
Update to the latest image
Use docker pull docker.getoutline.com/outlinewiki/outline:latest to download the image or
If you are using docker-compose with a fixed version number then you will need to update the docker-compose.yml file to point to the latest version.
Run the container
#Migrations
Note that as of v0.69.0 database migrations are ran automatically when changing image.
Migrations are ran automatically when the container starts up. If you want to prevent this behavior and run migrations manually pass the --no-migrate flag.


Authentication
Outline can be configured to accept a variety of SSO, OIDC, and SAML authentication options depending on the edition in use.


OIDC
Outline supports all OIDC-compatible authentication providers out of the box. Currently the OIDC configuration must be provided manually as environment variables; the following values are required:
OIDC_CLIENT_ID – OAuth client ID
OIDC_CLIENT_SECRET – OAuth client secret
OIDC_AUTH_URI
OIDC_TOKEN_URI
OIDC_USERINFO_URI
Note, one of the following should be set or it will be very difficult for users to logout correctly:
OIDC_DISABLE_REDIRECT – If set, will prevent Outline from automatically redirecting to the OIDC provider
OIDC_LOGOUT_URI – A url that the client will be redirect to post-logout
The following configuration is optional:
OIDC_USERNAME_CLAIM – Supports any valid JSON path with the JWT payload (preferred_username by default)
OIDC_DISPLAY_NAME – The text that should be displayed on the login button (OpenID by default)
OIDC_SCOPES – The scopes to request (openid profile email by default)
#Examples
OIDC providers will often publish the correct values for these at the “.well-known” url, some examples that could be used to sign-in to Outline using this method include:
GitLab: https://gitlab.com/.well-known/openid-configuration
Apple: https://appleid.apple.com/.well-known/openid-configuration
Twitch: https://id.twitch.tv/oauth2/.well-known/openid-configuration
Facebook: https://www.facebook.com/.well-known/openid-configuration
Keycloak: https://www.keycloak.org/ (self-hosted)
Mattermost: https://developers.mattermost.com/integrate/admin-guide/admin-oauth2/#oauth-endpoints (self-hosted and cloud)
Authelia: https://www.authelia.com/integration/openid-connect/outline/ (self-hosted)
Authentik: https://docs.goauthentik.io/integrations/services/outline/ (self-hosted)




Integrations
Outline comes with many integrations that work out-of-the-box, however some can be optionally configured, find details on them within this section.




GitHub
This integration is available in all editions from v0.76.0 onwards
The GitHub integration allows previews of issues and pull requests in GitHub within Outline. For example, hovering over a GitHub url will show a preview of the issue status at that url.
#Setup
Register a new “GitHub App”, you’ll need to fill out the following fields:
Name – “Outline” might make sense
URL – It is advisable to choose the address of your Outline installation
Callback URL – Use <yourdomain.com>/api/github.callback
“Request user authorization (OAuth) during installation” – Make sure this option is checked
“Webhooks” – This option can be disabled
On the GitHub app settings make a note of the last part of the “Public link” field, this is the “app name”.
Generate a client secret and make a note of the value.
Still on the GitHub app settings generate a private key, the file will be saved to your machine.
Base64 encode the private key using the following command, where <infile> is the path to the private key that was just downloaded. Make a note of the value in the out file.
openssl base64 -in <infile> -out <outfile>


Add the configuration to the Outline environment, using the following variable names:
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
GITHUB_APP_NAME=
GITHUB_APP_ID=
GITHUB_APP_PRIVATE_KEY=
Restart the server, you should see “GitHub” available under Settings → Integrations



Notion
This importer is available in all editions from v0.83.0 onwards
The Notion integration allows importing documents from Notion into Outline – directly converting to our internal format bringing across document content, images, and videos for one or more pages in your Teamspaces.
#Setup
Register a new “Notion app”
Integration name – “Outline” might make sense
Associated workspace – choose the workspace you want to import from
Type – Change to “Public”
Redirect URIs – Use <yourdomain.com>/api/notion.callback
The other required fields should be filled out but do not matter.
Click “Save”
Copy the “OAuth Client ID” and “OAuth Client Secret” on the next page.
Add the configuration to the Outline environment, using the following variable names:
NOTION_CLIENT_ID=
NOTION_CLIENT_SECRET=
Restart the server
Under “Settings → Import → Notion” you will now be able to choose “Import”




OpenAI
Note: This integration is available in the Business + Enterprise editions from v0.75.1 onwards

AI answers in Outline
The OpenAI integration will allow for various AI functionality around the Outline workspace. Today, it enables “AI answers” to questions by semantically indexing your knowledge, retrieving related documents in realtime, and generating direct answers to questions using their content.
#Setup
Ensure you have the outline-enterprise image installed and licensed
Sign up for an OpenAI account, create an API key and add it to the server environment under OPENAI_API_KEY.
Optionally configure OPENAI_URL to point to an alternative compatible API endpoint, but note that “compatible” providers have not been tested. By default the public OpenAI API will be used.
You will need a database with the pgvector extension installed to store the search index. This can be a new database, or the same database you use for other Outline data. If you’re using a separate database then an additional DATABASE_URL_VECTOR environment config should be provided to tell Outline where to find this database.
Restart the application server, Under Settings → Features → AI Answers you can now enable the integration which will automatically begin indexing content.
#Limitations
AI answers are limited to information contained in documents that the logged in user has access to.
The initial indexing may take a few seconds or a few minutes depending on your OpenAI rate limit and the number of documents in your knowledgebase.
Content inside attachments will not be indexed.




PDF export
Note: This integration is only available in the Business + Enterprise editions
PDF exporting is supported through an integration with Gotenberg. Once setup you’ll see an additional Download as → PDF option available in Outline for all documents.
#Setup
Ensure you have the Business or Enterprise edition installed and licensed
Run a copy of Gotenberg 7+ in your infrastructure, this can be as simple as using the public Docker image gotenberg/gotenberg – no additional configuration is required.
Add GOTENBERG_URL to your Outline environment pointing to the fully qualified url of the instance. Note that the Gotenberg instance does not need to be accessible to the public internet, but must be accessible to the server running Outline.
Restart Outline
#Notes
There is a demo instance available at https://demo.gotenberg.dev
Currently PDF generation is per-document. We may further integrate to allow export of entire collections or nested documents to PDF in the future.



SSL
Outline should always be exposed over HTTPS whether over the wider internet or your internal network. Several methods for achieving this are supported:
#Reverse Proxy
If you do not have your own certificate and would like to have one generated automatically via LetsEncrypt then we recommend using the https-portal docker image. If you’re following the instructions from our on-prem repository then this will be provisioned automatically, as long as your server is accessible to the internet at the url specified during the installation process then there is nothing else to do.
You can also setup your own reverse proxy with nginx, for Outline to work correctly you must ensure that websockets are forwarded through the proxy. This type of advanced deployment is outside the scope of this documentation.
The following options are available in all editions from v0.62.0 onwards
#Certificate (Environment Variables)
If you have your own certificate then it can be provided via environment variables. To pass as an environment variable your key and certifica`te must be base64 encoded. You can use the following command to do so:
openssl base64 -in <infile> -out <outfile>
Then use the base64 encoded values from the generated files for the private key and public certificate with SSL_KEY and SSL_CERT variables respectively.
#Certificate (File System)
If you have your own certificate then it can also be provided via the file system. Outline looks for the files private.pem and public.pem in the root of the installation. When using the Docker image you will need to mount these files into the container. Using docker compose the following syntax would be correct where private.pem and public.pem are the relative location on your host system.
outline:
  image: outlinewiki/outline
  ports:
    - "443:443"
    - "80:80"
  volumes:
    - type: bind
      source: ./private.pem
      target: /opt/outline/private.pem
      read_only: true
    - type: bind
      source: ./public.pem
      target: /opt/outline/public.pem
      read_only: true
You’ll also need to update your environment variables to tell Outline to use the SSL port:
PORT=443


Nginx
Using Nginx is not required. If you would like to use it as a reverse proxy then the following is an example configuration that is known to work with Outline and it’s websockets setup.
This example does not include configuring SSL with Nginx, however once setup you can set FORCE_HTTPS=true in Outline’s .env configuration and any http requests will be automatically forwarded to https.
server {
  server_name docs.mycompany.com;
  listen 80;
  listen 443 ssl;

  # SSL certificate configuration is not included in this example

  location / {
    proxy_pass http://localhost:3000/;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "Upgrade";
    proxy_set_header Host $host;

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Scheme $scheme;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off;
  }
}
 Outline




Redis
#Simple server
A single Redis server is recommended, in self-hosted setups anything more is overkill. You can configure this through the environment variable REDIS_URLwhich should point to the fully qualified url of the Redis installation, e.g: redis://redis:6379
#Advanced
If your Redis connection requires more configuration it can be provided through a base64 encoded JSON connection options object. Using this method you can configure multiple hosts, for example:
{"sentinels": [{"host": "sentinel-0","port":26379}, {"host": "sentinel-1","port":26379}],"name": "mymaster"}
Can be base64 encoded as the following config:
REDIS_URL=ioredis://eyJzZW50aW5lbHMiOlt7Imhvc3QiOiJzZW50aW5lbC0wIiwicG9ydCI6MjYzNzl9LHsiaG9zdCI6InNlbnRpbmVsLTEiLCJwb3J0IjoyNjM3OX1dLCJuYW1lIjoibXltYXN0ZXIifQ==
Available options are documented in ioredis.



File storage
#Local file system
The following options are available from v0.72.0 onwards
If you would like to store file uploads on the same server that Outline is running from then you can do so using the local file system storage option.
#Environment Variables
Set the following environment variables.
FILE_STORAGE=local
FILE_STORAGE_UPLOAD_MAX_SIZE=26214400
FILE_STORAGE_IMPORT_MAX_SIZE=
FILE_STORAGE_WORKSPACE_IMPORT_MAX_SIZE=
When running in docker, files are stored inside the container by default (at /var/lib/outline/data), in order for files to be persisted to the host file system a mapping needs to exist in the docker-compose.yml. You will find that the configuration is very similar to Postgres, the following two options are available – docker volumes are preferred for simplicity and portability:
#Docker Volume
To store the files in a docker volume use the following mapping style:
   volumes:
      - storage-data:/var/lib/outline/data
#File system folder
To store the files in a location on your local file system using a bind mount. This is particularly useful when migrating from previously using Minio, as you can point the app at where the files were previously stored and everything will continue to work.
   volumes:
      - /location/on/host/filesystem:/var/lib/outline/data
If you get permissions errors writing files, ensure that Outline has permission to write to the directory, by giving the user ID access: chown 1001 /location/on/host/filesystem
#AWS S3
This guide assumes that you’ve created a bucket in Amazon S3 region us-east-2 called my-bucket-name . If your bucket is named differently or in another region update the values appropriately. Since September 2020 all new buckets in S3 must be accessed via the “VirtualHost” method where the bucket and region are contained within the hostname, the config below reflects this change.
#Environment Variables
FILE_STORAGE=s3
AWS_REGION=us-east-2
AWS_S3_FORCE_PATH_STYLE=false
AWS_S3_UPLOAD_BUCKET_NAME=my-bucket-name
AWS_S3_UPLOAD_BUCKET_URL=https://my-bucket-name.s3.us-east-2.amazonaws.com
AWS_S3_UPLOAD_MAX_SIZE=26214400
AWS_ACCESS_KEY_ID=AK66L9HZpTtfrpFFgtVxcxOUTn
AWS_SECRET_ACCESS_KEY=97L9HfssZpTtfrxOUTVxcnpgtSa
#Permissions
Ensure that “Block public access” is set to “off” at the bucket level.
#Cross-origin resource sharing (CORS)
Under your bucket permissions add the following policy json and update the AllowedOrigins to match your installation url.
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "PUT",
            "POST"
        ],
        "AllowedOrigins": [
            "https://docs.mycompany.com"
        ],
        "ExposeHeaders": []
    },
    {
        "AllowedHeaders": [],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": []
    }
]
#IAM Policy
The following actions must to be allowed, It is recommended to create an IAM user with only this policy attached but you may add to another user, then generate an access key and secret for the user to be included in the environment variables.
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor",
            "Effect": "Allow",
            "Action": [
                "s3:GetObjectAcl",
                "s3:DeleteObject",
                "s3:PutObject",
                "s3:GetObject",
                "s3:PutObjectAcl"
            ],
            "Resource": "arn:aws:s3:::my-bucket-name/*"
        }
    ]
}
#Transfer Acceleration
Note: This configuration is only available from v0.63.0 onwards
S3 offers “transfer acceleration” as an optional feature which takes advantage of Cloudfront’s CDN to improve performance of uploads and downloads. If enabled for your bucket enter the url provided by AWS in the AWS_S3_ACCELERATE_URL environment variable.
#S3-compatible services
Outline can be used with the vast majority of S3-compatible API’s as a very small subset of the available API interface is used, the following have been tested by members of the community.
Service
Compatible
Amazon S3
✅
Minio
✅
DigitalOcean Object Storage
✅
Alibaba Cloud / Aliyun OSS
✅ (discussion)
Scaleway
✅ (discussion)
Cloudflare R2
❌ (discussion)
Backblaze
❌ (discussion)
OVH Object Storage
❌




SMTP
To support sending outgoing transactional emails such as "document updated" or "you've been invited" you'll need to provide authentication for an SMTP server. This is also required to support email “magic link” sign-in. Environment variables are mostly self explanatory – add the following to your environment:
#Configuration
SMTP_HOST – The hostname of your mail server, e.g. mail.mycompany.com
SMTP_PORT – The port of your mail server, usually 465
SMTP_USERNAME – An optional username for secured mail servers
SMTP_PASSWORD – An optional password for secured mail servers
SMTP_FROM_EMAIL – You can also use the mailbox format, Outline <noreply@mycompany.com>
SMTP_REPLY_EMAIL – Optional. You can also use the mailbox format, Outline <reply@mycompany.com>
SMTP_TLS_CIPHERS – Optional. Allows passing the ciphers to use with TLS
SMTP_SECURE – Optional. If false the connection to the mail server will no use TLS, defaults to true
SMTP_NAME – Optional. hostname to send to the server, sometimes required, eg for Google Mail. (Available from
0.67.2 onwards)
#Examples
Some known working configurations can be found below
#Apple Mail
SMTP_HOST=smtp.mail.me.com
SMTP_PORT=587
SMTP_USERNAME=<username>
SMTP_PASSWORD=<password>
SMTP_FROM_EMAIL=<email>
SMTP_TLS_CIPHERS=TLSv1.2
SMTP_SECURE=false
#Mailgun
SMTP_HOST=smtp.mailgun.org
SMTP_PORT=465
SMTP_USERNAME=<username>
SMTP_PASSWORD=<password>
SMTP_FROM_EMAIL=<email>
Contents
Configuration
Examples
Apple Mail
Mailgun
 Outline





Branding
#Logo and colors
From v0.69.0 onwards there is the option to change the accent color of the application from the default blue. This enables closer matching to your team’s branding. You can also upload a logo under Settings → Workspace → Details.
#Public branding
Under Settings → Features → Public branding you can enable the option to show the branding on public pages such as login, this will show both any accent color that has been chosen and the logo uploaded under Settings → Workspace → Details.
#Application name
Note: This functionality is only available in the Enterprise Edition
With the Enterprise installation it is possible to change the name of the application from “Outline” to something else entirely by setting the APP_NAME environment variables.
The chosen name will be used in all places around the UI, notification emails, etc.


Rate limiter
The following options are available from v0.66.0 onwards
Outline includes an optional rate limiter that can be enabled for increased protection against outside brute-force attacks if your installation is facing the public internet.
Once enabled, the rate limiter has sensible defaults for sensitive endpoints such as mutations and also includes a global IP-based rate limiter that applies to all endpoints and can be configured manually with the following environment variables:
#Configuration
# To enable the limiter set to true (default disabled)
RATE_LIMITER_ENABLED=true

# The window duration in seconds – it is recommended to leave this at 60.
RATE_LIMITER_DURATION_WINDOW=60

# The number of requests per-IP in the set window. In this example 1000 API requests/minute/ip.
RATE_LIMITER_REQUESTS=1000


Scheduled jobs
There are a number of pieces of functionality in Outline that rely on scheduled jobs being ran on a daily basis. Cleanup of temporary files, sending email reminders, and removing items from the trash being just a few.
#Setup
In order to have these be triggered you should setup a cron job that triggers the utility endpoint with the token defined in UTILS_SECRET environment variable once per day. The request should look like:
https://docs.mycompany.com/api/cron.daily?token=<UTILS_SECRET>


Self-signed certificates
If any of your services use self-signed certificates for SSL such as Minio, or the authentication provider then the Outline docker container needs to be made aware of the root certificate authority in order to verify the certificate. You can do this by providing the certificate at runtime or by bundling it with a custom docker image.
#Runtime configuration
To have Outline trust your self-signed root certificate, you must bind mount it to the container and inform Node where to find it. For example, in your docker-compose.yml file, bind mount the CA certificate file in the volumes section, e.g:


 outline:
    image: docker.getoutline.com/outlinewiki/outline:latest
    env_file: ./docker.env
    ports:
      - "3000:3000"
    volumes:
      - storage-data:/var/lib/outline/data
      - /etc/ssl/myca.crt:/etc/ssl/myca.crt:ro
    depends_on:
      - postgres
      - redis
Then, add NODE_EXTRA_CA_CERTS to your environment, with the path to the CA cert:
NODE_EXTRA_CA_CERTS=/etc/ssl/myca.crt
#Custom docker image
To bundle your self-signed root certificate you can build your own Docker image based on the Outline image. The Dockerfile would look similar to the following – note that update-ca-certificates must be installed and ran to install the certificate in the correct location.
FROM outlinewiki/outline:latest

USER root

COPY ./myCA.pem /usr/local/share/ca-certificates
RUN update-ca-certificates
With this Dockerfile you can create your own image by running the following command in the same folder:
docker build . --tag outline-custom:latest
Tell Node/Outline to use the custom certificates with the following environment variable:
NODE_OPTIONS=--use-openssl-ca
#Disable verification
Instead, you can disable verification of all TLS certificates by setting the following environment variable. Note that this isn’t advised as it opens your installation up to man-in-the-middle attacks, you may decide this is an acceptable risk depending on your hosting environment.
NODE_TLS_REJECT_UNAUTHORIZED="0"


Backups
#Database
It is recommended to take regular backups or snapshots of your Postgres database, we recommend daily at a minimum retained for a month for the most safety. You should always take a database backup or server snapshot before upgrading between releases – each release introduces automatic database migrations and changes that are not always backwards compatible.
The specifics of backing up a Postgres database depend on your hosting situation and are not considered in scope for this guide – see the official documentation.
#Environment
If your hosting environment does not provide secret storage and environment variables are stored in a .env file then this file should be backed up after every change, for example to a password manager or other encrypted store. Losing the SECRET_KEY environment variable for example will result in all encrypted data in the database becoming inaccessible.
#Exporting content
Alternatively you can also create backups in HTML, Markdown, or JSON format from Settings → Export. These could also be automated using a script and the associated API endpoints. Documentation for this endpoint can be found here (API documentation).
Contents
Database
Environment
Exporting content
 Outline



Hosting Outline
Troubleshooting
Troubleshooting
To view more logging for troubleshooting set LOG_LEVEL=debug in your environment variables, this will introduce many more logs that may be useful, this will also often be requested if you ask a question on the discussion forums.
#HTTP → HTTPS redirect loop
During setup it can be useful to first get things working on http, if the browser is redirecting to HTTPS then there are two potential culprits:
First set FORCE_HTTPS=false in the .env .
If the redirect continues to happen it’s likely that your browser has cached the HTTPS attempt, follow this guide to reset: https://howchoo.com/chrome/stop-chrome-from-automatically-redirecting-https
#Cannot edit documents
The wiki loads and you can create collections but documents appear to be uneditable. This is likely because your proxy is not allowing websocket connections through – websocket support is a requirement of Outline. See discussions for several ways this has been fixed in the past:
https://github.com/outline/outline/discussions/2222
#Cannot access Postgres / Redis
If using Docker and you setup your Redis or Postgres on the host machine you could use additional flag --net="host" for docker. In this case you will be able to use localhost as host in your .env file for redis and Postgres.
#EADDRINUSE
Something else is already using the port that Outline is trying to run on. You could change the port with the PORT environment variable or find the other service and kill it. It could also be another copy of Outline that is already running.
#Error connecting to IDP with self-signed certificate
Please see the guide to using self signed certificates for details on how to make your root certificate authority available to the Outline container.
#RangeError: Invalid key length
Your SECRET_KEY environment variable is not formatted correctly, follow the instructions in .env.sample carefully to ensure it is in the correct format and length.
#Invalid Request (request has multiple authentication types, please use one) with Minio
The Outline authorization header is likely being passed Minio if it is on the same domain, this is decided by the browser and we have no control over it. The solution is to host Minio on a separate domain which will result in the headers not being forwarded.
#Config could not be parsed
This issue is caused when a third-party modifies the HTML output in a manner that breaks Outline’s security policies and prevents scripts from being read correctly. Cloudflare Rocket Loader is often the culprit – the solution is to disable Cloudflare Rocket Loader, it is not necessary with Outline.
You can disable this feature for the whole site, or use a Configuration Rule to disable it on a specific hostname.
Contents
HTTP → HTTPS redirect loop
Cannot edit documents
Cannot access Postgres / Redis
EADDRINUSE
Error connecting to IDP with self-signed certificate
RangeError: Invalid key length
Invalid Request (request has multiple authentication types, please use one) with Minio
Config could not be parsed
 Outline



