## 🌦️ Weather Forecast Workflow (n8n)

An automated daily weather check workflow built in n8n.
Every morning at 07:30, the workflow fetches rain data from Open-Meteo, analyzes the forecast, and sends a personalized weather message to Telegram.

<p align="center"> <img src="https://img.shields.io/badge/n8n-Workflow-orange?style=for-the-badge" /> <img src="https://img.shields.io/badge/Weather-API-blue?style=for-the-badge" /> <img src="https://img.shields.io/badge/Telegram-Alerts-26A5E4?style=for-the-badge" /> </p>

---

##🖼️ Workflow Preview

(You can replace these placeholders with real screenshots)

<p align="center"> <img src="https://via.placeholder.com/900x350?text=n8n+Weather+Workflow+Overview" /> </p>

---

## 🚀 Features

⏰ Scheduled every morning at 07:30

☁️ Pulls daily weather data using Open-Meteo API

🧠 Detects:

strong rain

medium rain

no rain

💬 Generates a personalized weather message

📲 Sends the message directly to your Telegram

🔄 Randomized messages so it feels more natural

🛡️ Credentials not included in the workflow export (safe for GitHub)

---

## 📁 Workflow Structure
⏰ 1. Schedule Trigger

Runs automatically at:

07:30 (Berlin time)

Starts the entire workflow every morning.

---

## 🌍 2. HTTP Request Node

Requests weather data for a specific location:

https://api.open-meteo.com/v1/forecast
    ?latitude=48.7665
    &longitude=11.4258
    &daily=precipitation_sum,precipitation_probability_max
    &timezone=Europe/Berlin


Returns fields like:

daily.time

daily.precipitation_sum

daily.precipitation_probability_max

---

## 🧩 3. Edit Fields Node (Set)

Extracts only the daily object from the API response:

{
  "daily": { ... }
}


Allows easier processing in the code node.

---

## 🧠 4. Code Node (JavaScript Logic)

Analyzes today’s weather and picks a random message:

````js

const daily = $json["daily"];
const todayStr = new Date().toISOString().split('T')[0];

const idx = daily.time.indexOf(todayStr);
if (idx === -1) return [];

const rain_mm = daily.precipitation_sum[idx];
const rain_prob = daily.precipitation_probability_max[idx];

let message = "";

if (rain_prob >= 80) {
    const options = [
        `Achtung! Starkregen heute (${rain_prob}%, ${rain_mm} mm)!`,
        `Wow, today will be very rainy (${rain_prob}%, ${rain_mm} mm)!`,
        `Heavy rain alert: ${rain_prob}% chance, ${rain_mm} mm.`,
    ];
    message = options[Math.floor(Math.random() * options.length)];
} else if (rain_prob >= 50) {
    const options = [
        `It might rain today (${rain_prob}%, ${rain_mm} mm).`,
        `Rain probability today: ${rain_prob}% (${rain_mm} mm).`,
        `Small chance of rain today (${rain_prob}%, ${rain_mm} mm).`,
    ];
    message = options[Math.floor(Math.random() * options.length)];
} else {
    return [];
}

return [{ json: { message } }];
