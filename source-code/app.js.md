/\*\*  
 \* PTP1588Forge Pro \- Application State Orchestrator  
 \*/

document.addEventListener('DOMContentLoaded', () \=\> {  
  const PRESETS \= {  
    '5g-oran-fronthaul-g8275-1-cisco': {  
      nodeName: 'O-DU\_CELL\_SITE\_TBC',  
      clockRole: 'TELECOM\_BOUNDARY\_CLOCK',  
      slaveOnly: false,  
      masterGuardEnabled: true,  
      telecomProfile: 'G\_8275\_1',  
      transportType: 'ETHERNET\_L2',  
      domainNumber: 24,  
      timestampingMode: 'HARDWARE\_PHY',  
      twoStep: true,  
      fiberAsymmetryNs: 45,  
      asymmetryCompensationEnabled: true,  
      syncRatePacketsPerSec: 64,  
      vlanIsolationEnabled: true,  
      vlanId: 100,  
      grandmasterPriority1: 128,  
      grandmasterPriority2: 128,  
      grandmasterClockClass: 6,  
      grandmasterClockAccuracy: 32,  
      timeTraceable: true,  
      frequencyTraceable: true,  
      oscillatorType: 'OCXO',  
      cosValue: 7,  
      dscpValue: 46,  
      interfaceName: 'TenGigE0/0/0/1'  
    },  
    'datacenter-hft-g8275-2-arista': {  
      nodeName: 'HFT\_EQUINIX\_LD4\_PTP',  
      clockRole: 'BOUNDARY\_CLOCK',  
      slaveOnly: false,  
      masterGuardEnabled: true,  
      telecomProfile: 'G\_8275\_2',  
      transportType: 'IP\_UDP\_UNICAST',  
      domainNumber: 44,  
      timestampingMode: 'HARDWARE\_PHY',  
      twoStep: false,  
      fiberAsymmetryNs: 12,  
      asymmetryCompensationEnabled: true,  
      syncRatePacketsPerSec: 32,  
      vlanIsolationEnabled: true,  
      vlanId: 200,  
      grandmasterPriority1: 128,  
      grandmasterPriority2: 128,  
      grandmasterClockClass: 6,  
      grandmasterClockAccuracy: 32,  
      timeTraceable: true,  
      frequencyTraceable: true,  
      oscillatorType: 'RUBIDIUM',  
      cosValue: 7,  
      dscpValue: 56,  
      interfaceName: 'Ethernet1'  
    },  
    'vulnerable-flapping-rogue-master': {  
      nodeName: 'UNPROTECTED\_ROGUE\_CLOCK',  
      clockRole: 'ORDINARY\_CLOCK',  
      slaveOnly: false,  
      masterGuardEnabled: false,  
      telecomProfile: 'G\_8275\_1',  
      transportType: 'IP\_UDP\_UNICAST',  
      domainNumber: 0,  
      timestampingMode: 'SOFTWARE\_HOST',  
      twoStep: true,  
      fiberAsymmetryNs: 0,  
      asymmetryCompensationEnabled: false,  
      syncRatePacketsPerSec: 1,  
      vlanIsolationEnabled: false,  
      vlanId: 1,  
      grandmasterPriority1: 10,  
      grandmasterPriority2: 10,  
      grandmasterClockClass: 248,  
      grandmasterClockAccuracy: 254,  
      timeTraceable: false,  
      frequencyTraceable: false,  
      oscillatorType: 'TCXO',  
      cosValue: 0,  
      dscpValue: 0,  
      interfaceName: 'eth0'  
    }  
  };

  let activeFormat \= 'cisco';

  function getFormData() {  
    return {  
      nodeName: document.getElementById('nodeName').value,  
      clockRole: document.getElementById('clockRole').value,  
      telecomProfile: document.getElementById('telecomProfile').value,  
      transportType: document.getElementById('transportType').value,  
      domainNumber: parseInt(document.getElementById('domainNumber').value, 10\) || 24,  
      syncRatePacketsPerSec: parseInt(document.getElementById('syncRatePacketsPerSec').value, 10\) || 16,  
      timestampingMode: document.getElementById('timestampingMode').value,  
      oscillatorType: document.getElementById('oscillatorType').value,  
      fiberAsymmetryNs: parseFloat(document.getElementById('fiberAsymmetryNs').value) || 0,  
      grandmasterPriority1: parseInt(document.getElementById('grandmasterPriority1').value, 10\) || 128,  
      grandmasterPriority2: 128,  
      grandmasterClockClass: parseInt(document.getElementById('grandmasterClockClass').value, 10\) || 6,  
      grandmasterClockAccuracy: 32,  
      interfaceName: document.getElementById('interfaceName').value || 'eth0',  
      twoStep: document.getElementById('twoStep').checked,  
      masterGuardEnabled: document.getElementById('masterGuardEnabled').checked,  
      asymmetryCompensationEnabled: document.getElementById('asymmetryCompensationEnabled').checked,  
      vlanIsolationEnabled: document.getElementById('vlanIsolationEnabled').checked,  
      vlanId: document.getElementById('vlanIsolationEnabled').checked ? 100 : 1,  
      timeTraceable: document.getElementById('timeTraceable').checked,  
      frequencyTraceable: document.getElementById('timeTraceable').checked,  
      slaveOnly: document.getElementById('slaveOnly').checked,  
      cosValue: 7,  
      dscpValue: 46  
    };  
  }

  function applyDataToForm(data) {  
    if (\!data) return;  
    document.getElementById('nodeName').value \= data.nodeName || '';  
    document.getElementById('clockRole').value \= data.clockRole || 'TELECOM\_BOUNDARY\_CLOCK';  
    document.getElementById('telecomProfile').value \= data.telecomProfile || 'G\_8275\_1';  
    document.getElementById('transportType').value \= data.transportType || 'ETHERNET\_L2';  
    document.getElementById('domainNumber').value \= data.domainNumber \!== undefined ? data.domainNumber : 24;  
    document.getElementById('syncRatePacketsPerSec').value \= data.syncRatePacketsPerSec || 16;  
    document.getElementById('timestampingMode').value \= data.timestampingMode || 'HARDWARE\_PHY';  
    document.getElementById('oscillatorType').value \= data.oscillatorType || 'OCXO';  
    document.getElementById('fiberAsymmetryNs').value \= data.fiberAsymmetryNs || 0;  
    document.getElementById('grandmasterPriority1').value \= data.grandmasterPriority1 \!== undefined ? data.grandmasterPriority1 : 128;  
    document.getElementById('grandmasterClockClass').value \= data.grandmasterClockClass || 6;  
    document.getElementById('interfaceName').value \= data.interfaceName || 'TenGigE0/0/0/1';  
    document.getElementById('twoStep').checked \= \!\!data.twoStep;  
    document.getElementById('masterGuardEnabled').checked \= \!\!data.masterGuardEnabled;  
    document.getElementById('asymmetryCompensationEnabled').checked \= \!\!data.asymmetryCompensationEnabled;  
    document.getElementById('vlanIsolationEnabled').checked \= \!\!data.vlanIsolationEnabled;  
    document.getElementById('timeTraceable').checked \= \!\!data.timeTraceable;  
    document.getElementById('slaveOnly').checked \= \!\!data.slaveOnly;  
    renderAll();  
  }

  function renderAudit(data) {  
    const auditRes \= PTPSecurityAuditor.auditPtpConfiguration(data);  
    const scoreBadge \= document.getElementById('scoreBadge');  
    const headerScoreBadge \= document.getElementById('headerScoreBadge');  
    const scoreText \= document.getElementById('scoreText');  
    const scoreSummary \= document.getElementById('scoreSummary');  
    const scoreBanner \= document.getElementById('scoreBanner');  
    const auditList \= document.getElementById('auditList');

    scoreBadge.textContent \= auditRes.grade;  
    headerScoreBadge.textContent \= \`SCORE: \${auditRes.score}/100 (\${auditRes.grade})\`;  
    scoreText.textContent \= \`Zero-Trust PTP Score: \${auditRes.score} / 100\`;

    if (auditRes.grade \=== 'A+' || auditRes.grade \=== 'A') {  
      scoreBadge.style.color \= '\#38bdf8';  
      scoreBadge.style.borderColor \= '\#38bdf8';  
      scoreBadge.style.background \= 'rgba(56, 189, 248, 0.2)';  
      scoreBanner.style.background \= 'rgba(56, 189, 248, 0.1)';  
      scoreBanner.style.borderColor \= 'rgba(56, 189, 248, 0.3)';  
      scoreSummary.textContent \= 'Hardware PHY Timestamping • Master Guard Guarding • G.8275.1 Profile • Class 6 PRTC';  
    } else if (auditRes.grade \=== 'B' || auditRes.grade \=== 'C') {  
      scoreBadge.style.color \= '\#f59e0b';  
      scoreBadge.style.borderColor \= '\#f59e0b';  
      scoreBadge.style.background \= 'rgba(245, 158, 11, 0.2)';  
      scoreBanner.style.background \= 'rgba(245, 158, 11, 0.1)';  
      scoreBanner.style.borderColor \= 'rgba(245, 158, 11, 0.3)';  
      scoreSummary.textContent \= 'Warning: Low sync rate or uncompensated fiber asymmetry may cause wander accumulation.';  
    } else {  
      scoreBadge.style.color \= '\#ef4444';  
      scoreBadge.style.borderColor \= '\#ef4444';  
      scoreBadge.style.background \= 'rgba(239, 68, 68, 0.2)';  
      scoreBanner.style.background \= 'rgba(239, 68, 68, 0.1)';  
      scoreBanner.style.borderColor \= 'rgba(239, 68, 68, 0.3)';  
      scoreSummary.textContent \= 'CRITICAL: Rogue master BMCA vulnerability, free-running oscillator, and software timestamping noise.';  
    }

    auditList.innerHTML \= '';  
    auditRes.findings.forEach(f \=\> {  
      const item \= document.createElement('div');  
      item.className \= \`audit-item \${f.status}\`;  
      item.innerHTML \= \`  
        \<div\>  
          \<strong\>\${f.title}\</strong\>  
          \<p style="font-size: 0.75rem; color: \#94a3b8; margin-top: 2px;"\>\${f.desc}\</p\>  
        \</div\>  
        \<span style="font-weight: 700; font-family: monospace;"\>\${f.status} (\${f.points} pts)\</span\>  
      \`;  
      auditList.appendChild(item);  
    });  
  }

  function renderHex(data) {  
    const pktBytes \= PTPEngine.encodeAnnouncePacket({  
      domainNumber: data.domainNumber,  
      twoStep: data.twoStep,  
      grandmasterPriority1: data.grandmasterPriority1,  
      grandmasterClockClass: data.grandmasterClockClass,  
      grandmasterClockAccuracy: data.grandmasterClockAccuracy,  
      timeTraceable: data.timeTraceable,  
      frequencyTraceable: data.frequencyTraceable,  
      ptpTimescale: true  
    });  
    const hexBox \= document.getElementById('hexBox');  
    const dump \= PTPEngine.formatHexDump(pktBytes);

    hexBox.textContent \= \`// IEEE 1588v2 Announce Message (\${pktBytes.length} Bytes, Domain \${data.domainNumber}):\\n\` \+ dump;  
  }

  function renderConfig(data) {  
    const configOutput \= document.getElementById('configOutput');  
    let code \= '';

    switch (activeFormat) {  
      case 'cisco':  
        code \= PTPConfigSynthesizer.synthesizeCisco(data);  
        break;  
      case 'junos':  
        code \= PTPConfigSynthesizer.synthesizeJunos(data);  
        break;  
      case 'arista':  
        code \= PTPConfigSynthesizer.synthesizeArista(data);  
        break;  
      case 'linux':  
        code \= PTPConfigSynthesizer.synthesizeLinuxPtp(data);  
        break;  
      case 'prom':  
        code \= PTPConfigSynthesizer.synthesizePrometheusExporter(data);  
        break;  
      default:  
        code \= '\! Select a format';  
    }

    configOutput.textContent \= code;  
  }

  function renderAll() {  
    const data \= getFormData();  
    renderAudit(data);  
    renderHex(data);  
    renderConfig(data);  
  }

  // Event Listeners  
  document.getElementById('presetSelect').addEventListener('change', (e) \=\> {  
    const presetKey \= e.target.value;  
    if (PRESETS\[presetKey\]) {  
      applyDataToForm(PRESETS\[presetKey\]);  
    }  
  });

  document.getElementById('btnResetPreset').addEventListener('click', () \=\> {  
    const presetKey \= document.getElementById('presetSelect').value;  
    if (PRESETS\[presetKey\]) {  
      applyDataToForm(PRESETS\[presetKey\]);  
    }  
  });

  document.getElementById('btnRecalculate').addEventListener('click', renderAll);

  // Form Inputs live re-render  
  document.querySelectorAll('input, select').forEach(elem \=\> {  
    elem.addEventListener('input', renderAll);  
    elem.addEventListener('change', renderAll);  
  });

  // Synthesizer Tabs  
  document.querySelectorAll('.tab-btn').forEach(btn \=\> {  
    btn.addEventListener('click', () \=\> {  
      document.querySelectorAll('.tab-btn').forEach(b \=\> b.classList.remove('active'));  
      btn.classList.add('active');  
      activeFormat \= btn.getAttribute('data-format');  
      renderConfig(getFormData());  
    });  
  });

  // Copy Buttons  
  document.getElementById('btnCopyHex').addEventListener('click', () \=\> {  
    const hex \= document.getElementById('hexBox').textContent;  
    navigator.clipboard.writeText(hex).then(() \=\> {  
      alert('PTP packet hex dump copied to clipboard\!');  
    });  
  });

  document.getElementById('btnCopyConfig').addEventListener('click', () \=\> {  
    const cfg \= document.getElementById('configOutput').textContent;  
    navigator.clipboard.writeText(cfg).then(() \=\> {  
      alert('Synthesized PTP configuration copied to clipboard\!');  
    });  
  });

  // Initial render  
  renderAll();  
});  
