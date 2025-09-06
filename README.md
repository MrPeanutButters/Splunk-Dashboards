# Splunk Dashboards — Blue Team Homelab

This repo contains ready-to-import **Splunk dashboards** that mirror my homelab workflows:
- **RDP Brute Force Triage** (Windows Security / Sysmon)
- **Network Threats (Suricata + Zeek)** (Security Onion)
- **Vulnerability Remediation Tracker** (Nessus Essentials)

> These dashboards focus on repeatable triage & IR workflows: rapid detection, context, and verification.
> They are written in **SimpleXML** for compatibility and easy import (`Settings → User interface → Views → Import`).

## Dashboards
1. `01_rdp_bruteforce.xml` — Time series of failed RDP logons, top source IPs, affected accounts/hosts, and burst detection.
2. `02_network_threats_suricata_zeek.xml` — Suricata/Zeek alert trends, top signatures, talkers, and scan detectors.
3. `03_vuln_remediation_nessus.xml` — Open vulns by severity/CVE, affected hosts, and rescan/closure tracking.

## Saved Searches
`default/savedsearches.conf` includes:
- **Detect RDP Brute Force (4625 burst)** — burst rule for EventCode 4625 (LogonType 10).
- **Port Scan Detector (Suricata/Zeek)** — scan-y signatures + connection bursts.
- **Nessus Rescan Verification** — post-patch success tracking.

> **Adjust indexes/sourcetypes** for your environment. Examples assume:
> - Windows: `index=wineventlog sourcetype=XmlWinEventLog:Security` (or `WinEventLog:Security`)
> - Sysmon: `index=wineventlog sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational` (or `XmlWinEventLog:Sysmon`)
> - Suricata: `index=securityonion sourcetype=suricata:fast`
> - Zeek: `index=securityonion sourcetype=zeek:*`
> - Nessus: `index=nessus sourcetype=nessus:sc:vuln` (varies by TA)

## How to import
1. **Dashboards**: Splunk > *Settings* > *User interface* > *Views* > **Import** (`dashboards/*.xml`).
2. **Saved searches**: Copy `default/savedsearches.conf` into your app (e.g., `$SPLUNK_HOME/etc/apps/search/local/`), then restart Splunk or reload confs.
3. **Lookups** (optional): Place any CSVs under `lookups/` and reference by name in searches.

## Screenshots
See `assets/` for sample screenshots that show layout/intent.

## License
MIT


---

## Dashboard Studio versions (JSON)
Import from **Splunk > Dashboards > Create > Upload** and select files from `/dashboards_studio/`:

- `studio_rdp_bruteforce.json`
- `studio_network_threats.json`
- `studio_vuln_remediation.json`

These Studio dashboards use **macros** so you can point to your own indexes and sourcetypes without editing SPL. Edit `default/macros.conf`:

```ini
[index_windows]
definition = index=wineventlog

[sourcetype_windows_security]
definition = (sourcetype=XmlWinEventLog:Security OR sourcetype=WinEventLog:Security)

[index_so]
definition = index=securityonion

[sourcetype_suricata]
definition = sourcetype=suricata:fast

[sourcetype_zeek_conn]
definition = sourcetype=zeek:conn

[index_nessus]
definition = index=nessus

[sourcetype_nessus_vuln]
definition = sourcetype=nessus:sc:vuln
```

### Lookup example (GeoIP/ASN enrichment)
- CSV: `lookups/asn_geoip_sample.csv`
- Transform: `default/transforms.conf` (`asn_lookup`)
- Usage in SPL: `| lookup asn_lookup ip as Source_Network_Address OUTPUT asn asn_org country city`

> Replace the sample CSV with your own export from MaxMind/Team Cymru/etc.
