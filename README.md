# CAPSTONE-PROJECT---GOOGLE
AI Social Media Content Creator Agent
# ============================================
# 🔥 AI SOCIAL MEDIA CONTENT CREATOR AGENT
# ============================================

!pip install transformers gradio googletrans==4.0.0-rc1 torch --quiet

from transformers import pipeline
from googletrans import Translator
import gradio as gr
import random
import datetime
import os

# ------------------------------
# 🧠 LOAD LANGUAGE MODEL
# ------------------------------
generator = pipeline("text-generation", model="gpt2")
translator = Translator()

# ------------------------------
# 🤖 FUNCTIONS
# ------------------------------

# 1. Generate Caption
def generate_caption(topic, lang):
    base = generator(f"Instagram caption about {topic}", max_length=60, num_return_sequences=1)[0]["generated_text"]

    if lang == "Tamil":
        base = translator.translate(base, dest="ta").text
    
    return base


# 2. Generate Hashtags
def generate_hashtags(topic):
    words = topic.lower().split()
    tags = [f"#{w}" for w in words]
    common_tags = ["#trending", "#viral", "#motivation", "#dailyquotes", "#reelsidea", "#creators"]
    return " ".join(tags + random.sample(common_tags, 3))


# 3. Reel Ideas
def reel_ideas(topic, lang):
    ideas = [
        f"{topic} – Step-by-step guide",
        f"{topic} – Before vs After",
        f"{topic} – Quick tips in 30 seconds",
        f"{topic} – Top 3 secrets nobody tells you",
    ]

    text = "\n".join(ideas)
    if lang == "Tamil":
        text = translator.translate(text, dest="ta").text
    
    return text


# 4. Motivational / Tech Quotes
def quote_generator(style, lang):
    quotes = {
        "Motivation": [
            "Success begins with small consistent steps.",
            "You are stronger than your excuses.",
            "Every day is a new chance to grow.",
        ],
        "Tech": [
            "AI is not the future. It is the present.",
            "Code is like humor – when you have to explain it, it’s bad.",
            "Technology is best when it brings people together.",
        ]
    }

    quote = random.choice(quotes[style])

    if lang == "Tamil":
        quote = translator.translate(quote, dest="ta").text
    
    return quote


# 5. Export report (optional for project submission)
def export_report(topic, caption, hashtags, reel, quote):
    os.makedirs("reports", exist_ok=True)
    filename = f"reports/report_{datetime.datetime.now().strftime('%Y%m%d_%H%M%S')}.txt"

    with open(filename, "w", encoding="utf-8") as f:
        f.write("AI SOCIAL MEDIA CONTENT REPORT\n")
        f.write("=====================================\n\n")
        f.write(f"Topic: {topic}\n\n")
        f.write(f"Caption:\n{caption}\n\n")
        f.write(f"Hashtags:\n{hashtags}\n\n")
        f.write(f"Reel Ideas:\n{reel}\n\n")
        f.write(f"Quote:\n{quote}\n\n")
    
    return filename


# ------------------------------
# 🎨 BUILD GRADIO UI
# ------------------------------
def full_agent(topic, lang, quote_style):

    caption = generate_caption(topic, lang)
    hashtags = generate_hashtags(topic)
    reels = reel_ideas(topic, lang)
    quote = quote_generator(quote_style, lang)
    
    file = export_report(topic, caption, hashtags, reels, quote)

    return caption, hashtags, reels, quote, file


# UI Layout
app = gr.Interface(
    fn = full_agent,
    inputs = [
        gr.Textbox(label="Enter Topic (e.g., AI, Motivation, Nature)"),
        gr.Dropdown(["English", "Tamil"], label="Select Language"),
        gr.Dropdown(["Motivation", "Tech"], label="Quote Type")
    ],
    outputs = [
        gr.Textbox(label="Generated Caption"),
        gr.Textbox(label="Hashtags"),
        gr.Textbox(label="Reel Ideas"),
        gr.Textbox(label="Quote"),
        gr.File(label="Download Report")
    ],
    title="AI Social Media Content Creator Agent",
    description="Generate captions, hashtags, reels, motivational quotes in Tamil/English."
)

app.launch()
