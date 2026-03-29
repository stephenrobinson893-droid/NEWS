import feedparser
import requests
import time

# ==============================
# CONFIG
# ==============================

WEBHOOK_URL = "[YOUR_NEW_WEBHOOK_HERE](https://discord.com/api/webhooks/1487947839870074960/DVq-eMNFOAwRu3ICAhDXllN5xLb5pD04U5QWJBu3QnlQLj_GHdD2mARAlJOQ-NwYkabh)"  # Put your new Discord webhook here

FEEDS = [
    "https://www.fxstreet.com/rss/news",
    "https://www.forexlive.com/feed/",
    "https://news.google.com/rss/search?q=gold+price+fed+inflation+USD",
    "https://news.google.com/rss/search?q=stock+market+economy+finance"
]

# Keywords for filtering
HIGH_IMPACT_KEYWORDS = ["fed", "fomc", "cpi", "interest rate", "treasury", "yield"]
MEDIUM_IMPACT_KEYWORDS = ["inflation", "gold", "xauusd", "usd", "dollar"]
LOW_IMPACT_KEYWORDS = ["market", "stocks", "economy", "finance"]

CHECK_INTERVAL = 300  # seconds (5 minutes)

sent_links = set()

# ==============================
# FUNCTIONS
# ==============================

def classify_impact(title):
    """Determine the impact level based on keywords"""
    title_lower = title.lower()
    if any(k in title_lower for k in HIGH_IMPACT_KEYWORDS):
        return "🔥 HIGH IMPACT"
    elif any(k in title_lower for k in MEDIUM_IMPACT_KEYWORDS):
        return "⚠️ MEDIUM IMPACT"
    elif any(k in title_lower for k in LOW_IMPACT_KEYWORDS):
        return "ℹ️ LOW IMPACT"
    else:
        return None  # ignore irrelevant news

def send_to_discord(title, link, impact):
    """Send message to Discord with impact tagging"""
    message = f"{impact}\n\n**{title}**\n{link}"
    requests.post(WEBHOOK_URL, json={"content": message})

def fetch_news():
    """Fetch news from all feeds and post relevant items"""
    for feed_url in FEEDS:
        feed = feedparser.parse(feed_url)
        for entry in feed.entries:
            title = entry.title
            link = entry.link

            if link in sent_links:
                continue

            impact = classify_impact(title)
            if impact:
                send_to_discord(title, link, impact)
                sent_links.add(link)

# ==============================
# MAIN LOOP
# ==============================

while True:
    try:
        fetch_news()
        time.sleep(CHECK_INTERVAL)
    except Exception as e:
        print("Error:", e)
        time.sleep(60)
