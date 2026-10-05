# Speedtest + iperf3 Network Measurement Bridge

**Version:** 1.0.1

WAN internet speed tests, LAN throughput benchmarks via **iperf3**, and host
**ping** / latency measurement — with JSON and HTML report output.

## Try it

1. Sidebar → **Extensions** → **Install from file...**
2. Pick `../network-measurement.dsext` (sibling to this folder after packing).
3. Confirm the trust warning. `speedtest-cli==2.1.3` installs into this
   extension's private `deps/` folder.
4. Optionally set **iperf3 Path**, **Report Output Directory**, and
   **Default Output Format** on the extension card.
5. Nodes appear in the palette's **network** category:
   - **Run Speedtest** — download / upload Mbps, ping, jitter
   - **Run iperf3 Client** — LAN throughput against an iperf3 server
   - **Ping Host** — latency, jitter, packet loss
6. An example workflow (**Network Measurement Demo**) is inserted on the
   Workflows page after install.

## Prerequisites

| Tool | Required for | Notes |
|------|--------------|-------|
| `speedtest-cli` (pip) | Run Speedtest | Shipped as a pip dependency; Ookla CLI optional override |
| `iperf3` binary | Run iperf3 Client | Install separately and put on PATH (or set **iperf3 Path**) |
| OS `ping` | Ping Host | Built into Windows / macOS / Linux |

## What each file does

- **`nodes/_measure.py`**: shared binary resolution, result normalisation,
  JSON/HTML report generation and template injection
- **`report_templates/report-template.html`**: light/dark-aware report
  (follows the OS `prefers-color-scheme`); Python injects measurement JSON
  via `/*__MEASUREMENT_RESULTS_PLACEHOLDER__*/`
- **`templates/network-measurement-demo.json`**: example workflow showcasing
  all three nodes with HTML report output

## Contract (v1.0.1)

**Install:** Extensions → **Install from file** → choose `network-measurement.dsext`.

**Permissions:**
- **network**
- **filesystem**
- **shell**

Example workflow template already ships with this extension.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/network-measurement
python tools/deskstride_ext_cli.py pack extensions/network-measurement -o network-measurement.dsext
```
