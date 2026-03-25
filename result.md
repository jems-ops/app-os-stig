
✅ Finding

The web server and supporting components were not fully tuned to handle expected operational workload.

Apache configuration lacked explicit worker and connection limits (e.g., MaxRequestWorkers, ServerLimit, Timeout).

FreeIPA Directory Server (389-DS) tuning parameters were not configured under cn=config, and no corresponding entries were present in the directory configuration file (/etc/dirsrv/slapd-VA2-NEPTUNE-NET/dse.ldif).

OS-level file descriptor limits (ulimit -n) were set too low (1024), increasing the risk of resource exhaustion and potential denial-of-service (DoS) conditions under load.

⸻

🔧 Fix

The system was tuned across all layers to support expected workload and prevent DoS conditions:
	•	Apache configured in:
	•	/etc/httpd/conf.d/tuning.conf
With parameters:
	•	MaxRequestWorkers 150
	•	ServerLimit 150
	•	Timeout 60
	•	KeepAlive On
	•	KeepAliveTimeout 30
	•	FreeIPA Directory Server (389-DS) updated using ldapmodify under:
	•	DN: cn=config
	•	Backend file: /etc/dirsrv/slapd-VA2-NEPTUNE-NET/dse.ldif
Applied settings:
	•	nsslapd-threadnumber: 16
	•	nsslapd-maxdescriptors: 8192
	•	OS file descriptor limits updated in:
	•	/etc/security/limits.conf
With:
	•	* soft nofile 65536
	•	* hard nofile 65536
	•	All changes validated using:
	•	grep (Apache + LDAP file)
	•	ldapsearch (cn=config)
	•	ulimit and sysctl


💬 Comment

Apache, FreeIPA (389-DS), and OS resource limits have been tuned to handle expected operational workload.

LDAP configuration was safely updated using ldapmodify under cn=config, which is persisted in /etc/dirsrv/slapd-VA2-NEPTUNE-NET/dse.ldif.

System limits were increased to support concurrent connections and prevent resource exhaustion.

These configurations ensure the system can handle expected traffic without degradation or denial-of-service conditions.

Status: Not a Finding
