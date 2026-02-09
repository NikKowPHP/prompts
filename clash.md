# ==========================================
# CLASH VERGE TUN MODE CONFIGURATION (FIXED)
# ==========================================
port: 7890
socks-port: 7891
mixed-port: 7892
allow-lan: true
mode: rule
log-level: info
ipv6: false

# --- 1. TUN CONFIGURATION ---
tun:
  enable: true
  stack: gvisor 
  dns-hijack:
    - any:53
  auto-route: true
  auto-detect-interface: true

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
  
  # Standard DNS might be blocked in your office. 
  # If 1.1.1.1 fails, Clash will try to route domains via the Proxy anyway.
  default-nameserver:
    - 1.1.1.1
    - 8.8.8.8
  
  nameserver:
    - https://1.1.1.1/dns-query
    - tls://8.8.8.8

  fake-ip-filter:
    - "*.lan"
    - "*.local"
    # DISABLED SUPABASE FILTER TEMPORARILY
    # If 1.1.1.1 is blocked by your company, keeping this enabled breaks the connection.
    # Letting it use Fake-IP allows the Proxy to handle the DNS resolution for us.
    # - "+.supabase.com"
    # - "+.supabase.co"

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

  # 2. Database Ports -> MUST GO THROUGH PROXY
  # If we send these Direct, the firewall kills them (ERR_CONNECTION_CLOSED)
  - DST-PORT,5432,Proxy-Selector
  - DST-PORT,6543,Proxy-Selector

  # 3. Supabase Domains -> MUST GO THROUGH PROXY
  - DOMAIN-SUFFIX,pooler.supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.co,Proxy-Selector

  # 4. General
  - GEOIP,PRIVATE,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,Proxy-Selector