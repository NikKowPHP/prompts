# ==========================================
# CLASH VERGE TUN MODE CONFIGURATION
# ==========================================
port: 7890
socks-port: 7891
mixed-port: 7892
allow-lan: true
mode: rule
log-level: info
ipv6: false

# --- 1. TUN CONFIGURATION ---
# This section completely replaces 'tun2socks' and your 'ip route' commands.
tun:
  enable: true
  stack: system              # 'system' is stable on Linux; use 'gvisor' if you get errors.
  dns-hijack:
    - any:53                 # Intercepts standard DNS requests
  auto-route: true           # Replaces 'ip route add default...'
  auto-detect-interface: true # Replaces manually specifying 'enp2s0'

# --- 2. DNS CONFIGURATION ---
# This replaces your 'systemd-resolved' and 'dig' logic.
dns:
  enable: true
  listen: :1053
  ipv6: false
  enhanced-mode: fake-ip     # Returns instant fake IPs to apps; resolves remotely.
  fake-ip-range: 198.19.0.1/16
  
  # These are the bootstrap servers used ONLY to resolve the secure DNS servers below.
  default-nameserver:
    - 1.1.1.1
    - 8.8.8.8
  
  # The actual Secure DNS (DoT) you wanted.
  nameserver:
    - tls://1.1.1.1
    - tls://8.8.8.8
    # Fallback
    - https://1.1.1.1/dns-query

  # Fake-IP Filter: Forces real IP resolution for specific domains 
  # (Useful if Supabase/Postgres tools have issues with Fake-IPs, though usually not needed).
  fake-ip-filter:
    - "*.lan"
    - "*.local"
    - "+.supabase.com"  # <--- ADDED: Covers pooler.supabase.com and api.supabase.com
    - "+.supabase.co"   # <--- ADDED: Just in case

# --- 3. PROXY SERVER ---
proxies:
  - name: "Corporate-Proxy"
    type: http
    server: 172.16.2.254
    port: 3128
    # username: "user" 
    # password: "pass" 

proxy-groups:
  - name: "Proxy-Selector"
    type: select
    proxies:
      - "Corporate-Proxy"
      - DIRECT

# --- 4. ROUTING RULES ---
# Clash processes rules from top to bottom. First match wins.
rules:
  # === 1. DNS Servers ===
  # We must route the encrypted DNS traffic directly to avoid loops
  - IP-CIDR,1.1.1.1/32,DIRECT
  - IP-CIDR,8.8.8.8/32,DIRECT

  # === 2. The Proxy Server Itself ===
  # Prevents infinite routing loops (Critical)
  - IP-CIDR,172.16.2.254/32,DIRECT

  # === 3. Database Exemptions ===
  # Replaces your manual 'dig' and route addition.
  # Clash will detect the domain and route traffic out the physical interface.
  - DOMAIN-SUFFIX,pooler.supabase.com,DIRECT

  # === 4. Local Network & Geo Exemptions ===
  - GEOIP,PRIVATE,DIRECT
  - GEOIP,CN,DIRECT

  # === 5. Default Route (The Tunnel) ===
  - MATCH,Proxy-Selector