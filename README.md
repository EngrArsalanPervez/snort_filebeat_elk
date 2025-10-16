# Dated: 15-Oct-2025
# Ubuntu 24.04.3 LTS

# Update
```bash
sudo apt update
```

# Pre-Requisite
```bash
sudo apt install -y git curl cmake make pkg-config tmux screen net-tools tshark bridge-utils htop

sudo apt install -y build-essential cmake pkg-config git autoconf automake \
libtool bison flex libpcap-dev libpcre3-dev libpcre2-dev libdumbnet-dev  \
libluajit-5.1-dev libhwloc-dev liblzma-dev zlib1g-dev libssl-dev uuid-dev \
libcmocka-dev libsqlite3-dev cpputest libunwind-dev libmnl-dev libnetfilter-queue-dev \
libmnl-dev ethtool lsb-release wget ca-certificates
```

# IP Forwarding
```bash
nano /etc/sysctl.conf 
    net.ipv4.ip_forward=1
sudo sysctl -p /etc/sysctl.conf

sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv4.conf.all.rp_filter=0
sudo sysctl -w net.ipv4.conf.default.rp_filter=0

sudo ip link set enp2s0f2 up
sudo ip link set enp2s0f3 up
sudo ethtool -K enp2s0f2 gro off lro off gso off
sudo ethtool -K enp2s0f3 gro off lro off gso off

sudo ip addr flush dev enp2s0f2
sudo ip addr flush dev enp2s0f3
```



# Download
```bash
https://www.snort.org/downloads
Snort3:
    libdaq-3.0.21.tar.gz
    libml-2.0.0.tar.gz
    snort3-3.9.6.0.tar.gz
    snort3_extra-3.9.6.0.tar.gz
```

# Daq
```bash
tar xzf libdaq
cd libdaq
./bootstrap
./configure
make
make install
```

# libml
```bash
tar xzf libml
cd libml
./configure.sh
cd build
sudo make -j$(nproc) install
```

# snort
```bash
tar xzf snort
./configure_cmake.sh
cd build
make -j $(nproc) install
```

# snort extras
```bash
tar xzf snort extras
./configure_cmake.sh
cd build
make
make install
/usr/local/snort/bin/snort -c /usr/local/snort/etc/snort/snort.lua --version

sudo /usr/local/snort/bin/snort -c /usr/local/snort/etc/snort/snort.lua \
  --daq afpacket \
  --daq-var mode=inline \
  --daq-var buffer_size_mb=512 \
  --daq-var fanout_type=hash \
  --daq-var fanout_flag=defrag \
  --daq-var block_size=65536 \
  --daq-var block_timeout=1000 \
  --daq-var num_threads=2 \
  -i enp2s0f2:enp2s0f3 \
  -Q \
  -l /var/log/snort
```


#   ELK
```bash
sudo apt install openjdk-17-jdk -y
wget https://artifacts.elastic.co/GPG-KEY-elasticsearch -O /etc/apt/keyrings/GPG-KEY-elasticsearch.key
echo "deb [signed-by=/etc/apt/keyrings/GPG-KEY-elasticsearch.key] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
sudo apt install elasticsearch -y
sudo systemctl daemon-reload
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl status elasticsearch

sudo nano /etc/elasticsearch/elasticsearch.yml
    network.host: 0.0.0.0
    xpack.security.enabled: false
sudo systemctl restart elasticsearch
curl -X GET "localhost:9200"

sudo apt install kibana -y
sudo systemctl start kibana
sudo systemctl enable kibana
sudo systemctl status kibana
sudo nano /etc/kibana/kibana.yml
    server.host: 0.0.0.0
sudo systemctl restart kibana



curl -X PUT "localhost:9200/_index_template/snort-alerts-template" -H 'Content-Type: application/json' -d'
{
  "index_patterns": ["snort-alerts-*"],
  "priority": 500,
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    },
    "mappings": {
      "dynamic": true,
      "properties": {
        "@timestamp": {"type": "date"},
        "timestamp": {"type": "text"},
        "pkt_num": {"type": "long"},
        "proto": {"type": "keyword"},
        "pkt_gen": {"type": "keyword"},
        "pkt_len": {"type": "long"},
        "dir": {"type": "keyword"},
        "src_ap": {"type": "keyword"},
        "dst_ap": {"type": "keyword"},
        "rule": {"type": "keyword"},
        "action": {"type": "keyword"},
        "gid": {"type": "long"},
        "sid": {"type": "long"},
        "rev": {"type": "long"}
      }
    }
  }
}'
```

# Filebeat
```bash
sudo apt install -y filebeat
sudo systemctl enable filebeat
sudo systemctl enable --now filebeat
sudo systemctl status filebeat
sudo tail -f /var/log/filebeat/filebeat
```













