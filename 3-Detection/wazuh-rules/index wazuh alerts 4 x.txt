{
  "_index": "wazuh-alerts-4.x-2026.06.25",
  "_id": "GGh__Z4B-SmZWk_t4PVb",
  "_score": null,
  "_source": {
    "input": {
      "type": "log"
    },
    "agent": {
      "ip": "192.168.128.162",
      "name": "Nayak_RO",
      "id": "002"
    },
    "manager": {
      "name": "nayak-VMware-Virtual-Platform"
    },
    "data": {
      "win": {
        "eventdata": {
          "originalFileName": "Cmd.Exe",
          "image": "C:\\\\Windows\\\\System32\\\\cmd.exe",
          "product": "Microsoft® Windows® Operating System",
          "parentProcessGuid": "{e1ca0067-ccba-6a3c-ba03-000000004200}",
          "description": "Windows Command Processor",
          "logonGuid": "{e1ca0067-c131-6a3c-e703-000000000000}",
          "parentCommandLine": "\\\"C:\\\\Program Files\\\\SplunkUniversalForwarder\\\\bin\\\\splunkd.exe\\\" service",
          "processGuid": "{e1ca0067-ccc0-6a3c-f203-000000004200}",
          "logonId": "0x3e7",
          "parentProcessId": "1156",
          "processId": "1708",
          "currentDirectory": "C:\\\\WINDOWS\\\\system32\\\\",
          "utcTime": "2026-06-25 06:37:52.517",
          "hashes": "MD5=D87E9FA4C7C878F4C1DC8885CEA7F9F0,SHA256=75320A519959CC6D089EA3EBA33C38CACCB7F138A025EA439BC9686CDB79DED4,IMPHASH=B0F049C014592B156EB1FA857E99CEB9",
          "parentImage": "C:\\\\Program Files\\\\SplunkUniversalForwarder\\\\bin\\\\splunkd.exe",
          "company": "Microsoft Corporation",
          "commandLine": "C:\\\\WINDOWS\\\\system32\\\\cmd.exe /c \\\"\\\"C:\\\\Program Files\\\\SplunkUniversalForwarder\\\\etc\\\\system\\\\bin\\\\WinRegMon.cmd\\\" --scheme\\\"",
          "integrityLevel": "System",
          "fileVersion": "10.0.26100.8521 (WinBuild.160101.0800)",
          "user": "NT AUTHORITY\\\\SYSTEM",
          "terminalSessionId": "0",
          "parentUser": "NT AUTHORITY\\\\SYSTEM"
        },
        "system": {
          "eventID": "1",
          "keywords": "0x8000000000000000",
          "providerGuid": "{5770385f-c22a-43e0-bf4c-06f5698ffbd9}",
          "level": "4",
          "channel": "Microsoft-Windows-Sysmon/Operational",
          "opcode": "0",
          "message": "\"Process Create:\r\nRuleName: -\r\nUtcTime: 2026-06-25 06:37:52.517\r\nProcessGuid: {e1ca0067-ccc0-6a3c-f203-000000004200}\r\nProcessId: 1708\r\nImage: C:\\Windows\\System32\\cmd.exe\r\nFileVersion: 10.0.26100.8521 (WinBuild.160101.0800)\r\nDescription: Windows Command Processor\r\nProduct: Microsoft® Windows® Operating System\r\nCompany: Microsoft Corporation\r\nOriginalFileName: Cmd.Exe\r\nCommandLine: C:\\WINDOWS\\system32\\cmd.exe /c \"\"C:\\Program Files\\SplunkUniversalForwarder\\etc\\system\\bin\\WinRegMon.cmd\" --scheme\"\r\nCurrentDirectory: C:\\WINDOWS\\system32\\\r\nUser: NT AUTHORITY\\SYSTEM\r\nLogonGuid: {e1ca0067-c131-6a3c-e703-000000000000}\r\nLogonId: 0x3E7\r\nTerminalSessionId: 0\r\nIntegrityLevel: System\r\nHashes: MD5=D87E9FA4C7C878F4C1DC8885CEA7F9F0,SHA256=75320A519959CC6D089EA3EBA33C38CACCB7F138A025EA439BC9686CDB79DED4,IMPHASH=B0F049C014592B156EB1FA857E99CEB9\r\nParentProcessGuid: {e1ca0067-ccba-6a3c-ba03-000000004200}\r\nParentProcessId: 1156\r\nParentImage: C:\\Program Files\\SplunkUniversalForwarder\\bin\\splunkd.exe\r\nParentCommandLine: \"C:\\Program Files\\SplunkUniversalForwarder\\bin\\splunkd.exe\" service\r\nParentUser: NT AUTHORITY\\SYSTEM\"",
          "version": "5",
          "systemTime": "2026-06-25T06:37:52.5231428Z",
          "eventRecordID": "38378",
          "threadID": "4956",
          "computer": "Nayak_RO",
          "task": "1",
          "processID": "3480",
          "severityValue": "INFORMATION",
          "providerName": "Microsoft-Windows-Sysmon"
        }
      }
    },
    "rule": {
      "firedtimes": 17,
      "mail": false,
      "level": 4,
      "description": "Windows command prompt started by an abnormal process",
      "groups": [
        "sysmon",
        "sysmon_eid1_detections",
        "windows"
      ],
      "mitre": {
        "technique": [
          "Windows Command Shell"
        ],
        "id": [
          "T1059.003"
        ],
        "tactic": [
          "Execution"
        ]
      },
      "id": "92052"
    },
    "location": "EventChannel",
    "decoder": {
      "name": "windows_eventchannel"
    },
    "id": "1782369471.936451",
    "timestamp": "2026-06-25T12:07:51.715+0530"
  },
  "fields": {
    "timestamp": [
      "2026-06-25T06:37:51.715Z"
    ]
  },
  "sort": [
    1782369471715
  ]
}
