# heart-geometry-map
心を冒険地図に変える、革新的メンタルヘルスアプリ！ 毎日スライダーで「安心」「不安」「希望」などの感情をポチポチ入力するだけ。感情の強さが「移動距離」、種類が「方向」になり、2D軌跡がクネクネ描かれて、心の旅路が一目瞭然。うつのループ、回復の前進、エネルギーの偏りを視覚化。従来の「点」スコアじゃなく、「線」の物語で心のダイナミクスを捉える！ キャラ選択（🐱🤖など）&amp;ランダム🍀イベントでモチベUP。 使い方: ブラウザアクセス、スライダー調整→記録で地図更新。プライベートJSON保存。 🔧 技術: Python (Flask, Pandas, Plotly) の軽量ベクトル合成。 📜 CC0パブリックドメイン: 誰でも自由に改良・アプリ化OK。ML診断やモバイル版を一緒に作ろうら
# エモ・ナビ 🧭 心の幾何学すごろく

## 概要
心の感情を「方向と距離」で地図化するメンタルヘルスアプリ。毎日スライダーで感情スコア入力すると、2D軌跡が描かれて心の旅路が見える。うつ傾向（ループ）や回復（前進）を視覚化。

## 解決課題
- メンタル診断の「点」ではなく「線」で動きを捉える。
- 自己認識を楽しく（キャラ選択、🍀イベント）。

## 使い方
- Flaskでローカル実行: `python app.py`
- ブラウザでhttp://localhost:9500/ アクセス。
- 感情スライダー調整→記録→地図表示。

## 技術
- Python: Flask, Pandas, Plotly
- データ: JSON保存
- ベクトル: 感情種類=方向、強さ=距離

## ライセンス
CC0 (パブリックドメイン) - 自由に使って、改良・配布OK。メンタルヘルスを世界に！

---

# Emo-Navi 🧭 Heart Geometry Sugoroku

## Overview
A mental health app that maps emotions as "direction and distance" on a 2D trail. Input daily emotion scores via sliders, and visualize your heart's journey. Detect patterns like depression loops or recovery progress.

## Problems Solved
- From "points" to "lines" for mental tracking.
- Fun self-awareness with character selection and random events (🍀).

## Usage
- Run locally with Flask: `python app.py`
- Access http://localhost:9500/ in browser.
- Adjust sliders → Record → View map.

## Tech
- Python: Flask, Pandas, Plotly
- Data: JSON storage
- Vectors: Emotion type = direction, intensity = distance

## License
CC0 (Public Domain) - Free to use, modify, distribute. Change mental health worldwide!

from flask import Flask, request, render_template_string
import pandas as pd
import plotly.graph_objects as go
import random
import os
from datetime import datetime

app = Flask(__name__)

DATA_FILE = "emotion_data.json"

# データ読み込み
if os.path.exists(DATA_FILE):
    df = pd.read_json(DATA_FILE)
else:
    df = pd.DataFrame(columns=[
        "Date", "calm", "hope", "fear", "anger", "void", "lonely", "normal", "Character"
    ])

# HTMLテンプレート
HTML = """
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>心の足あと 🐾</title>
<style>
body { font-family: Arial; background: #1a1a1a; color: #fff; text-align: center; padding: 10px; }
h1 { font-size: 24px; }
input[type=range] { width: 80%; }
.character { font-size: 24px; margin: 5px; }
button { padding: 5px 10px; margin: 5px; border-radius: 5px; }
.slider-label { display: block; margin-top: 10px; }
.feedback { margin-top: 10px; color: #fcd34d; font-weight: bold; }
</style>
</head>
<body>
<h1>心の足あと 🐾</h1>
<form method="POST">
<div>
    <h3>キャラクター</h3>
    {% for c in characters %}
        <label><input type="radio" name="character" value="{{c}}" required class="character">{{c}}</label>
    {% endfor %}
</div>
<div>
    {% for key, label in emotions.items() %}
        <label class="slider-label">{{label}} <input type="range" name="{{key}}" min="0" max="1" step="0.1" value="0.5"></label>
    {% endfor %}
</div>
<label>日付 <input type="date" name="date" value="{{date}}"></label>
<br>
<button type="submit">記録</button>
</form>
<div class="feedback">{{feedback|safe}}</div>
<div>{{plot|safe}}</div>
</body>
</html>
"""

# キャラ・感情定義
CHARACTERS = ["🐱","🤖","🐻","🐥","🐕️","🐑"]
EMOTIONS = {
    "calm":"安心", "hope":"希望", "fear":"不安", "anger":"怒り",
    "void":"虚無", "lonely":"孤独", "normal":"いつも通り"
}
DIRECTIONS = {
    "calm":(0,1), "hope":(1,1), "fear":(1,0), "anger":(0,-1),
    "void":(-1,-1), "lonely":(-1,0), "normal":(-0.5,0.5)
}
EVENTS = ["🍀 クローバーを見つけた！", "🌷 お花が咲いてた！", "🐾 足跡を発見！"]

@app.route("/", methods=["GET","POST"])
def index():
    global df
    feedback = "今日の一歩を歩こう！🐾"
    plot_div = ""
    date = datetime.today().strftime("%Y-%m-%d")

    if request.method == "POST":
        # 入力取得
        character = request.form["character"]
        step_data = {"Date": request.form.get("date", date), "Character": character}
        for key in EMOTIONS.keys():
            step_data[key] = float(request.form.get(key,0.5))

        # キャラ補正
        if character=="🐱": step_data["hope"] = min(1, step_data["hope"]+0.1)
        elif character=="🤖": step_data["fear"] = max(0, step_data["fear"]-0.1)
        elif character=="🐻": step_data["anger"] = min(1, step_data["anger"]+0.1)

        # データ追加・保存
        df = pd.concat([df, pd.DataFrame([step_data])], ignore_index=True)
        df = df.tail(7)
        df.to_json(DATA_FILE, force_ascii=False)

        # 軌跡計算
        trail = [(0,0)]
        for _, row in df.iterrows():
            x,y = trail[-1]
            for e in EMOTIONS.keys():
                dx,dy = DIRECTIONS[e]
                x += dx * row[e]
                y += dy * row[e]
            trail.append((x,y))
        # ランダムイベント
        event_msg = ""
        if random.random() < 0.2:
            event_msg = random.choice(EVENTS)
        # 連続ボーナス
        bonus_msg = ""
        if len(df)>=3: bonus_msg += "🌟 3日連続で歩けてる！"
        if len(df)>=5: bonus_msg += "🌟 5日連続で歩けてる！"

        feedback = f"{character} が歩いたよ！🐾 {event_msg} {bonus_msg}"

        # Plotly描画
        fig = go.Figure()
        fig.add_trace(go.Scatter(
            x=[p[0] for p in trail], y=[p[1] for p in trail],
            mode="lines+markers",
            line=dict(color="lightblue", width=3),
            marker=dict(size=6, color="cyan")
        ))
        fig.add_trace(go.Scatter(
            x=[trail[-1][0]], y=[trail[-1][1]],
            mode="markers+text", text=[character], textposition="top center",
            marker=dict(size=20, color="gold")
        ))
        plot_div = fig.to_html(full_html=False)

    return render_template_string(HTML, characters=CHARACTERS, emotions=EMOTIONS, plot=plot_div, feedback=feedback, date=date)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=9500, debug=True)
