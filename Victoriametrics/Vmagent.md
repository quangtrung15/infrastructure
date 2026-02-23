# 1.Vmagent là gì?
# 2.Install vmagent on ubuntu
- Cài đặt và giải nén
  - `wget "https://github.com/VictoriaMetrics/VictoriaMetrics/releases/download/v1.96.0/vmutils-linux-amd64-v1.96.0.tar.gz"`
  - `tar xzf vmutils-linux-amd64-v1.96.0.tar.gz`
    - <img width="1185" height="310" alt="image" src="https://github.com/user-attachments/assets/e6e7cce7-6377-4d6f-973c-5e0c034e14d7" />
- Tạo file config
  - `sudo vi /etc/vmagent/vmagent.yml`
  -   ```yaml
      global:
        scrape_interval: 30s
          # external_labels:
          # cluster: 'vm-cluster'
          # environment: 'production'
      
      scrape_configs:
        # Scrape VM1 - node_exporter
        - job_name: 'vm1-node'
          static_configs:
            - targets: ['172.16.0.89:9100']
              labels:
                instance: 'vm1'
      
          relabel_configs:
            - source_labels: [__address__]
              target_label: component
              replacement: 'node_exporter'
      
                # metric_relabel_configs:
            # - source_labels: [__name__]
            # regex: 'node_(cpu_seconds_total|memory_.*|disk_.*|filesystem_.*|network_.*|load.*)'
            # action: keep
      
        # Scrape VM1 - VictoriaMetrics components
        - job_name: 'vm1-victoria_primary'
          static_configs:
            - targets:
                - '172.16.0.89:8482'  # vmstorage
                - '172.16.0.89:8480'  # vminsert
                - '172.16.0.89:8481'  # vmselect
              labels:
                instance: 'vm1'
      
          relabel_configs:
            - source_labels: [__address__]
              regex: '.*:8482'
              target_label: component
              replacement: 'vmstorage'
            - source_labels: [__address__]
              regex: '.*:8480'
              target_label: component
              replacement: 'vminsert'
            - source_labels: [__address__]
              regex: '.*:8481'
              target_label: component
              replacement: 'vmselect'
      
        # Scrape VM2 - node_exporter
        - job_name: 'vm2-node'
          static_configs:
            - targets: ['172.16.0.90:9100']
              labels:
                instance: 'vm2'
                component: 'node_exporter'
      
        # Scrape VM2 - vmstorage replica
        - job_name: 'vm2-victoria_replica'
          static_configs:
            - targets: ['172.16.0.90:8482']
              labels:
                instance: 'vm2'
                component: 'vmstorage-replica'
      
        # Scrape VM3 (chính nó) - node_exporter
        - job_name: 'vm3-node'
          static_configs:
            - targets: ['localhost:9100']
              labels:
                instance: 'vm3'
                component: 'node_exporter'
      
        # Scrape VM3 - vmagent itself
        - job_name: 'vm3-vmagent'
          static_configs:
            - targets: ['localhost:8429']
              labels:
                instance: 'vm3'
                component: 'vmagent'
      ```
- Tạo file service
  - `sudo vi /etc/systemd/system/vmagent.service`
  - ```yaml
    [Unit]
    Description=VictoriaMetrics Agent
    After=network.target
    
    [Service]
    Type=simple
    User=victoriametrics
    WorkingDirectory=/opt/vmutils/
    ExecStart=/opt/vmutils/vmagent-prod \
      -promscrape.config=/etc/vmagent/vmagent.yml \
      -remoteWrite.url=http://172.16.0.89:8427/insert/0/prometheus \
      -remoteWrite.basicAuth.username=vmagent \
      -remoteWrite.basicAuth.password=vmagent123 \
      -remoteWrite.maxDiskUsagePerURL=1GB \
      -httpListenAddr=:8429 \
      -promscrape.maxScrapeSize=16MB
    Restart=on-failure
    LimitNOFILE=65536
    
    [Install]
    WantedBy=multi-user.target
    ```
- Khởi động lại service
  - <img width="1653" height="471" alt="image" src="https://github.com/user-attachments/assets/4d18f051-7e05-408e-abfe-30f623c6e2c7" />
  
- Kiểm tra 
  - <img width="1281" height="1017" alt="image" src="https://github.com/user-attachments/assets/9ae34df3-a989-452c-a9b8-f27b9c70d988" />
