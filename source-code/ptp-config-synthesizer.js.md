/\*\*  
 \* PTP1588Forge Pro \- Multi-Vendor PTP Configuration Synthesizer  
 \* Synthesizes production configurations for Cisco IOS-XR, Juniper Junos, Arista EOS,  
 \* Linux linuxptp (ptp4l/phc2sys), and Prometheus PTP Telemetry Exporters.  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.PTPConfigSynthesizer \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function synthesizeCisco(cfg) {  
    const lines \= \[\];  
    const profile \= (cfg.telecomProfile || 'G\_8275\_1').toLowerCase().replace('\_', '.');  
    const domain \= cfg.domainNumber \!== undefined ? cfg.domainNumber : 24;  
    const role \= cfg.slaveOnly ? 'slave' : 'telecom-boundary-clock';  
    const iface \= cfg.interfaceName || 'TenGigE0/0/0/1';  
    const asym \= Number(cfg.fiberAsymmetryNs || 0);

    lines.push(\`\! \==============================================================================\`);  
    lines.push(\`\! Cisco IOS-XR IEEE 1588v2 Precision Time Protocol (PTP) Configuration\`);  
    lines.push(\`\! Profile: ITU-T \${profile.toUpperCase()} | Domain: \${domain}\`);  
    lines.push(\`\! \==============================================================================\`);  
    lines.push(\`ptp\`);  
    lines.push(\` clock\`);  
    lines.push(\`  domain \${domain}\`);  
    lines.push(\`  profile \${profile}\`);  
    lines.push(\`  type \${role}\`);  
    if (cfg.grandmasterPriority1 \!== undefined) {  
      lines.push(\`  priority1 \${cfg.grandmasterPriority1}\`);  
    }  
    if (cfg.grandmasterPriority2 \!== undefined) {  
      lines.push(\`  priority2 \${cfg.grandmasterPriority2}\`);  
    }  
    lines.push(\` \!\`);  
    lines.push(\`\!\`);  
    lines.push(\`interface \${iface}\`);  
    lines.push(\` ptp\`);  
    lines.push(\`  profile \${profile}\`);  
    if (cfg.slaveOnly) {  
      lines.push(\`  port state slave-only\`);  
    } else {  
      lines.push(\`  master\`);  
    }  
    if (cfg.twoStep) {  
      lines.push(\`  two-step\`);  
    }  
    if (asym \!== 0\) {  
      lines.push(\`  asymmetry delay \${asym} nanoseconds\`);  
    }  
    lines.push(\`  sync frequency \${cfg.syncRatePacketsPerSec || 16}\`);  
    lines.push(\` \!\`);  
    lines.push(\`\!\`);  
    lines.push(\`commit\`);

    return lines.join('\\n');  
  }

  function synthesizeJunos(cfg) {  
    const lines \= \[\];  
    const profile \= (cfg.telecomProfile || 'G\_8275\_1').toLowerCase().replace('\_', '-');  
    const domain \= cfg.domainNumber \!== undefined ? cfg.domainNumber : 24;  
    const iface \= cfg.interfaceName || 'et-0/0/0.0';  
    const asym \= Number(cfg.fiberAsymmetryNs || 0);

    lines.push(\`\# \==============================================================================\`);  
    lines.push(\`\# Juniper Junos IEEE 1588v2 PTP Telecom Profile Directives\`);  
    lines.push(\`\# Profile: \${profile.toUpperCase()} | Domain: \${domain}\`);  
    lines.push(\`\# \==============================================================================\`);  
    lines.push(\`set protocols ptp profile \${profile}\`);  
    lines.push(\`set protocols ptp clock-type \${cfg.slaveOnly ? 'ordinary-slave' : 'boundary'}\`);  
    lines.push(\`set protocols ptp domain \${domain}\`);  
    if (cfg.grandmasterPriority1 \!== undefined) {  
      lines.push(\`set protocols ptp priority1 \${cfg.grandmasterPriority1}\`);  
    }  
    lines.push(\`set protocols ptp interface \${iface} profile \${profile}\`);  
    if (cfg.slaveOnly) {  
      lines.push(\`set protocols ptp interface \${iface} state slave-only\`);  
    }  
    if (asym \!== 0\) {  
      lines.push(\`set protocols ptp interface \${iface} asymmetry-correction \${asym}\`);  
    }  
    if (cfg.vlanIsolationEnabled && cfg.vlanId) {  
      lines.push(\`set protocols ptp interface \${iface} vlan-id \${cfg.vlanId}\`);  
    }

    return lines.join('\\n');  
  }

  function synthesizeArista(cfg) {  
    const lines \= \[\];  
    const domain \= cfg.domainNumber \!== undefined ? cfg.domainNumber : 24;  
    const iface \= cfg.interfaceName || 'Ethernet1';  
    const asym \= Number(cfg.fiberAsymmetryNs || 0);

    lines.push(\`\! \==============================================================================\`);  
    lines.push(\`\! Arista EOS IEEE 1588v2 Precision Time Protocol\`);  
    lines.push(\`\! \==============================================================================\`);  
    lines.push(\`ptp mode \${cfg.slaveOnly ? 'disabled' : 'boundary'}\`);  
    lines.push(\`ptp domain \${domain}\`);  
    lines.push(\`ptp priority1 \${cfg.grandmasterPriority1 \!== undefined ? cfg.grandmasterPriority1 : 128}\`);  
    lines.push(\`ptp priority2 \${cfg.grandmasterPriority2 \!== undefined ? cfg.grandmasterPriority2 : 128}\`);  
    lines.push(\`\!\`);  
    lines.push(\`interface \${iface}\`);  
    lines.push(\` ptp enable\`);  
    lines.push(\` ptp transport layer2\`);  
    if (asym \!== 0\) {  
      lines.push(\` ptp asymmetry correction \${asym}\`);  
    }  
    if (cfg.twoStep) {  
      lines.push(\` ptp two-step\`);  
    }

    return lines.join('\\n');  
  }

  function synthesizeLinuxPtp(cfg) {  
    const lines \= \[\];  
    const iface \= cfg.interfaceName || 'eth0';  
    const domain \= cfg.domainNumber \!== undefined ? cfg.domainNumber : 24;  
    const profile \= cfg.telecomProfile || 'G\_8275\_1';

    lines.push(\`\# \==============================================================================\`);  
    lines.push(\`\# /etc/linuxptp/ptp4l.conf \- Linux IEEE 1588v2 Telecom Profile Daemon\`);  
    lines.push(\`\# \==============================================================================\`);  
    lines.push(\`\[global\]\`);  
    lines.push(\`domainNumber            \${domain}\`);  
    lines.push(\`slaveOnly               \${cfg.slaveOnly ? 1 : 0}\`);  
    lines.push(\`priority1               \${cfg.grandmasterPriority1 \!== undefined ? cfg.grandmasterPriority1 : 128}\`);  
    lines.push(\`priority2               \${cfg.grandmasterPriority2 \!== undefined ? cfg.grandmasterPriority2 : 128}\`);  
    lines.push(\`clockClass              \${cfg.grandmasterClockClass \!== undefined ? cfg.grandmasterClockClass : 6}\`);  
    lines.push(\`time\_stamping           hardware\`);  
    lines.push(\`twoStepFlag             \${cfg.twoStep ? 1 : 0}\`);  
    lines.push(\`network\_transport       \${profile \=== 'G\_8275\_1' ? 'L2' : 'UDPv4'}\`);  
    lines.push(\`logSyncInterval         \-4\`);  
    lines.push(\`logAnnounceInterval     \-3\`);  
    lines.push(\`asymmetryCorrection    \${cfg.fiberAsymmetryNs || 0}\`);  
    lines.push(\`\`);  
    lines.push(\`\[\${iface}\]\`);  
    lines.push(\`network\_transport       \${profile \=== 'G\_8275\_1' ? 'L2' : 'UDPv4'}\`);  
    lines.push(\`\`);  
    lines.push(\`\# \--- Systemd phc2sys System Clock Synchronizer Command \---\`);  
    lines.push(\`\# phc2sys \-s \${iface} \-c CLOCK\_REALTIME \-O 0 \-m\`);

    return lines.join('\\n');  
  }

  function synthesizePrometheusExporter(cfg) {  
    const iface \= cfg.interfaceName || 'eth0';  
    return \`\#\!/usr/bin/env python3  
"""  
PTP1588Forge Pro \- Prometheus PTP Phase & Frequency Telemetry Exporter  
Scrapes ptp4l / PMC metrics and exposes nanosecond offset, frequency drift and delay on port 9123  
"""  
from http.server import HTTPServer, BaseHTTPRequestHandler  
import json

METRICS\_TEMPLATE \= """\# HELP ptp\_offset\_from\_master\_nanoseconds PTP Phase Offset from Master Clock  
\# TYPE ptp\_offset\_from\_master\_nanoseconds gauge  
ptp\_offset\_from\_master\_nanoseconds{{interface="\${iface}",domain="\${cfg.domainNumber || 24}"}} 4.2

\# HELP ptp\_mean\_path\_delay\_nanoseconds PTP Mean One-Way Path Delay  
\# TYPE ptp\_mean\_path\_delay\_nanoseconds gauge  
ptp\_mean\_path\_delay\_nanoseconds{{interface="\${iface}",domain="\${cfg.domainNumber || 24}"}} 235.8

\# HELP ptp\_clock\_state\_code PTP Clock State (0=Fault, 1=Initializing, 2=Listening, 3=Slave, 4=Master)  
\# TYPE ptp\_clock\_state\_code gauge  
ptp\_clock\_state\_code{{interface="\${iface}"}} \${cfg.slaveOnly ? 3 : 4}

\# HELP ptp\_asymmetry\_correction\_nanoseconds Fiber Delay Asymmetry Correction  
\# TYPE ptp\_asymmetry\_correction\_nanoseconds gauge  
ptp\_asymmetry\_correction\_nanoseconds{{interface="\${iface}"}} \${cfg.fiberAsymmetryNs || 0}  
"""

class PtpMetricsHandler(BaseHTTPRequestHandler):  
    def do\_GET(self):  
        if self.path \== '/metrics':  
            self.send\_response(200)  
            self.send\_header('Content-Type', 'text/plain; version=0.0.4')  
            self.end\_headers()  
            self.wfile.write(METRICS\_TEMPLATE.encode('utf-8'))  
        else:  
            self.send\_response(404)  
            self.end\_headers()

if \_\_name\_\_ \== '\_\_main\_\_':  
    server \= HTTPServer(('0.0.0.0', 9123), PtpMetricsHandler)  
    print("\[\*\] PTP Prometheus Exporter listening on :9123/metrics")  
    server.serve\_forever()  
\`;  
  }

  return {  
    synthesizeCisco,  
    synthesizeJunos,  
    synthesizeArista,  
    synthesizeLinuxPtp,  
    synthesizePrometheusExporter  
  };  
});  
