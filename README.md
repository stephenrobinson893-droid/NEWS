import requests
import json
import os

# =========================
# CONFIG
# =========================
WEBHOOK_URL = os.getenv("https://discord.com/api/webhooks/1489008377722830968/XnfmkcvFbny_GNs0IaRJYIRLCXPz2gd2bwISuZM1VPtCEsa_7kUEB1zXpj4myaS7Tbn1")

URL = "https://nfs.faireconomy.media/ff_calendar_thisweek.json"

CURRENCY = "USD"
IMPACT = "High"

STATE_FILE = "sent_events.json"

# =========================
# LOAD STATE
# =========================
def load_sent_events():
    if os.path.exists(STATE_FILE):
        with open(STATE_FILE, "r") as f:
            return set(json.load(f))
    return set()

def save_sent_events(events):
    with open(STATE_FILE, "w") as f:
        json.dump(list(events), f)

sent_events = load_sent_events()

# =========================
# DISCORD
# =========================
def send_to_discord(message):
    data = {"content": message}
    requests.post(WEBHOOK_URL, json=data)

# =========================
# FETCH DATA
# =========================
def fetch_calendar():
    try:
        response = requests.get(URL)
        return response.json()
    except:
        return []

# =========================
# PROCESS EVENTS
# =========================
def process_events(events):
    global sent_events

    for event in events:
        try:
            currency = event.get("currency")
            impact = event.get("impact")
            title = event.get("title")
            actual = event.get("actual")
            forecast = event.get("forecast")
            previous = event.get("previous")
            time_str = event.get("time")

            # Filters
            if currency != CURRENCY or impact != IMPACT:
                continue

            # Only when actual data is released
            if actual in (None, "", " "):
                continue

            event_id = f"{title}-{time_str}"

            if event_id in sent_events:
                continue

            message = (
                f"🚨 **HIGH IMPACT ({currency})**\n"
                f"**{title}**\n"
                f"Actual: {actual}\n"
                f"Forecast: {forecast}\n"
                f"Previous: {previous}\n"
                f"Time: {time_str}"
            )

            send_to_discord(message)

            sent_events.add(event_id)

        except:
            continue

    save_sent_events(sent_events)

# =========================
# RUN ONCE (GitHub handles schedule)
# =========================
if __name__ == "__main__":
    events = fetch_calendar()
    process_events(events)
