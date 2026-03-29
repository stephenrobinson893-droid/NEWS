import feedparser
import requests
import time

# ==============================
# CONFIG
# ==============================

WEBHOOK_URL = "PASTE_YOUR_NEW_WEBHOOK_HERE"  # Use a new webhook

FEEDS = [
    "https://www.fxstreet.com/rss/news",
    "https://www.forexlive.com/feed/",
    "https://news.google.com/rss/search?q=gold+price+fed+inflation+USD"
]

KEYWORDS = [
    "fed", "inflation", "interest rate", "gold", "xauusd",
    "usd", "dollar", "treasury", "yield", "cpi", "fomc"
]

CHECK_INTERVAL = 300  # seconds (5 minutes)

# ==============================
# MEMORY (avoid duplicates)
# ==============================

sent_links = set()

# ==============================
# FUNCTIONS
# ==============================

def is_relevant(title):
    title_lower = title.lower()
    return any(keyword in title_lower for keyword in KEYWORDS)


def send_to_discord(title, link):
    message = f"🚨 **MARKET NEWS ALERT**\n\n**{title}**\n{link}"
    requests.post(WEBHOOK_URL, json={"content": message})


def fetch_news():
    for feed_url in FEEDS:
        feed = feedparser.parse(feed_url)

        for entry in feed.entries:
            title = entry.title
            link = entry.link

            if link in sent_links:
                continue

            if is_relevant(title):
                send_to_discord(title, link)
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
