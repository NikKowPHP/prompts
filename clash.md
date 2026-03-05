# ==========================================
# CLASH VERGE TUN MODE CONFIGURATION
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
  stack: gvisor 
  dns-hijack:
    - any:53
    - tcp://any:53
  auto-route: true
  auto-detect-interface: true
  strict-route: true 

# --- 2. SNIFFER (CRITICAL FOR DB) ---
sniffer:
  enable: true
  parse-pure-ip: true
  sniff:
    tls: # Added 6543 here
      ports: [443, 5432, 6543] 
    http:
      ports: [80, 8080-8888]

# --- 3. DNS CONFIGURATION ---
dns:
  enable: true
  listen: :1053
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.10.0.1/16
  
  default-nameserver:
    - 1.1.1.1
    - 8.8.8.8
  
  nameserver:
    - https://1.1.1.1/dns-query
    - tls://8.8.8.8

  fake-ip-filter:
    - "*.lan"
    - "*.local"
    - "*.supabase.co"
    - "*.supabase.com"
    - "*.pooler.supabase.com"
    - "*.ngrok.com"
    - "*.ngrok.io"
    - "crl.ngrok-agent.com"
    # Add these two lines:
    - "*.debian.org"
    - "*.debian.net"

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
  # Add these at the TOP of your rules section
  - DOMAIN-SUFFIX,pooler.supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.co,Proxy-Selector
  
  # Force all 443 traffic to Supabase through the proxy
  - DOMAIN,aws-0-us-west-1.pooler.supabase.com,Proxy-Selector
  # 1. Local / Bypass
  - IP-CIDR,127.0.0.0/8,DIRECT
  - IP-CIDR,172.16.0.0/12,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT

  # 2. Database & Supabase (Try Proxy first, then Direct if it fails)
  - DOMAIN-SUFFIX,pooler.supabase.com,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.co,Proxy-Selector
  - DOMAIN-SUFFIX,supabase.com,Proxy-Selector
  - DST-PORT,5432,Proxy-Selector
  - DST-PORT,6543,Proxy-Selector

  # 3. Cloudflare (For your Website Tunnel)
  - DOMAIN-KEYWORD,cloudflare,Proxy-Selector
  - DOMAIN-SUFFIX,cloudflare.com,Proxy-Selector
  - DOMAIN-SUFFIX,argotunnel.com,Proxy-Selector

  # 4. Docker / Debian Fixes
  - DOMAIN-SUFFIX,debian.org,Proxy-Selector
  - DOMAIN-SUFFIX,debian.net,Proxy-Selector

  # 5. General
  - GEOIP,PRIVATE,DIRECT
  - MATCH,Proxy-Selector