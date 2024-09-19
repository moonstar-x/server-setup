# Initialization

All the services inside this section will be run through on *Docker*. Since we'll use *Docker Compose* to execute the services, we'll create a folder on the main user's home folder dedicated to downloads services.

```bash
mkdir ~/services/downloads
```

For each service created, there will be a subfolder where a `docker-compose.yml` file will be located, alongside any data volumes required and even a `Dockerfile` if required.
