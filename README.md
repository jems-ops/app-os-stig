Good — this is exactly the kind of output you want for STIG validation 👍
Let’s break it down and tell you what’s missing + what to fix (quickly)

⸻

🔍 What your results show

✅ Apache (GOOD)

You have:

KeepAlive On
KeepAliveTimeout 30

✔ That’s valid tuning
✔ Not default → good for STIG

⸻

⚠️ Apache (MISSING IMPORTANT SETTINGS)

You did NOT see:

MaxRequestWorkers
ServerLimit
Timeout

👉 That means:
	•	Using defaults ❌
	•	This is where your finding comes from

⸻

❌ 389-DS (LDAP – NOT CONFIGURED)

Your grep returned nothing:

nsslapd-maxdescriptors
nsslapd-threadnumber
nsslapd-maxthreadsperconn

👉 This means:
	•	No tuning applied ❌
	•	This is a clear STIG gap

⸻

⚠️ OS Limits

ulimit -n = 1024 ❌ (LOW)
fs.file-max = 396063 ✅ (OK)

👉 Problem:
	•	Process limit too low → can cause DoS under load

⸻

🚨 Conclusion (Current State)

👉 This is still a FINDING

Because:
	•	Apache not fully tuned
	•	LDAP not tuned
	•	OS limits too low

⸻

🔧 Quick Fix (Do this now)

✅ 1. Fix Apache (add missing settings)

echo "MaxRequestWorkers 150" >> /etc/httpd/conf.d/tuning.conf
echo "ServerLimit 150" >> /etc/httpd/conf.d/tuning.conf
echo "Timeout 60" >> /etc/httpd/conf.d/tuning.conf


⸻

✅ 2. Fix OS limits (IMPORTANT)

echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

Then:

ulimit -n 65536


⸻

✅ 3. Fix 389-DS (LDAP tuning)

Run:

ldapmodify -x -D "cn=Directory Manager" -W <<EOF
dn: cn=config
changetype: modify
add: nsslapd-threadnumber
nsslapd-threadnumber: 16
EOF

Then:

ldapmodify -x -D "cn=Directory Manager" -W <<EOF
dn: cn=config
changetype: modify
add: nsslapd-maxdescriptors
nsslapd-maxdescriptors: 8192
EOF


⸻

🔄 Restart

systemctl restart httpd
systemctl restart dirsrv@*


⸻

🔍 Re-verify

grep -Ei "MaxRequestWorkers|ServerLimit|Timeout" /etc/httpd/conf.d/*.conf
grep -Ei "nsslapd-" /etc/dirsrv/slapd-*/dse.ldif
ulimit -n


⸻

✅ After Fix → Your Finding Details

Use this:

The web server and supporting components have been tuned to handle operational requirements. Apache is configured with connection and timeout limits, directory services (389-DS) are tuned for concurrent processing, and system file descriptor limits have been increased to support expected workload. These configurations reduce the risk of resource exhaustion and denial-of-service conditions. Therefore, this is Not a Finding.

⸻

🧾 Short Version

Apache, LDAP, and OS resource limits tuned. Not a finding.

⸻

🧠 Key Insight (important for you)

👉 STIG is NOT about “having config”
👉 It’s about:
	•	capacity handling
	•	preventing DoS
	•	proving tuning exists



