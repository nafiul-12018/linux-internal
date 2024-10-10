### System Metrics

1. **CPU Usage**:
    - Track overall CPU usage and per-core usage to identify any processes consuming excessive CPU resources.
    - Tools: `top`, `htop`, `mpstat`
    
    htop
    
    1  [||||||||||||||||||||||||||||||||||||||||||||||100.0%]   Tasks: 112, 4 thr; 1 running
    2  [||||||||||||||||||||||||||||||||||||||||||||||100.0%]   Load average: 12.34, 11.45, 10.67
    3  [||||||||||||||||||||||||||||||||||||||||||||||100.0%]   Uptime: 1 day, 03:45:23
    4  [||||||||||||||||||||||||||||||||||||||||||||||100.0%]
    Mem[|||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||1.31G/3.84G]
    Swp[|                                             16M/4.00G]
    
    - 
2. **Memory Usage**:
    - Monitor total memory usage, usage per process, and swap usage.
    - Tools: `free`, `vmstat`, `top`
    
    free -h
    
    ```
              total        used        free      shared  buff/cache   available
    
    ```
    
    Mem:          3.8Gi       1.3Gi       1.8Gi       150Mi       754Mi       2.2Gi
    Swap:         4.0Gi        16Mi       4.0Gi
    
3. **Disk Usage**:
    - Check disk space usage and inode usage on all filesystems.
    - Monitor disk I/O to identify any bottlenecks.
    - Tools: `df`, `du`, `iostat`, `iotop`
    
    df -h
    
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda1        50G   15G   33G  32% /
    
4. **Network Usage**:
    - Track incoming and outgoing traffic, packet rates, and error rates.
    - Tools: `iftop`, `nload`, `netstat`, `vnstat`
    
    netstat -tuln
    
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State
    
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
    
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
    
    tcp6       0      0 :::80                   :::*                           LISTEN
    
    tcp6       0      0 :::22                   :::*                           LISTEN
    
    udp        0      0 0.0.0.0:68              0.0.0.0:*
    
    udp        0      0 0.0.0.0:123             0.0.0.0:*
    
    **Analysis**:
    
    - **Port 80 (HTTP)** and **Port 22 (SSH)** are listening, indicating the web server and SSH service are running.
    - **No unexpected foreign addresses** are shown, suggesting no unusual connections.
    
    - **Check Detailed Network Connections**:
    Use `netstat` with additional options to view detailed connection states.
        
        ```bash
        netstat -anp | grep ':80'
        ```
        
        **Output**:
        
        ```bash
        tcp        0      0 192.168.1.2:80         192.168.1.10:45678      ESTABLISHED 1234/apache2
        tcp        0      0 192.168.1.2:80         192.168.1.11:45679      ESTABLISHED 1234/apache2
        tcp        0      0 192.168.1.2:80         192.168.1.12:45680      SYN_RECV    -
        tcp        0    100 192.168.1.2:80         192.168.1.13:45681      ESTABLISHED 1234/apache2
        tcp        0      0 192.168.1.2:80         192.168.1.14:45682      ESTABLISHED 1234/apache2
        
        ```
        
        **Analysis**:
        
        - **ESTABLISHED**: Active connections.
        - **SYN_RECV**: Half-open connections, possibly indicating a SYN flood attack.
        - High `Recv-Q` values may indicate that the server is receiving more data than it can process.
    - **Identify High Number of Connections from Specific IPs**:
    Check if any single IP is making an unusually high number of connections.
        
        ```bash
        netstat -anp | grep ':80' | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
        ```
        
        **Output**:
        
        ```
          1 192.168.1.10
          1 192.168.1.11
          1 192.168.1.12
        100 192.168.1.13
        
        ```
        
        **Analysis**:
        
        - IP `192.168.1.13` has a high number of connections, which could indicate a potential DoS attack or misbehaving client.
    - **Check Network Statistics for Errors**:
    Use `netstat` to check for packet errors.
        
        ```bash
        netstat -i
        ```
        
        **Output**:
        
        ```css
        Kernel Interface table
        Iface      MTU    RX-OK RX-ERR RX-DRP RX-OVR TX-OK TX-ERR TX-DRP TX-OVR Flg
        eth0      1500   123456      0      0 0      654321      0      0      0 BMRU
        
        ```
        
        **Analysis**:
        
        - Check for non-zero `RX-ERR`, `RX-DRP`, `TX-ERR`, or `TX-DRP` values which indicate network issues such as dropped packets or errors.
        1. **Monitor Real-Time Traffic**:
        Use `iftop` to monitor real-time traffic.
            
            ```bash
            sudo iftop -i eth0
            ```
            
            **Output**:
            
            ```markdown
              10.0.0.1       => 192.168.1.13  2.00Mb  2.00Mb  2.00Mb
                             <=              1.00Mb  1.00Mb  1.00Mb
            ```
            
            **Analysis**:
            
            - High traffic from/to `192.168.1.13`, confirming the potential issue with this IP.
        
        ### Response Steps:
        
        1. **Block Suspicious IP**:
        Block the IP making excessive connections using `iptables`.
            
            ```bash
            sudo iptables -A INPUT -s 192.168.1.13 -j DROP
            ```
            
        2. **Rate Limit Connections**:
        Implement rate limiting to prevent future issues.
            
            ```bash
            sudo iptables -A INPUT -p tcp --dport 80 -m connlimit --connlimit-above 20 -j REJECT --reject-with tcp-reset
            
            ```
            
        3. **Monitor for Stability**:
        Continuously monitor network connections and traffic after taking action to ensure stability.
            
            ```bash
            netstat -tuln
            ```
            
            ```bash
            sudo iftop -i eth0
            ```
            
        
        By following these steps, you can identify network issues using `netstat`, take necessary actions to mitigate problems, and ensure the stability and performance of your Linux-hosted website.
        
