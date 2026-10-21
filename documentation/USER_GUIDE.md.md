\# PTP1588Forge Pro \- User Guide & CI/CD Telemetry Pipeline Manual

\#\# 1\. Web Studio Quickstart

1\. Open \`source-code/index.html\` in any modern web browser.  
2\. Select an industry preset from the top dropdown:  
   \- \*\*5G O-RAN Fronthaul G.8275.1 Cisco (Grade A+)\*\*: 5G Open RAN O-RU / O-DU synchronization using G.8275.1 Layer 2 Ethernet multicast, Class C T-BC, hardware PHY timestamping, 64 pkts/sec sync rate, and 45ns fiber asymmetry offset.  
   \- \*\*Datacenter HFT G.8275.2 Arista (Grade A+)\*\*: Low-latency financial trading PTP probe over L3 IP/UDP unicast (Domain 44), 1-step mode, Rubidium atomic clock holdover, and DSCP CS7 tagging.  
   \- \*\*Hazardous Rogue Master Flapping (Grade F)\*\*: Demonstrates failure modes of unprotected BMCA ordinary clocks with Priority 1 \= 10, software host timestamping, 1 pkt/sec sync rate, and free-running local oscillator.  
3\. Review the \*\*10-Point Zero-Trust Scorecard\*\* to inspect clock class, profile consistency, and fiber delay asymmetry.  
4\. Modify PTP parameters (Domain, Profile, Transport, Priority, Asymmetry, Clock Class).  
5\. Inspect the generated \*\*IEEE 1588v2 Packet Wireframe Hex Dump\*\* with byte offsets and decoded fields.  
6\. Switch synthesizer tabs to export production configurations:  
   \- \*\*Cisco IOS-XR\*\*: \`ptp\`, \`profile g.8275.1\`, \`telecom-boundary-clock\`, \`asymmetry delay\`.  
   \- \*\*Juniper Junos\*\*: \`set protocols ptp profile g-8275-1\`, \`asymmetry-correction\`.  
   \- \*\*Arista EOS\*\*: \`ptp mode boundary\`, \`ptp transport layer2\`.  
   \- \*\*Linux \`linuxptp\`\*\*: \`/etc/linuxptp/ptp4l.conf\` & \`phc2sys\` systemd configuration.  
   \- \*\*Prometheus PTP Exporter\*\*: Prometheus metrics endpoint on \`:9123/metrics\`.

\---

\#\# 2\. DevSecOps CLI Tool (\`ptp1588forge-cli.py\`)

The standalone Python CLI tool requires zero external dependencies and runs on Python 3.8+.

\#\#\# 2.1 Encoding IEEE 1588v2 Packets to Binary Hex  
\`\`\`bash  
\# Announce Message (64 Bytes)  
python3 build/ptp1588forge-cli.py encode-packet \-p presets/5g-oran-fronthaul-g8275-1-cisco.json \--type announce

\# Sync Message (44 Bytes)  
python3 build/ptp1588forge-cli.py encode-packet \-p presets/5g-oran-fronthaul-g8275-1-cisco.json \--type sync  
\`\`\`

\#\#\# 2.2 Computing Sub-Nanosecond Phase Offset & Mean Path Delay  
\`\`\`bash  
python3 build/ptp1588forge-cli.py calc-offset \--t1 1000.0 \--t2 1250.0 \--t3 2000.0 \--t4 2230.0 \--asym 10.0  
\`\`\`  
Output:  
\`\`\`  
\=== IEEE 1588v2 Nanosecond Timing Calculation \===  
Phase Offset from Master: 5.0 ns  
Mean One-Way Path Delay:  240.0 ns  
Forward Delay (t2 \- t1):  250.0 ns  
Reverse Delay (t4 \- t3):  230.0 ns  
Asymmetry Correction:    10.0 ns  
\`\`\`

\#\#\# 2.3 Running CI/CD Pipeline Telemetry Audits  
\`\`\`bash  
\# Clean production configuration \-\> Exits code 0  
python3 build/ptp1588forge-cli.py audit \-p presets/5g-oran-fronthaul-g8275-1-cisco.json \--strict

\# Insecure/fragile configuration \-\> Exits code 1 (Pipeline Fails)  
python3 build/ptp1588forge-cli.py audit \-p presets/vulnerable-flapping-rogue-master.json \--strict  
\`\`\`

\#\#\# 2.4 Synthesizing Router Configurations  
\`\`\`bash  
python3 build/ptp1588forge-cli.py synthesize \-p presets/5g-oran-fronthaul-g8275-1-cisco.json \-f cisco  
python3 build/ptp1588forge-cli.py synthesize \-p presets/datacenter-hft-g8275-2-arista.json \-f arista  
\`\`\`

\---

\#\# 3\. GitHub Actions CI/CD Pipeline Gate

Add this workflow to your NetDevOps repository (\`.github/workflows/ptp-sla-gate.yml\`):

\`\`\`yaml  
name: Zero-Trust PTP Telecom Profile CI/CD Gate

on:  
  pull\_request:  
    paths:  
      \- 'ptp/\*\*'  
      \- 'timing/\*\*'  
      \- 'fronthaul/\*\*'

jobs:  
  ptp-audit:  
    runs-on: ubuntu-latest  
    steps:  
      \- name: Checkout Code  
        uses: actions/checkout@v4

      \- name: Set up Python  
        uses: actions/setup-python@v5  
        with:  
          python-version: '3.11'

      \- name: Audit PTP Session Configuration (Strict Gate)  
        run: |  
          python3 build/ptp1588forge-cli.py audit \-p presets/5g-oran-fronthaul-g8275-1-cisco.json \--strict  
\`\`\`  
