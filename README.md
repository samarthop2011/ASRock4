# asrock-hunter

Continuous keyless hunter for ASRock Rack BMCs and CVE-2024-54085
(AMI MegaRAC Redfish host-interface authentication bypass).

## Install

```bash
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
```

## Run

```bash
python hunter.py --once          # one cycle, then exit
python hunter.py                 # 24/7 daemon
python hunter.py --stats
python hunter.py --export
python hunter.py --full --ports 443,8443
```

Install as a service:

```bash
sudo cp -r . /opt/asrock-hunter && sudo useradd -r hunter
sudo chown -R hunter /opt/asrock-hunter
sudo cp systemd/asrock-hunter.service /etc/systemd/system/
sudo systemctl enable --now asrock-hunter
```

## Databases

`data/hunter.db` (SQLite, WAL)

- `hosts` — every IP ever seen, with port, fingerprint, score, geo/ASN, status, evidence
- `vulns` — CVE ledger with `confidence` (`suspect` / `likely` / `confirmed`), method, evidence
- `subnets`, `source_stats`, `runs`, `kv`

Exports in `data/exports/`: `all_ips.txt`, `all_hosts.csv`, `alive_hosts.csv`,
`vuln_ips.txt`, `vulns.csv`, `vulns.jsonl`.

## Confidence model

| Level | Meaning |
|---|---|
| `suspect` | ASRock/MegaRAC fingerprint, Redfish probes blocked |
| `likely` | one protected Redfish endpoint answered anonymously |
| `confirmed` | 2+ protected endpoints, or an X-Auth-Token issued, or InternetDB CVE tag |

Set `notify_min_confidence` to control what gets pushed to webhooks.

## Sources

Subnet files, live ASN prefix harvest, crt.sh, HackerTarget, DuckDuckGo/Bing dorks,
Shodan/FOFA/ZoomEye/Quake/Hunter/Netlas/Censys/Onyphe HTML search, Shodan
InternetDB and ip-api enrichment. Broken sources back off automatically after 5
consecutive failures and resume later.

## Tuning notes

- `max_ips_per_subnet` samples large ranges deterministically; `--full` disables sampling.
- Raise `concurrency` only if you have headroom; BMCs throttle hard above ~50 rps total.
- Ports 623 (IPMI) and 5900/5901 (KVM) are usually reachable on the same host as 443.
- `source_interval_hours` and `rescan_*` control politeness against both targets and engines.
