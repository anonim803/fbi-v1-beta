from flask import Flask, request, redirect, session, render_template_string
from werkzeug.security import generate_password_hash, check_password_hash
import sqlite3
import os

app = Flask(__name__)
app.secret_key = os.environ.get("FBI_SECRET_KEY", "demo-change-before-production")
DB = os.path.join(os.path.dirname(__file__), "fbi.db")

USERS = [
    ("PlayerDemo", "demo123", "player", "Игрок"),
    ("AgentDemo", "demo123", "agent", "Сотрудник FBI"),
    ("DeputyDemo", "demo123", "deputy", "Заместитель руководителя FBI"),
    ("ChiefDemo", "demo123", "chief", "Руководитель FBI"),
]

def connect():
    con = sqlite3.connect(DB)
    con.row_factory = sqlite3.Row
    return con

def init_db():
    with connect() as con:
        con.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY,
                nickname TEXT UNIQUE,
                password TEXT,
                role TEXT,
                rank_name TEXT
            )
        """)
        for nick, password, role, rank in USERS:
            con.execute(
                "INSERT OR IGNORE INTO users VALUES (NULL,?,?,?,?)",
                (nick, generate_password_hash(password), role, rank)
            )

PAGE = """
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>FBI v1.0 BETA</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#090a0d;color:#eee;
font-family:Arial,sans-serif}
header{padding:22px;background:#111319;
border-bottom:2px solid #a52b35;letter-spacing:2px}
header b{font-size:24px;color:#e0444e}
main{max-width:850px;margin:45px auto;padding:20px}
.card{background:#13161d;border:1px solid #292e39;
border-radius:12px;padding:26px;margin:18px 0}
h1{font-size:30px}
p{color:#a7adba;line-height:1.6}
label{display:block;margin:17px 0 8px}
input{width:100%;padding:14px;background:#090b10;
border:1px solid #343b49;border-radius:6px;color:white}
button,.btn{display:block;width:100%;margin-top:20px;
padding:14px;border:0;border-radius:6px;
background:#b92e39;color:white;font-weight:bold;
text-align:center;text-decoration:none;font-size:15px}
.tile{padding:17px;margin:12px 0;background:#0b0d12;
border:1px solid #292e39;border-radius:7px}
small{color:#dc7279}
.error{color:#ff858b}
footer{text-align:center;color:#626a78;padding:25px;font-size:12px}
</style>
</head>
<body>
<header><b>FBI</b>　FEDERAL BUREAU
<div><small>v1.0 [БЕТА-ТЕСТ]</small></div></header>
<main>
{% if not user %}
<div class="card">
<small>SECURE ACCESS PORTAL</small>
<h1>Система доступа FBI</h1>
<p>Вход в демонстрационный игровой портал.</p>
{% if error %}<p class="error">{{error}}</p>{% endif %}
<form method="post">
<label>Ваш никнейм</label>
<input name="nickname" required autocomplete="username">
<label>Тестовый пароль</label>
<input name="password" type="password" required
autocomplete="current-password">
<button>ВОЙТИ В СИСТЕМУ →</button>
</form>
</div>
<div class="card">
<h3>Уровни доступа</h3>
<div class="tile">01 — Стандартная панель игрока</div>
<div class="tile">02 — Организация FBI</div>
<div class="tile">03 — Управление организацией</div>
<p>Это тестовый сайт. Он не подключён к серверам Arizona RP.</p>
</div>
{% else %}
<div class="card">
<small>АВТОРИЗАЦИЯ УСПЕШНА</small>
<h1>{{heading}}</h1>
<p>Никнейм: <b>{{user.nickname}}</b></p>
<p>Роль: {{user.rank_name}}</p>
{% for item in items %}
<div class="tile"><h3>{{item}}</h3>
<p>Демонстрационный раздел портала.</p></div>
{% endfor %}
<a class="btn" href="/logout">ВЫЙТИ ИЗ АККАУНТА</a>
</div>
{% endif %}
</main>
<footer>FBI PORTAL · VERSION 1.0 · BETA TEST</footer>
</body>
</html>
"""

@app.route("/", methods=["GET", "POST"])
def index():
    error = None
    if request.method == "POST":
        nick = request.form.get("nickname", "").strip()
        password = request.form.get("password", "")
        with connect() as con:
            user = con.execute(
                "SELECT * FROM users WHERE nickname=?", (nick,)
            ).fetchone()
        if user and check_password_hash(user["password"], password):
            session.clear()
            session["uid"] = user["id"]
            return redirect("/")
        error = "Неверный тестовый никнейм или пароль."

    user = None
    heading = ""
    items = []

    if session.get("uid"):
        with connect() as con:
            user = con.execute(
                "SELECT * FROM users WHERE id=?", (session["uid"],)
            ).fetchone()

        if user:
            if user["role"] == "player":
                heading = "Стандартная панель"
                items = ["Мой профиль", "Новости", "Поддержка"]
            elif user["role"] == "agent":
                heading = "Организация FBI"
                items = ["Личный состав", "Служебная информация",
                         "Оперативные задачи"]
            else:
                heading = "Управление FBI"
                items = ["Личный состав", "Заявки и назначения",
                         "Настройки организации"]

    return render_template_string(
        PAGE, user=user, error=error, heading=heading, items=items
    )

@app.route("/logout")
def logout():
    session.clear()
    return redirect("/")

init_db()

if __name__ == "__main__":
    app.run(host="0.0.0.0",
            port=int(os.environ.get("PORT", 5000)),
            debug=False)