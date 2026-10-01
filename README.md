# أوامر المشرفين
from database import get_user

def is_admin(db, sender):
    return get_user(db,sender).get("is_admin") or sender==__import__("config").OWNER_NUMBER

def handle_admin(db, sender, text, mentions):
    s=db["settings"]
    msg=None
    t=text.strip()
    if t.startswith("طرد") and mentions:
        msg=f"🚫 تم طرد @{mentions[0]} من الجروب (تنفيذ وهمي - اربط بـ API الطرد)"
    elif t in ["قفل","فتح"]:
        s["group_locked"]=(t=="قفل")
        msg="🔒 تم قفل الشات للمشرفين فقط" if t=="قفل" else "🔓 تم فتح الشات للكل"
    elif t.startswith("ترقية") and mentions:
        get_user(db,mentions[0])["is_admin"]=True; msg=f"⬆️ تم ترقية @{mentions[0]} لمشرف"
    elif t.startswith("تنزيل") and mentions:
        get_user(db,mentions[0])["is_admin"]=False; msg=f"⬇️ تم تنزيل @{mentions[0]} من الإشراف"
    elif t.startswith("فلوس"):
        parts=t.split()
        if len(parts)>=3 and mentions:
            amount=int(parts[1]); get_user(db,mentions[0])["balance"]+=amount
            msg=f"💰 تم إعطاء {amount} MToken لـ @{mentions[0]} من البنك"
    elif t.startswith("خصم"):
        parts=t.split()
        if len(parts)>=3 and mentions:
            amount=int(parts[1]); get_user(db,mentions[0])["balance"]-=amount
            msg=f"➖ تم خصم {amount} MToken من @{mentions[0]}"
    elif t.startswith("حظر @") or (t.startswith("حظر") and mentions):
        get_user(db,mentions[0])["banned_game"]=True; msg=f"⛔ تم حظر @{mentions[0]} من الألعاب"
    elif t=="حظر":
        s["games_locked"]=True; msg="⛔ تم حظر الألعاب للكل (مشرفين فقط)"
    elif t.startswith("فك @") or (t.startswith("فك") and mentions):
        get_user(db,mentions[0])["banned_game"]=False; msg=f"✅ تم فك حظر @{mentions[0]}"
    elif t=="فك":
        s["games_locked"]=False; msg="✅ تم فتح الألعاب للكل"
    elif t=="ايقاف":
        s["bot_enabled"]=False; msg="⏸️ تم إيقاف البوت"
    elif t=="تشغيل":
        s["bot_enabled"]=True; msg="▶️ تم تشغيل البوت"
    elif t.startswith("تسجيل") and mentions:
        gender="ولد" if "ولد" in t else "بنت" if "بنت" in t else None
        get_user(db,mentions[0])["gender"]=gender; msg=f"📝 تم تسجيل @{mentions[0]} كـ {gender}"
    return msg
