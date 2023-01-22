# Initialization

All the services inside this section will be implemented on *Docker*. Since we'll use *Docker Compose* to execute the services, we'll create a folder on the main user's home folder dedicated to analytics servers.

```bash
mkdir ~/analytics
```

For each service created, there will be a subfolder where a `docker-compose.yml` file will be located, alongside any data volumes required and even a `Dockerfile` if required.
