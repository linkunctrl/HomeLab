# Navidrome with remote access 

---
### docker-compose.yml for Navidrome:
```bash
services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    restart: unless-stopped

    ports:
      - "4533:4533"

    volumes:
      - ./data:/data
      - /path/to/Music:/music:ro

    environment:
      ND_BASEURL: ""
      ND_PORT: 4533
      ND_LOGLEVEL: info

    #networks:
     # - caddy_network


#networks:
 # caddy_network:
  #  external: true

```

---
### Set up Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
tailscale ip
```

---
### Set up MusicBrainz Picard on Arch based distros 
	[OPTIONAL]

```bash
sudo pacman -S picard 
```

---
### Login to your navidrome clients using tailscale:

1. Server Setup

   - Install Tailscale: Ensure Tailscale is running on the machine hosting Navidrome.

   - Expose Port: Use tailscale serve --bg 4533 to generate a secure HTTPS URL (e.g., [https://your-machine.tailnet.ts.net](https://your-machine.tailnet.ts.net)).

   - Create Users: In the Navidrome web UI, go to Settings > Users and create unique credentials for yourself and your friends.

2. Adding Personal Devices

   - Install App: Download Tailscale on your mobile device or secondary PC.

   - Authenticate: Log in to Tailscale using your primary account credentials.

   - Connect: Use your generated Tailscale URL or IP to log in via a client (e.g., Aonsoku, Feishin, or Symfonium).

3. Adding Friends

   - Invite to Tailscale: Add the friend as a user in the Tailscale Admin Console.

   - Friend Login: Ensure the friend logs into the Tailscale app on their device using their own invited account.

   - Share Credentials: Send the friend the Navidrome URL and the specific username/password you created for them in Step 1.

4. Connection Details

   - Server URL: [https://your-machine.tailnet.ts.net](https://your-machine.tailnet.ts.net) or [http://[Tailscale-IP]:4533](http://[Tailscale-IP]:4533).

   - Client Login: Enter the URL, Username, and Password into the music client app.

   - Troubleshooting: If the connection fails, verify the server’s firewall allows port 4533 and that both parties are active on the same Tailnet.
	```bash
	sudo firewall-cmd --add-port=4533/tcp --permanent
	sudo firewall-cmd --reload

	or

	sudo ufw allow 4533/tcp
	```
---

### Scrobbling and Automation with Last.fm, Soundiiz

#### Enable Scrobbling:
- Connect Last.fm in the personal section of the Navidrome Web UI.

- Use Soundiiz to connect your music services to Last.fm.

#### Tailscale Funnel Configuration

- In Tailscale, enable MagicDNS and HTTPS certificates.
	
- Add the following to your Access Control JSON to enable Funnel
	```
	"nodeAttrs": [
    {
        "target": ["autogroup:member"],
        "attr": ["funnel"]
    }
	]	
	```
		
Run the command to expose the Navidrome port:
``` Bash

tailscale funnel --bg 4533
```
Check the status:
```Bash
tailscale funnel status
```

This provides an HTTPS URL: https://devicename.yourgivensitename.ts.net.

 #### [Lidarr might have trouble with Indexers]
	Lidarr Setup on Arch-based Distros

        Use the URL from the previous step to connect Navidrome to Soundiiz (this creates a metadata playlist for Lidarr).

        Install and enable Lidarr:
        Bash

        yay -S lidarr-bin
        sudo systemctl daemon-reload
        sudo systemctl enable --now lidarr

        Access the UI at http://localhost:8686.

    Lidarr Configuration and Permissions

        Import Playlists: Go to Settings > Import Playlists > Last.fm User.

        Add your Last.fm username, select "Top Albums," and set the music root folder path.

        If permission issues occur, edit the service:
        Bash

        sudo systemctl edit lidarr.service

        Paste this between or above the comments:
        Ini, TOML

        [Service]
        ProtectHome=false
        ReadWritePaths=/path/to/Music

    Indexers and Download Clients

        Pick a torrent indexer in Settings > Indexers.

        qBittorrent Setup:

            In qBittorrent, go to Preferences > Web UI > Enable Web UI.

            Set IP to 127.0.0.1 and port to 8080.

            Set a password.

            Use these details to enable the download client within Lidarr.

    Remote Access for Others

        Ensure Tailscale is running on the Navidrome server with Funnel access enabled. This allows friends and family to access the server from different networks easily. Don't forget to add them on your tailscale network.
       











