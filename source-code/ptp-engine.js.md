/\*\*  
 \* PTP1588Forge Pro \- IEEE 1588v2 Precision Time Protocol Engine  
 \* Standard 34-Byte Header, Announce, Sync, Follow\_Up, Delay\_Req, Delay\_Resp Encoders,  
 \* Sub-Nanosecond Phase Offset & Path Delay Math, and Formatted Hex Dumper.  
 \*/

(function (root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.PTPEngine \= factory();  
  }  
})(typeof self \!== 'undefined' ? self : this, function () {  
  'use strict';

  const MESSAGE\_TYPES \= {  
    SYNC: 0x0,  
    DELAY\_REQ: 0x1,  
    PDELAY\_REQ: 0x2,  
    PDELAY\_RESP: 0x3,  
    FOLLOW\_UP: 0x8,  
    DELAY\_RESP: 0x9,  
    PDELAY\_RESP\_FOLLOW\_UP: 0xA,  
    ANNOUNCE: 0xB,  
    SIGNALING: 0xC,  
    MANAGEMENT: 0xD  
  };

  const CLOCK\_CLASSES \= {  
    PRTC\_GPS: 6,         // Primary Reference Time Clock (ePRTC/PRTC GNSS locked)  
    PRTC\_HOLDOVER: 7,    // In holdover, within specification  
    TELECOM\_SLAVE: 135,  // T-BC in slave-only mode  
    DEFAULT\_UNLOCKED: 248 // Default free-running oscillator  
  };

  const TIME\_SOURCES \= {  
    ATOMIC\_CLOCK: 0x10,  
    GPS: 0x20,  
    TERRESTRIAL\_RADIO: 0x30,  
    PTP: 0x40,  
    NTP: 0x50,  
    HAND\_SET: 0x60,  
    OTHER: 0x90,  
    INTERNAL\_OSCILLATOR: 0xA0  
  };

  function uint16ToBytes(val) {  
    const v \= maxClamp(val, 0, 0xffff);  
    return \[(v \>\> 8\) & 0xff, v & 0xff\];  
  }

  function uint32ToBytes(val) {  
    const v \= maxClamp(val, 0, 0xffffffff);  
    return \[(v \>\> 24\) & 0xff, (v \>\> 16\) & 0xff, (v \>\> 8\) & 0xff, v & 0xff\];  
  }

  function int8ToByte(val) {  
    return (val \< 0 ? (256 \+ val) : val) & 0xff;  
  }

  function maxClamp(val, min, max) {  
    const n \= Number(val) || 0;  
    return Math.max(min, Math.min(max, Math.floor(n)));  
  }

  function parseClockIdentity(str) {  
    if (\!str) return \[0x00, 0x11, 0x22, 0xff, 0xfe, 0x33, 0x44, 0x55\];  
    const cleaned \= str.replace(/\[^0-9a-fA-F\]/g, '');  
    const bytes \= \[\];  
    for (let i \= 0; i \< 16; i \+= 2\) {  
      if (i \< cleaned.length) {  
        bytes.push(parseInt(cleaned.substr(i, 2), 16\) || 0);  
      } else {  
        bytes.push(0);  
      }  
    }  
    return bytes.length \=== 8 ? bytes : \[0x00, 0x11, 0x22, 0xff, 0xfe, 0x33, 0x44, 0x55\];  
  }

  function encodeTimestamp(seconds, nanoseconds) {  
    const sec \= BigInt(Math.floor(Number(seconds) || 0));  
    const ns \= maxClamp(nanoseconds, 0, 999999999);

    const b \= \[\];  
    // 48-bit seconds (6 bytes)  
    for (let i \= 5; i \>= 0; i--) {  
      b.push(Number((sec \>\> BigInt(i \* 8)) & 0xffn));  
    }  
    // 32-bit nanoseconds (4 bytes)  
    b.push(...uint32ToBytes(ns));  
    return b;  
  }

  /\*\*  
   \* Build 34-byte IEEE 1588v2 Common Header  
   \*/  
  function buildCommonHeader(opts) {  
    const msgType \= opts.messageType \!== undefined ? opts.messageType : MESSAGE\_TYPES.SYNC;  
    const transportSpecific \= (opts.transportSpecific || 0\) & 0x0f;  
    const byte0 \= (transportSpecific \<\< 4\) | (msgType & 0x0f);  
    const versionPTP \= 0x02; // IEEE 1588-2008 (v2)  
    const msgLength \= opts.messageLength || 34;  
    const domainNumber \= maxClamp(opts.domainNumber \!== undefined ? opts.domainNumber : 24, 0, 255); // G.8275.1 uses 24

    // Flag Field (16 bits)  
    let flags \= 0;  
    if (opts.twoStep) flags |= (1 \<\< 1); // twoStepFlag (bit 1\)  
    if (opts.unicast) flags |= (1 \<\< 2); // unicastFlag (bit 2\)  
    if (opts.ptpTimescale) flags |= (1 \<\< 11); // ptpTimescale (bit 11\)  
    if (opts.timeTraceable) flags |= (1 \<\< 12); // timeTraceable (bit 12\)  
    if (opts.frequencyTraceable) flags |= (1 \<\< 13); // frequencyTraceable (bit 13\)

    // Correction field: 64-bit nanosecond fraction (scaled by 2^16)  
    const corrNs \= BigInt(Math.floor(Number(opts.correctionFieldNs || 0\) \* 65536));  
    const corrBytes \= \[\];  
    for (let i \= 7; i \>= 0; i--) {  
      corrBytes.push(Number((corrNs \>\> BigInt(i \* 8)) & 0xffn));  
    }

    const clockIdBytes \= parseClockIdentity(opts.clockIdentity || '00-11-22-ff-fe-33-44-55');  
    const portNumberBytes \= uint16ToBytes(opts.portNumber || 1);  
    const seqIdBytes \= uint16ToBytes(opts.sequenceId || 1);

    let ctrl \= 0x05;  
    if (msgType \=== MESSAGE\_TYPES.SYNC) ctrl \= 0x00;  
    else if (msgType \=== MESSAGE\_TYPES.DELAY\_REQ) ctrl \= 0x01;  
    else if (msgType \=== MESSAGE\_TYPES.FOLLOW\_UP) ctrl \= 0x02;  
    else if (msgType \=== MESSAGE\_TYPES.DELAY\_RESP) ctrl \= 0x03;

    const logInterval \= int8ToByte(opts.logMessageInterval \!== undefined ? opts.logMessageInterval : \-4); // \-4 \= 16 pkts/s

    return \[  
      byte0,  
      versionPTP,  
      ...uint16ToBytes(msgLength),  
      domainNumber,  
      0x00, // reserved1  
      (flags \>\> 8\) & 0xff,  
      flags & 0xff,  
      ...corrBytes,  
      0x00, 0x00, 0x00, 0x00, // reserved2  
      ...clockIdBytes,  
      ...portNumberBytes,  
      ...seqIdBytes,  
      ctrl,  
      logInterval  
    \];  
  }

  /\*\*  
   \* Encode IEEE 1588v2 Announce Message (64 bytes total)  
   \*/  
  function encodeAnnouncePacket(opts \= {}) {  
    const totalLen \= 64;  
    const header \= buildCommonHeader(Object.assign({}, opts, {  
      messageType: MESSAGE\_TYPES.ANNOUNCE,  
      messageLength: totalLen  
    }));

    const originTs \= encodeTimestamp(opts.originSeconds || 1700000000, opts.originNanoseconds || 0);  
    const utcOffset \= uint16ToBytes(opts.currentUtcOffset || 37);  
    const priority1 \= maxClamp(opts.grandmasterPriority1 \!== undefined ? opts.grandmasterPriority1 : 128, 0, 255);  
    const clockClass \= maxClamp(opts.grandmasterClockClass \!== undefined ? opts.grandmasterClockClass : CLOCK\_CLASSES.PRTC\_GPS, 0, 255);  
    const clockAccuracy \= maxClamp(opts.grandmasterClockAccuracy \!== undefined ? opts.grandmasterClockAccuracy : 0x20, 0, 255); // 0x20 \= \<= 25ns  
    const variance \= uint16ToBytes(opts.offsetScaledLogVariance \!== undefined ? opts.offsetScaledLogVariance : 0x4B32);  
    const priority2 \= maxClamp(opts.grandmasterPriority2 \!== undefined ? opts.grandmasterPriority2 : 128, 0, 255);  
    const gmIdentity \= parseClockIdentity(opts.grandmasterIdentity || opts.clockIdentity || '00-11-22-ff-fe-33-44-55');  
    const stepsRemoved \= uint16ToBytes(opts.stepsRemoved || 0);  
    const timeSource \= maxClamp(opts.timeSource \!== undefined ? opts.timeSource : TIME\_SOURCES.GPS, 0, 255);

    const body \= \[  
      ...originTs,  
      ...utcOffset,  
      0x00, // reserved  
      priority1,  
      clockClass,  
      clockAccuracy,  
      ...variance,  
      priority2,  
      ...gmIdentity,  
      ...stepsRemoved,  
      timeSource  
    \];

    return header.concat(body);  
  }

  /\*\*  
   \* Encode IEEE 1588v2 Sync or Follow\_Up Message (44 bytes total)  
   \*/  
  function encodeSyncPacket(opts \= {}) {  
    const isFollowUp \= \!\!opts.isFollowUp;  
    const totalLen \= 44;  
    const header \= buildCommonHeader(Object.assign({}, opts, {  
      messageType: isFollowUp ? MESSAGE\_TYPES.FOLLOW\_UP : MESSAGE\_TYPES.SYNC,  
      messageLength: totalLen  
    }));

    const preciseOriginTs \= encodeTimestamp(opts.originSeconds || 1700000000, opts.originNanoseconds || 123456789);  
    return header.concat(preciseOriginTs);  
  }

  /\*\*  
   \* Encode IEEE 1588v2 Delay\_Req Message (44 bytes total)  
   \*/  
  function encodeDelayReqPacket(opts \= {}) {  
    const totalLen \= 44;  
    const header \= buildCommonHeader(Object.assign({}, opts, {  
      messageType: MESSAGE\_TYPES.DELAY\_REQ,  
      messageLength: totalLen  
    }));

    const originTs \= encodeTimestamp(opts.originSeconds || 1700000000, opts.originNanoseconds || 234567890);  
    return header.concat(originTs);  
  }

  /\*\*  
   \* Encode IEEE 1588v2 Delay\_Resp Message (54 bytes total)  
   \*/  
  function encodeDelayRespPacket(opts \= {}) {  
    const totalLen \= 54;  
    const header \= buildCommonHeader(Object.assign({}, opts, {  
      messageType: MESSAGE\_TYPES.DELAY\_RESP,  
      messageLength: totalLen  
    }));

    const recvTs \= encodeTimestamp(opts.receiveSeconds || 1700000000, opts.receiveNanoseconds || 234600123);  
    const reqClockId \= parseClockIdentity(opts.requestingClockIdentity || opts.clockIdentity || '00-11-22-ff-fe-33-44-55');  
    const reqPortNum \= uint16ToBytes(opts.requestingPortNumber || 1);

    return header.concat(recvTs).concat(reqClockId).concat(reqPortNum);  
  }

  /\*\*  
   \* Calculate Nanosecond Phase Offset, Mean Path Delay, and Asymmetry  
   \* @param {Object} ts \- { t1Ns, t2Ns, t3Ns, t4Ns, c1Ns, c2Ns, asymmetryNs }  
   \*/  
  function computePtpMetrics(ts \= {}) {  
    const t1 \= Number(ts.t1Ns || 0); // Master Sync Egress  
    const t2 \= Number(ts.t2Ns || 0); // Slave Sync Ingress  
    const t3 \= Number(ts.t3Ns || 0); // Slave Delay\_Req Egress  
    const t4 \= Number(ts.t4Ns || 0); // Master Delay\_Req Ingress  
    const c1 \= Number(ts.c1Ns || 0); // Sync Correction Field  
    const c2 \= Number(ts.c2Ns || 0); // Delay\_Resp Correction Field  
    const asym \= Number(ts.asymmetryNs || 0); // Fiber Delay Asymmetry (fwd \- rev)

    // Forward Delay \= (t2 \- t1 \- c1)  
    const fwdDelay \= t2 \- t1 \- c1;  
    // Reverse Delay \= (t4 \- t3 \- c2)  
    const revDelay \= t4 \- t3 \- c2;

    // Mean Path Delay \= ((t2 \- t1 \- c1) \+ (t4 \- t3 \- c2)) / 2  
    const meanPathDelayNs \= (fwdDelay \+ revDelay) / 2.0;

    // Offset \= ((t2 \- t1 \- c1) \- (t4 \- t3 \- c2) \- asym) / 2  
    const offsetFromMasterNs \= (fwdDelay \- revDelay \- asym) / 2.0;

    return {  
      offsetFromMasterNs: Math.round(offsetFromMasterNs \* 10\) / 10,  
      meanPathDelayNs: Math.round(meanPathDelayNs \* 10\) / 10,  
      fwdDelayNs: Math.round(fwdDelay \* 10\) / 10,  
      revDelayNs: Math.round(revDelay \* 10\) / 10,  
      asymmetryNs: asym,  
      timeErrorNs: Math.round(Math.abs(offsetFromMasterNs) \* 10\) / 10  
    };  
  }

  /\*\*  
   \* Format Byte Array into Wireshark / Tcpdump-Style Hex Dump  
   \*/  
  function formatHexDump(byteArr) {  
    const lines \= \[\];  
    for (let i \= 0; i \< byteArr.length; i \+= 16\) {  
      const chunk \= byteArr.slice(i, i \+ 16);  
      const offset \= i.toString(16).padStart(4, '0');  
      const hexParts \= \[\];  
      const asciiParts \= \[\];

      for (let j \= 0; j \< 16; j++) {  
        if (j \< chunk.length) {  
          const b \= chunk\[j\];  
          hexParts.push(b.toString(16).padStart(2, '0'));  
          asciiParts.push(b \>= 32 && b \<= 126 ? String.fromCharCode(b) : '.');  
        } else {  
          hexParts.push('  ');  
        }  
      }

      const hexStr \= hexParts.slice(0, 8).join(' ') \+ '  ' \+ hexParts.slice(8).join(' ');  
      lines.push(\`\${offset}:  \${hexStr.padEnd(49, ' ')} |\${asciiParts.join('')}|\`);  
    }  
    return lines.join('\\n');  
  }

  return {  
    MESSAGE\_TYPES,  
    CLOCK\_CLASSES,  
    TIME\_SOURCES,  
    buildCommonHeader,  
    encodeAnnouncePacket,  
    encodeSyncPacket,  
    encodeDelayReqPacket,  
    encodeDelayRespPacket,  
    computePtpMetrics,  
    formatHexDump,  
    parseClockIdentity  
  };  
});  
