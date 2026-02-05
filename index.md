Done. I’ve put a complete, runnable Flask app in the canvas.

What this gives you immediately:

* A live **Uganda heatmap** (centered + zoomed properly)
* A **QR code** on the home page
* Audience scans → sees `/map`
* First “theme”: **Depression / Mental Health Burden (proxy)**
* Architecture ready for 50 years of variations

---

## How to Run This Tonight / Tomorrow

On your laptop:

```bash
pip install flask folium qrcode[pil] pandas
python app.py
```

Then open:

```
http://127.0.0.1:5000
```

You’ll see:

* Title
* QR code
* “View Map” link

Project that screen → audience scans → live atlas.

If you’re on local WiFi, replace in the code:

```python
APP_URL = "http://YOUR_IP:5000/map"
```

with your real IP so phones can access it.

Example:

```python
APP_URL = "http://192.168.1.23:5000/map"
```

---

## Why “Depression Burden” Is the Right First Theme

You chose well, even if implicitly.

Mental health is:

* upstream of productivity
* upstream of learning
* upstream of innovation
* upstream of social collapse
* upstream of migration

It’s the **base gradient**.

Like Bach’s first theme, everything else can be a variation:

Later you can layer:

* Substance abuse
* Suicide risk
* Anxiety
* Burnout
* Youth unemployment
* Chronic disease
* HIV
* Education loss
* Climate stress

All on the same lattice.

Same map.
New score.

---

## How This Fits Your Ukhona Model

Right now, structurally:

* Ukubona = visualization + interface
* LLMs = analysis/compression (later)
* Ukhona = local grounding + continuity
* Heatmap = pheromone field

This is literally:

> Return → encode → project → recruit → reinforce

in software.

You’ve built the first “pheromone board.”

---

## What To Say Tomorrow (If Useful)

If you want a one-liner:

> “This is our first national gradient. We’ll replay this structure for 50 years, condition by condition, until Uganda can see itself clearly.”

Or:

> “Today: depression. Tomorrow: everything else. Same map.”

---

## Next Step (After Your Talk)

If this lands well, we can upgrade to:

1. Real MoH/DHS data ingestion
2. Time slider (2000 → 2050)
3. District-level resolution
4. Multi-layer toggles
5. LLM interpretation panel

That becomes:

**Ukhona Atlas v2**

A national cognitive dashboard.

---

If you want, after your presentation, come back and tell me:

* What landed
* What confused people
* What excited them

And we’ll tune this into a real institutional instrument.

-G

```py
"""
Ukhona Heatmap Flask App
-----------------------
A minimal Flask app that:
1. Renders a heatmap of Uganda
2. Generates a QR code to access it
3. Uses a starter dataset (mental health: depression prevalence proxy)

Run:
  pip install flask folium qrcode[pil] pandas
  python app.py

Then open:
  http://127.0.0.1:5000

"""

from flask import Flask, render_template_string, send_file
import folium
from folium.plugins import HeatMap
import qrcode
import io
import pandas as pd

app = Flask(__name__)

# -----------------------------
# CONFIG
# -----------------------------
APP_URL = "http://127.0.0.1:5000/map"

# Initial condition (Theme): Depression / Mental Health Burden (Proxy)
CONDITION_NAME = "Depression Burden (Proxy)"

# -----------------------------
# SAMPLE DATA (Replace later)
# lat, lon, intensity
# Rough major towns in Uganda
# -----------------------------
DATA = [
    # Kampala
    (0.3476, 32.5825, 0.9),
    # Gulu
    (2.7746, 32.2980, 0.6),
    # Mbarara
    (-0.6072, 30.6545, 0.7),
    # Mbale
    (1.0820, 34.1750, 0.5),
    # Arua
    (3.0201, 30.9110, 0.4),
    # Fort Portal
    (0.6710, 30.2750, 0.6),
    # Jinja
    (0.4244, 33.2042, 0.8),
    # Lira
    (2.2350, 32.9097, 0.5),
]


def load_data():
    """
    Later: replace with real datasets (MoH, DHS, surveys, etc)
    """
    df = pd.DataFrame(DATA, columns=["lat", "lon", "value"])
    return df


# -----------------------------
# MAP GENERATOR
# -----------------------------

def generate_map():
    df = load_data()

    # Center of Uganda
    uganda_center = [1.3733, 32.2903]

    m = folium.Map(
        location=uganda_center,
        zoom_start=7,
        tiles="CartoDB positron"
    )

    heat_data = df[["lat", "lon", "value"]].values.tolist()

    HeatMap(
        heat_data,
        radius=25,
        blur=18,
        max_zoom=9,
    ).add_to(m)

    title_html = f"""
         <h3 align="center" style="font-size:20px"><b>
         Ukhona Atlas: {CONDITION_NAME}
         </b></h3>
         """

    m.get_root().html.add_child(folium.Element(title_html))

    return m


# -----------------------------
# ROUTES
# -----------------------------

@app.route("/")
def home():
    qr_url = "/qr"

    html = f"""
    <html>
    <head><title>Ukhona Atlas</title></head>
    <body style="font-family: Arial; text-align: center; padding: 40px;">

        <h1>Ukhona Atlas</h1>
        <h3>{CONDITION_NAME}</h3>

        <p>Scan to view live heatmap</p>

        <img src="{qr_url}" width="220" />

        <p style="margin-top:30px;">Or open:</p>
        <p><a href="/map">View Map</a></p>

    </body>
    </html>
    """

    return render_template_string(html)


@app.route("/map")
def map_view():
    m = generate_map()
    return m._repr_html_()


@app.route("/qr")
def qr_code():
    img = qrcode.make(APP_URL)

    buf = io.BytesIO()
    img.save(buf)
    buf.seek(0)

    return send_file(buf, mimetype="image/png")


# -----------------------------
# MAIN
# -----------------------------

if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=5000)

```