5. **Load Average**:
    - Monitor the system load average over 1, 5, and 15 minutes.
    - Tools: `uptime`, `top`
    
    uptime
    

### Application Metrics

1. **Web Server Metrics**:
    - Requests per second, error rates, response times.
    - Tools: Server logs, monitoring tools specific to web servers (e.g., Apache’s `mod_status`, Nginx’s `stub_status`)
2. **Database Metrics**:
    - Query performance, slow queries, connection counts.
    - Tools: Database logs, `mysqladmin`, `pg_stat_activity` for PostgreSQL
3. **Application-Specific Metrics**:
    - Application logs for errors and performance issues.
    - Tools: Depends on the application (e.g., custom logging, monitoring solutions like New Relic, Datadog)

### Security Metrics

1. **Login Attempts**:
    - Monitor successful and failed login attempts.
    - Tools: `/var/log/auth.log`, `/var/log/secure`
2. **Firewall Logs**:
    - Track blocked and allowed traffic.
    - Tools: `iptables`, `ufw`, `/var/log/syslog`
3. **File Integrity**:
    - Monitor changes to critical system files.
    - Tools: `aide`, `tripwire`

### Service-Specific Metrics

1. **Uptime and Downtime**:
    - Track the uptime and any periods of downtime for critical services.
    - Tools: `systemctl`, `service`
2. **Process Health**:
    - Ensure critical processes are running and check for any unexpected crashes.
    - Tools: `ps`, `pgrep`, `systemctl`

### User Experience Metrics

1. **Response Time**:
    - Measure how quickly the server responds to requests.
    - Tools: `curl`, `ab` (ApacheBench), `JMeter`
2. **Error Rates**:
    - Track HTTP error rates (e.g., 404, 500 errors).
    - Tools: Web server logs, application logs

### Monitoring and Alerting Tools

1. **System Monitoring Tools**:
    - `Nagios`, `Zabbix`, `Prometheus`, `Grafana`
2. **Log Management Tools**:
    - `ELK Stack` (Elasticsearch, Logstash, Kibana), `Graylog`, `Splunk`
3. **Cloud-Based Monitoring**:
    - Solutions like `AWS CloudWatch`, `Google Cloud Monitoring`, `Azure Monitor` if using cloud services.
