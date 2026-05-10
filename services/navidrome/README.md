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













