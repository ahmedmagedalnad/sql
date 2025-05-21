# sql
This Bash script automates security scans on a target server using nmap (ports), nikto (web flaws), OWASP ZAP (web app security), and ping (reachability). It auto-installs missing tools via apt and saves results in folders. Designed for Kali Linux and adaptable to Termux for fast, consistent scans

nano run_all_scans.sh

  #!/bin/bash

# الكلمة المطلوبة في السكريبت
echo "CyberCode"

# IP السيرفر الهدف
TARGET_IP="الحصول على عنوان ip "

# مجلد حفظ النتائج
OUTPUT_DIR="./scan_results"
mkdir -p "$OUTPUT_DIR"

# دالة لفحص وجود برنامج، وتثبيته لو مش موجود
install_if_not_found() {
    CMD=$1
    PKG=$2
    if ! command -v "$CMD" &> /dev/null
    then
        echo "$CMD not found. Installing $PKG..."
        sudo apt update
        sudo apt install -y "$PKG"
    else
        echo "$CMD is already installed."
    fi
}

# تثبيت الأدوات لو مش موجودة
install_if_not_found nmap nmap
install_if_not_found nikto nikto
install_if_not_found zap-cli zaproxy   #zap-cli غالباً تَتطلب تثبيت zaproxy وجافا

# ملاحظة: zaproxy يحتاج java، نثبتها لو مش موجودة
if ! command -v java &> /dev/null
then
    echo "Java not found. Installing default-jre..."
    sudo apt install -y default-jre
fi

# تشغيل nmap
echo "Running nmap scan on $TARGET_IP..."
nmap -sV -oN "$OUTPUT_DIR/nmap_$TARGET_IP.txt" "$TARGET_IP" &
NMAP_PID=$!

# تشغيل nikto
echo "Running nikto scan on $TARGET_IP..."
nikto -host "$TARGET_IP" -output "$OUTPUT_DIR/nikto_$TARGET_IP.txt" &
NIKTO_PID=$!

# تشغيل zap-cli (نفترض انه تم تثبيت zaproxy مسبقاً)
echo "Running ZAP Proxy scan on http://$TARGET_IP ..."
zap-cli quick-scan --self-contained --spider --scanners all "http://$TARGET_IP" > "$OUTPUT_DIR/zaproxy_$TARGET_IP.txt" 2>&1 &
ZAP_PID=$!

# تشغيل ping scan بسيط
echo "Running ping scan on $TARGET_IP..."
ping -c 4 "$TARGET_IP" > "$OUTPUT_DIR/ping_$TARGET_IP.txt" &
PING_PID=$!

# انتظر انتهاء كل العمليات
wait $NMAP_PID
wait $NIKTO_PID
wait $ZAP_PID
wait $PING_PID

echo "All scans completed. Results saved in $OUTPUT_DIR."


chmod +x run_all_scans.sh

./run_all_scans.sh
