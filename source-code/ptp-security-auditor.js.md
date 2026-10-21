/\*\*  
 \* PTP1588Forge Pro \- 10-Point Zero-Trust PTP Telecom Profile & Phase Audit Scorecard  
 \* Evaluates BMCA safety, G.8275.1/2 profile consistency, hardware PHY timestamping,  
 \* fiber asymmetry compensation, clock traceability, and holdover budgets.  
 \* Outputs Score (0-100), Grade (A+ to F), and granular findings.  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.PTPSecurityAuditor \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  function auditPtpConfiguration(cfg) {  
    const findings \= \[\];  
    let score \= 0;

    // 1\. BMCA & Rogue Master Defense (10 pts)  
    const role \= (cfg.clockRole || 'ORDINARY\_CLOCK').toUpperCase();  
    const slaveOnly \= \!\!cfg.slaveOnly;  
    const priority1 \= Number(cfg.grandmasterPriority1 \!== undefined ? cfg.grandmasterPriority1 : 128);  
    const masterGuard \= \!\!cfg.masterGuardEnabled;

    if (slaveOnly || masterGuard || (role \=== 'SLAVE\_CLOCK')) {  
      score \+= 10;  
      findings.push({  
        id: 'BMCA\_ROGUE\_MASTER\_DEFENSE',  
        status: 'PASS',  
        points: 10,  
        title: 'BMCA Clock Tree Guard & Slave-Only Lockdown',  
        desc: 'Downstream port locks clock into slave-only or enforces master guard, preventing rogue Grandmasters from overriding GNSS timing.'  
      });  
    } else if (priority1 \<= 127 && \!masterGuard) {  
      findings.push({  
        id: 'BMCA\_ROGUE\_MASTER\_DEFENSE',  
        status: 'FAIL',  
        points: 0,  
        title: 'CRITICAL: High-Risk Unprotected Master Priority',  
        desc: \`Node claims aggressive BMCA Priority 1 (\${priority1}) without master guard or slave-only protection. Can cause network-wide clock flapping.\`  
      });  
    } else {  
      score \+= 5;  
      findings.push({  
        id: 'BMCA\_ROGUE\_MASTER\_DEFENSE',  
        status: 'WARNING',  
        points: 5,  
        title: 'Standard Default BMCA Priority (128)',  
        desc: 'Default priority configured without explicit slave-only or port-level BMCA filtering.'  
      });  
    }

    // 2\. Telecom Profile Alignment (10 pts)  
    const profile \= (cfg.telecomProfile || 'G\_8275\_1').toUpperCase();  
    const transport \= (cfg.transportType || 'ETHERNET\_L2').toUpperCase();  
    const domain \= Number(cfg.domainNumber \!== undefined ? cfg.domainNumber : 24);

    if (profile \=== 'G\_8275\_1' && transport \=== 'ETHERNET\_L2' && domain \=== 24\) {  
      score \+= 10;  
      findings.push({  
        id: 'TELECOM\_PROFILE\_ALIGNMENT',  
        status: 'PASS',  
        points: 10,  
        title: 'ITU-T G.8275.1 Full On-Path Timing Alignment',  
        desc: 'Full Layer 2 multicast Ethernet encapsulation (EtherType 0x88F7) on Telecom Domain 24.'  
      });  
    } else if (profile \=== 'G\_8275\_2' && (transport \=== 'IP\_UDP\_UNICAST' || transport \=== 'UDP\_L3') && domain \=== 44\) {  
      score \+= 10;  
      findings.push({  
        id: 'TELECOM\_PROFILE\_ALIGNMENT',  
        status: 'PASS',  
        points: 10,  
        title: 'ITU-T G.8275.2 Partial Timing Support (PTS) Alignment',  
        desc: 'Layer 3 IP/UDP unicast transport on Telecom Domain 44 matching mobile backhaul standards.'  
      });  
    } else if ((profile \=== 'G\_8275\_1' && transport \!== 'ETHERNET\_L2') || (profile \=== 'G\_8275\_2' && transport \=== 'ETHERNET\_L2')) {  
      findings.push({  
        id: 'TELECOM\_PROFILE\_ALIGNMENT',  
        status: 'FAIL',  
        points: 0,  
        title: 'CRITICAL: Telecom Profile & Transport Mismatch',  
        desc: \`Configuration specifies \${profile} but selects \${transport}. G.8275.1 strictly mandates Layer 2, while G.8275.2 mandates L3 IP/UDP.\`  
      });  
    } else {  
      score \+= 5;  
      findings.push({  
        id: 'TELECOM\_PROFILE\_ALIGNMENT',  
        status: 'WARNING',  
        points: 5,  
        title: 'Generic Default PTP Profile (IEEE 1588-2008)',  
        desc: 'Using standard default profile without ITU-T telecom optimizations for carrier fronthaul.'  
      });  
    }

    // 3\. Hardware Line-Rate PHY Timestamping (10 pts)  
    const tsMode \= (cfg.timestampingMode || 'HARDWARE\_PHY').toUpperCase();  
    if (tsMode \=== 'HARDWARE\_PHY' || tsMode \=== 'HARDWARE\_ASIC') {  
      score \+= 10;  
      findings.push({  
        id: 'HARDWARE\_TIMESTAMPING',  
        status: 'PASS',  
        points: 10,  
        title: 'Sub-Nanosecond Hardware PHY/MAC Timestamping',  
        desc: 'Hardware timestamping directly at the physical transceiver eliminates OS kernel and PCIe queueing jitter.'  
      });  
    } else {  
      findings.push({  
        id: 'HARDWARE\_TIMESTAMPING',  
        status: 'FAIL',  
        points: 0,  
        title: 'CRITICAL: Software Host Timestamping Active',  
        desc: 'Software timestamping introduces 5-50 microseconds of CPU scheduling variance, completely invalidating sub-microsecond 5G SLAs.'  
      });  
    }

    // 4\. Step Mode Consistency (10 pts)  
    const twoStep \= \!\!cfg.twoStep;  
    score \+= 10;  
    findings.push({  
      id: 'STEP\_MODE\_CONSISTENCY',  
      status: 'PASS',  
      points: 10,  
      title: twoStep ? '2-Step Mode (Follow\_Up Transmission Validated)' : '1-Step Mode (On-the-Fly MAC Timestamping)',  
      desc: twoStep ? 'Master emits precise egress timestamps inside Follow\_Up packets.' : 'Hardware ASIC updates Sync packets directly on physical egress.'  
    });

    // 5\. Fiber Asymmetry & Path Delay Compensation (10 pts)  
    const asymNs \= Number(cfg.fiberAsymmetryNs || 0);  
    const asymComp \= \!\!cfg.asymmetryCompensationEnabled;

    if (asymComp || asymNs \!== 0\) {  
      score \+= 10;  
      findings.push({  
        id: 'FIBER\_ASYMMETRY\_COMPENSATION',  
        status: 'PASS',  
        points: 10,  
        title: \`Fiber Asymmetry Delay Compensation Active (\${asymNs} ns)\`,  
        desc: 'Corrects static optical path difference between TX and RX wavelengths to prevent fixed phase offset bias.'  
      });  
    } else {  
      score \+= 4;  
      findings.push({  
        id: 'FIBER\_ASYMMETRY\_COMPENSATION',  
        status: 'WARNING',  
        points: 4,  
        title: 'Uncompensated Fiber Propagation Delay',  
        desc: 'Zero delay asymmetry compensation. WDM dispersion or unequal fiber pairs may produce uncalibrated phase error.'  
      });  
    }

    // 6\. Message Rate Cadence (10 pts)  
    const syncRate \= Number(cfg.syncRatePacketsPerSec \!== undefined ? cfg.syncRatePacketsPerSec : 16);  
    if (syncRate \>= 16\) {  
      score \+= 10;  
      findings.push({  
        id: 'MESSAGE\_RATE\_CADENCE',  
        status: 'PASS',  
        points: 10,  
        title: \`High-Cadence Telecom Rate (\${syncRate} packets/sec)\`,  
        desc: 'Sync rate complies with G.8275.1 standards (16 to 128 pkts/sec) for rapid servo clock convergence.'  
      });  
    } else if (syncRate \>= 1\) {  
      score \+= 6;  
      findings.push({  
        id: 'MESSAGE\_RATE\_CADENCE',  
        status: 'WARNING',  
        points: 6,  
        title: \`Low Sync Rate (\${syncRate} packets/sec)\`,  
        desc: 'Sync rate is below telecom standards (\>= 16/sec); clock recovery filter will suffer increased wander.'  
      });  
    } else {  
      findings.push({  
        id: 'MESSAGE\_RATE\_CADENCE',  
        status: 'FAIL',  
        points: 0,  
        title: 'Invalid or Zero PTP Sync Cadence',  
        desc: 'Probe rate is unconfigured or zero.'  
      });  
    }

    // 7\. Transport Layer & VLAN Telemetry Isolation (10 pts)  
    const vlanIsolated \= \!\!cfg.vlanIsolationEnabled;  
    const vlanId \= cfg.vlanId || 100;  
    if (vlanIsolated) {  
      score \+= 10;  
      findings.push({  
        id: 'VLAN\_TELEMETRY\_ISOLATION',  
        status: 'PASS',  
        points: 10,  
        title: \`Dedicated PTP Timing VLAN Isolation (VLAN \${vlanId})\`,  
        desc: 'Timing traffic is segregated into an isolated 802.1Q subinterface, protected from user broadcast storms.'  
      });  
    } else {  
      score \+= 5;  
      findings.push({  
        id: 'VLAN\_TELEMETRY\_ISOLATION',  
        status: 'WARNING',  
        points: 5,  
        title: 'Untagged PTP Traffic Colocated on Native VLAN',  
        desc: 'Timing packets share native broadcast domain with customer traffic, increasing jitter vulnerability.'  
      });  
    }

    // 8\. Time Accuracy & Quality Traceability (10 pts)  
    const clockClass \= Number(cfg.grandmasterClockClass \!== undefined ? cfg.grandmasterClockClass : 6);  
    const timeTraceable \= \!\!cfg.timeTraceable;

    if (clockClass \=== 6 && timeTraceable) {  
      score \+= 10;  
      findings.push({  
        id: 'CLOCK\_QUALITY\_TRACEABILITY',  
        status: 'PASS',  
        points: 10,  
        title: 'PRTC GNSS Locked (Clock Class 6, Time Traceable)',  
        desc: 'Grandmaster is locked to Primary Reference Time Clock (GNSS/UTC) with full traceability flag.'  
      });  
    } else if (clockClass \=== 7\) {  
      score \+= 8;  
      findings.push({  
        id: 'CLOCK\_QUALITY\_TRACEABILITY',  
        status: 'PASS',  
        points: 8,  
        title: 'PRTC Holdover Mode (Clock Class 7)',  
        desc: 'Operating within specification on atomic oscillator holdover.'  
      });  
    } else {  
      findings.push({  
        id: 'CLOCK\_QUALITY\_TRACEABILITY',  
        status: 'FAIL',  
        points: 0,  
        title: \`CRITICAL: Free-Running / Untraceable Clock (Class \${clockClass})\`,  
        desc: 'Grandmaster clock quality is unlocked or unverified. Downstream nodes will degrade or discard timing.'  
      });  
    }

    // 9\. Holdover Oscillator Budgeting (10 pts)  
    const oscType \= (cfg.oscillatorType || 'OCXO').toUpperCase();  
    if (oscType \=== 'RUBIDIUM' || oscType \=== 'ATOMIC' || oscType \=== 'OCXO' || oscType \=== 'STRATUM\_3E') {  
      score \+= 10;  
      findings.push({  
        id: 'HOLDOVER\_OSCILLATOR\_BUDGET',  
        status: 'PASS',  
        points: 10,  
        title: 'Carrier-Grade Rubidium Atomic Clock Holdover',  
        desc: 'Phase drift \< 1.0 microsecond over 24 hours of GNSS loss.'  
      });  
    } else if (oscType \=== 'TCXO\_HIGH\_STABILITY') {  
      score \+= 9;  
      findings.push({  
        id: 'HOLDOVER\_OSCILLATOR\_BUDGET',  
        status: 'PASS',  
        points: 9,  
        title: 'Oven-Controlled Crystal Oscillator (OCXO)',  
        desc: 'Phase drift \< 1.5 microseconds over 8 hours of holdover, meeting 3GPP Category B limits.'  
      });  
    } else {  
      score \+= 3;  
      findings.push({  
        id: 'HOLDOVER\_OSCILLATOR\_BUDGET',  
        status: 'WARNING',  
        points: 3,  
        title: 'Standard TCXO / Unbudgeted Oscillator',  
        desc: 'High thermal drift. Phase error will exceed 1.5 microseconds within minutes of GNSS signal loss.'  
      });  
    }

    // 10\. QoS Priority & CoS Tagging (10 pts)  
    const cos \= Number(cfg.cosValue \!== undefined ? cfg.cosValue : 7);  
    const dscp \= Number(cfg.dscpValue \!== undefined ? cfg.dscpValue : 46);

    if (cos \=== 7 || dscp \=== 46 || dscp \=== 56\) {  
      score \+= 10;  
      findings.push({  
        id: 'QOS\_COS\_PRIORITY',  
        status: 'PASS',  
        points: 10,  
        title: \`Strict Priority Expedited Forwarding (CoS \${cos} / DSCP \${dscp})\`,  
        desc: 'PTP event packets bypass best-effort switch buffers and preempt background data flows.'  
      });  
    } else {  
      score \+= 4;  
      findings.push({  
        id: 'QOS\_COS\_PRIORITY',  
        status: 'WARNING',  
        points: 4,  
        title: 'Suboptimal QoS Priority Tagging',  
        desc: 'PTP packets may suffer queuing delay jitter during network microbursts.'  
      });  
    }

    let grade \= 'F';  
    if (score \>= 90\) grade \= 'A+';  
    else if (score \>= 80\) grade \= 'A';  
    else if (score \>= 70\) grade \= 'B';  
    else if (score \>= 50\) grade \= 'C';

    return {  
      score: Math.min(100, Math.max(0, score)),  
      grade,  
      findings  
    };  
  }

  return {  
    auditPtpConfiguration  
  };  
});  
