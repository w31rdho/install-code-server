If you want to set up **code-server** on Ubuntu Server 24.04 without using SSL (i.e., without HTTPS), you can still run the service on **HTTP**. However, note that this may expose your connection to security risks, especially for remote access. If security is important, you should consider enabling SSL later on.

### Updated Tutorial for Installing **code-server** without SSL on Ubuntu 24.04 LTS

---

## Prerequisites:
- Ubuntu 24.04 LTS server
- A **non-root** user with **sudo** privileges
- A firewall configured to allow access on **port 8080** (code-server's default port)

---

## Step 1: Update your system
Start by ensuring your system is up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2: Install `code-server`
1. **Install the required dependencies**:
   ```bash
   sudo apt install -y curl wget gnupg
   ```

2. **Download and install `code-server`**:
   - First, add the GPG key and repository:
     ```bash
     curl -fsSL https://code-server.dev/install.sh | sh
     ```

   - This will install `code-server` and set it up automatically on your system.

---

## Step 3: Start `code-server` service
1. **Start `code-server`**:
   - By default, the `code-server` package creates a systemd service, which can be started using:
     ```bash
     sudo systemctl start code-server@$USER
     ```

2. **Enable `code-server` to start on boot**:
   ```bash
   sudo systemctl enable code-server@$USER
   ```

3. **Check the status** of `code-server`:
   ```bash
   sudo systemctl status code-server@$USER
   ```

   You should see an output indicating that the service is running. For example:
   ```
   Active: active (running) since Mon 2024-11-11 22:42:49 UTC; 1min 3s ago
   ```

---

## Step 4: Configure `code-server` to Use HTTP (without SSL)
1. **Modify the `code-server` configuration file** to disable HTTPS and bind it to the desired address:
   ```bash
   nano ~/.config/code-server/config.yaml
   ```

   - Change or add the following lines:
     ```yaml
     bind-addr: 0.0.0.0:8080  # Listen on all interfaces and port 8080
     auth: password           # Enable password authentication
     password: your-password  # Set your password here
     cert: false              # Disable SSL
     ```

   - Save the file and exit the editor (Ctrl + X, then Y to confirm, and Enter).

2. **Restart the `code-server` service** for the changes to take effect:
   ```bash
   sudo systemctl restart code-server@$USER
   ```

---

## Step 5: Set Up Nginx (Optional Reverse Proxy)
You can use **Nginx** as a reverse proxy to forward requests from port 80 (HTTP) to port 8080 (where `code-server` is running). This allows you to access `code-server` through your domain or IP on port 80.

1. **Install Nginx**:
   ```bash
   sudo apt install nginx
   ```

2. **Configure Nginx**:
   Create a configuration file for `code-server`:
   ```bash
   sudo nano /etc/nginx/sites-available/code-server
   ```

   Add the following configuration:
   ```nginx
   server {
       listen 80;
       server_name your-ip;  # Your server's IP or domain

       location / {
           proxy_pass http://127.0.0.1:8080;  # Forward traffic to code-server
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;

           # WebSocket support (important for code-server)
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
       }
   }
   ```

3. **Enable the site configuration**:
   Create a symbolic link from `sites-available` to `sites-enabled`:
   ```bash
   sudo ln -s /etc/nginx/sites-available/code-server /etc/nginx/sites-enabled/
   ```

4. **Test the Nginx configuration**:
   ```bash
   sudo nginx -t
   ```

   If the test is successful, reload Nginx:
   ```bash
   sudo systemctl reload nginx
   ```

---

## Step 6: Open Ports in Firewall
Make sure that ports 80 (HTTP) and 8080 (for `code-server`) are open in your firewall.

For **UFW** (Uncomplicated Firewall), run:
```bash
sudo ufw allow 80/tcp    # Allow HTTP
sudo ufw allow 8080/tcp  # Allow code-server's internal port
sudo ufw enable          # Enable the firewall (if not already enabled)
```

---

## Step 7: Access `code-server`
You should now be able to access `code-server` by navigating to your **IP address** or **domain** in your web browser:
```
http:/your-ip:8080
```

If you configured **Nginx** to reverse proxy, use just the domain or IP address:
```
http://your-ip
```

Enter the **password** you set in the configuration file to access the `code-server` interface.

---
