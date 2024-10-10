1. **Create the Golang Application:**
    
    Let's assume you have a compiled Golang application named `myapp` located in `/usr/local/bin/myapp`.
    
2. **Create a Systemd Service Unit File:**
    
    Create a service unit file for your Golang application. You typically place this file in `/etc/systemd/system/`.
    
    ```bash
    sudo nano /etc/systemd/system/myapp.service
    ```
    
3. **Edit the Service Unit File:**
    
    Add the following content to the `myapp.service` file:
    
    ```
    [Unit]
    Description=My Golang Application
    After=network.target
    
    [Service]
    ExecStart=/usr/local/bin/myapp
    Restart=always
    RestartSec=5
    User=your_user
    Group=your_group
    
    [Install]
    WantedBy=multi-user.target
    ```
    
    **Explanation:**
    
    - **[Unit] Section:**
        - `Description`: A brief description of the service.
        - `After`: Ensures the service starts after the network is up.
    - **[Service] Section:**
        - `ExecStart`: The command to start your Golang application.
        - `Restart=always`: Automatically restarts the service if it stops unexpectedly.
        - `RestartSec=5`: Waits 5 seconds before attempting to restart the service.
        - `User` and `Group`: Specifies the user and group under which the service runs (replace `your_user` and `your_group` with appropriate values).
    - **[Install] Section:**
        - `WantedBy=multi-user.target`: Ensures the service starts in multi-user mode (typical for most services).
4. **Reload Systemd Configuration:**
    
    After creating the service file, reload the systemd configuration to apply the changes:
    
    ```bash
    
    sudo systemctl daemon-reload
    ```
    

**Enable the Service to Start at Boot:**

Enable the service so that it starts automatically at boot:

```bash
sudo systemctl enable myapp
```

1. **Start the Service:**
    
    Start the service:
    
    ```bash
    sudo systemctl start myapp
    ```
    
2. **Check the Status of the Service:**
    
    Verify that the service is running correctly:
    
    ```bash
    bashCopy code
    sudo systemctl status myapp
    ```
    
    Example output:
    
    ```yaml
    ● myapp.service - My Golang Application
       Loaded: loaded (/etc/systemd/system/myapp.service; enabled; vendor preset: enabled)
       Active: active (running) since Tue 2023-04-04 12:34:56 UTC; 1min ago
     Main PID: 1234 (myapp)
        Tasks: 5 (limit: 1152)
       CGroup: /system.slice/myapp.service
               └─1234 /usr/local/bin/myapp
    
    ```
    

### Handling Logs:

Systemd captures the stdout and stderr output of your service, which can be viewed using `journalctl`:

```bash
sudo journalctl -u myapp
```

This will show you the logs for your Golang application, which is helpful for debugging and monitoring purposes.

### Summary:

By following these steps, you can ensure your Golang application runs as a systemd service and automatically restarts if it stops unexpectedly. This setup enhances the reliability and availability of your application.
