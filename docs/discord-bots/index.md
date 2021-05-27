# Initialization

All the bots inside this section will be implemented on *Docker*. Since we'll use *Docker Compose* to execute the bots, we'll create a folder on the main user's home folder dedicated to Discord bots.

```bash
mkdir ~/discord
```

For each bot created, there will be a subfolder where a `docker-compose.yml` file will be located, alongside any data volumes required and even `Dockerfile` if required.
