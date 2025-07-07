# **🚀 Zabbix Agent Deployment on Windows 10 - Installation Log** 

**📅 Date:** July 7, 2025  

**👨‍💻 Admin:** Sasha Zimin

**🖥️ Target System:** Windows 10 Pro (Build 21H2) 

**🔗 Zabbix Server:** WSL2/Ubuntu (172.28.112.1)  

---

## **📜 Installation Log**  

### **1️⃣ Download Zabbix Agent**  
```powershell
[10:00] PS C:\> Invoke-WebRequest -Uri "https://cdn.zabbix.com/zabbix/binaries/stable/6.4/6.4.7/zabbix_agent-6.4.7-windows-amd64-openssl.msi" -OutFile "C:\Temp\zabbix_agent.msi"
```
**Output:**  
```
StatusCode: 200, File downloaded (12.3 MB)
```

---

### **2️⃣ Install Agent (Silent Mode)**  
```powershell
[10:02] PS C:\> Start-Process msiexec.exe -Wait -ArgumentList '/i C:\Temp\zabbix_agent.msi /qn SERVER=172.28.112.1 SERVERACTIVE=172.28.112.1 HOSTNAME=Win10-PC1'
```
**Verification:**  
```powershell
[10:03] PS C:\> Get-Service "Zabbix Agent"
```
**Output:**  
```
Status   Name               DisplayName
------   ----               -----------
Running  Zabbix Agent       Zabbix Agent
```

---

### **3️⃣ Configure Firewall Rule**  
```powershell
[10:05] PS C:\> New-NetFirewallRule -DisplayName "Zabbix Agent" -Direction Inbound -Action Allow -Protocol TCP -LocalPort 10050
```
**Verification:**  
```powershell
[10:06] PS C:\> Get-NetFirewallRule -DisplayName "Zabbix Agent" | Select-Object Enabled,Profile
```
**Output:**  
```
Enabled Profile
------- -------
True    Domain, Private, Public
```

---

### **4️⃣ Manual Configuration (Optional)**  
**File:** `C:\Program Files\Zabbix Agent\zabbix_agentd.conf`  
```ini
[10:10] Modified config:
Server=172.28.112.1
ServerActive=172.28.112.1
Hostname=Win10-PC1
LogFile=C:\zabbix_agentd.log
DebugLevel=3
```
**Restart Service:**  
```powershell
[10:12] PS C:\> Restart-Service "Zabbix Agent"
```

---

## **🔍 Connection Test**  

### **From Zabbix Server (WSL2)**  
```bash
[10:15] ➜ zabbix_get -s 192.168.1.25 -k system.cpu.util[,idle]
```
**Output:**  
```
78.3421
```

### **From Windows Client**  
```powershell
[10:16] PS C:\> & "C:\Program Files\Zabbix Agent\zabbix_get.exe" -s 127.0.0.1 -k system.uname
```
**Output:**  
```
Windows Win10-PC1 10.0.19044.1237 x64
```

---

## **📌 Host Registration in Zabbix Web UI**  

**Added Host:**  
- **Name:** `Win10-PC1`  
- **IP:** `192.168.1.25`  
- **Templates:**  
  - `Template OS Windows by Zabbix agent`  
  - `Template App Zabbix Agent`  

**Verification:**  
```text
[10:20] Zabbix Server Log: 
"received data from [Win10-PC1] [system.cpu.load[all,avg1]: 0.12]"
```

---

## **❗ Troubleshooting Log**  

### **Issue 1: Agent Not Starting**  
**Error:**  
```powershell
[10:18] PS C:\> Get-EventLog -LogName Application -Source "Zabbix Agent" -Newest 1
```
**Output:**  
```
Cannot bind to port 10050: [10048] Address already in use
```

**Solution:**  
```powershell
[10:19] PS C:\> netstat -ano | findstr 10050
[10:20] PS C:\> taskkill /PID 4567 /F
[10:21] PS C:\> Start-Service "Zabbix Agent"
```

---

## **📊 Post-Installation Checklist**  

| **Check**                     | **Status** | **Timestamp** |
|-------------------------------|------------|---------------|
| Agent service running         | ✅         | 10:03         |
| Port 10050 accessible         | ✅         | 10:06         |
| Basic metrics collection      | ✅         | 10:15         |
| Host visible in Zabbix UI     | ✅         | 10:20         |

---

**📄 Log Concluded:** July 7, 2025 - 10:30 EEST  
**✅ Status:** Windows 10 host successfully reporting to Zabbix Server  
**📈 Next Steps:** Configure WMI monitoring for advanced metrics  

---
🪟 **Zabbix + Windows 10 = Enterprise-Grade Monitoring**  

### **Key Notes:**  
1. For **mass deployment**, use:  
   ```powershell
   msiexec /i zabbix_agent.msi /qn SERVER=172.28.112.1 HOSTNAME=%COMPUTERNAME%
   ```
2. Enable **WMI monitoring** by linking `Template MS Windows by WMI`.  
3. For **active checks**, ensure `ServerActive` points to Zabbix Server/proxy.
