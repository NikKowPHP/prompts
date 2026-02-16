# ==========================================
# CLASH VERGE TUN MODE CONFIGURATION (FIXED FOR DOCKER)
# ==========================================
port: 7890
socks-port: 7891
mixed-port: 7892
allow-lan: true
bind-address: "*"
mode: rule
log-level: info
ipv6: false

# --- 1. TUN CONFIGURATION ---
tun:
  enable: true
  stack: gvisor # gvisor is generally better for Linux/Docker compatibility
  dns-hijack:
    - any:53
    - tcp://any:53
  auto-route: true
  auto-detect-interface: true
  # STRICT ROUTE: Helps capture traffic that might otherwise bypass TUN
  strict-route: true 

# --- 2. SNIFFER ---
sniffer:
  enable: true
  parse-pure-ip: true
  sniff:
    tls:
      ports: [443, 5432, 6543] 
    http:
      ports: [80, 8080-8888]

# --- 3. DNS CONFIGURATION ---
dns:
  enable: true
  listen: :1053
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.19.0.1/16
  
  default-nameserver:
    - 1.1.1.1
    - 8.8.8.8
  
  nameserver:
    - https://1.1.1.1/dns-query
    - tls://8.8.8.8

  fake-ip-filter:
    - "*.lan"
    - "*.local"
    - "*.ngrok.com"
    - "*.ngrok.io"
    - "crl.ngrok-agent.com"

# --- 4. PROXY SERVER ---
proxies:
  - name: "Corporate-Proxy"
    type: http
    server: 172.16.2.254
    port: 3128

proxy-groups:
  - name: "Proxy-Selector"
    type: select
    proxies:
      - "Corporate-Proxy"
      - DIRECT

# --- 5. ROUTING RULES ---
rules:
  # 1. DNS & Local (Keep Direct)
  - IP-CIDR,1.1.1.1/32,DIRECT
  - IP-CIDR,8.8.8.8/32,DIRECT
  - IP-CIDR,172.16.2.254/32,DIRECT
  
  # === DOCKER FIX: DEBIAN ARCHIVES ===
  # Old PHP images need to reach archive.debian.org. 
  # Force this through proxy to bypass local firewall blocks.
  - DOMAIN-SUFFIX,debian.org,Proxy-Selector
  - DOMAIN-SUFFIX,debian.net,Proxy-Selector
  - DOMAIN-KEYWORD,archive.debian,Proxy-Selector

  # 2. Database Ports
  - DST-PORT,5432,Proxy-Selector
  - DST-PORT,6543,Proxy-Selector

  # 3. Supabase Domains
  - DOMAIN-SUFFIX,pooler.supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.co,Proxy-Selector

  - DOMAIN-SUFFIX,ngrok.com,DIRECT
  - DOMAIN-SUFFIX,ngrok.io,DIRECT
  - DOMAIN-SUFFIX,ngrok-agent.com,DIRECT


# === CLOUDFLARE TUNNEL FIX ===
  - DOMAIN-KEYWORD,cloudflare,Proxy-Selector
  - DOMAIN-SUFFIX,cloudflare.com,Proxy-Selector
  - DOMAIN-SUFFIX,argotunnel.com,Proxy-Selector
  
  # 4. General
  - GEOIP,PRIVATE,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,Proxy-Selector