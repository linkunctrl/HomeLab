 
# Initialize the container to generate the directory structure:
```Bash
docker compose up -d
```

2. Configuration Setup

Navigate to your config directory and ensure all necessary files exist. Use the sample files provided by the container as a template.
--- 
# Create essential files if they weren't auto-generated
touch settings.yaml services.yaml widgets.yaml bookmarks.yaml docker.yaml

3. Customization

Edit the following YAML files according to your needs. Use the generated sample files in the repo for syntax reference.

    services.yaml: Define your homelab services (e.g., Navidrome, Vaultwarden).

    widgets.yaml: Set up system info widgets (e.g., CPU, RAM, Weather).

    bookmarks.yaml: Add frequent links.

    settings.yaml: Customize the dashboard title and layout.

    docker.yaml: Configure the Docker socket connection for real-time container stats.

4. Finalize

Apply your configuration changes:
```Bash

docker compose up -d
```

Access the dashboard at: http://localhost:3000
Troubleshooting

    Check logs for YAML syntax errors: docker compose logs -f

    Ensure file permissions allow the container to read the config folder.
