# nba-clutch-dashboard
Streamlit dashboard showing NBA clutch performance
🏀 NBA Clutch Performance Dashboard
This project visualizes NBA clutch performance in the final 5 minutes of close games. Built with Python, Streamlit, and the NBA API, it allows users to interactively explore key stats and shot data by player.

🚀 Live Demo: yourusername-nba-clutch-dashboard.streamlit.app
(Replace with your actual Streamlit Cloud URL)

✅ Features
⏱️ Collects clutch play-by-play data (last 5 minutes, 4th quarter)

📊 Interactive Streamlit dashboard

📦 Deployed online via GitHub + Streamlit Cloud (free)

📁 Project Structure
bash
Copy
Edit
nba_clutch_dashboard/
├── app.py                   # Streamlit dashboard code
├── clutch_play_by_play.csv # Pre-fetched clutch game data
└── README.md                # Project overview
🧱 Phase 1: Collect Clutch Data
Run this in Google Colab:



!pip install nba_api
Fetch clutch play-by-play data (last 5 minutes of the 4th quarter) using this script. The result will be saved as clutch_play_by_play.csv.

💻 Phase 2: Build the Dashboard
Create app.py with this code:
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

@st.cache_data
def load_data():
    return pd.read_csv("clutch_play_by_play.csv")

df = load_data()
st.title("🏀 NBA Clutch Performance Dashboard")
st.markdown("Analyze player performance in the final 5 minutes of close games.")

df = df[df['PLAYER1_NAME'].notna()]
player_list = sorted(df['PLAYER1_NAME'].unique())
player = st.selectbox("Select a Player", player_list)
player_df = df[df['PLAYER1_NAME'] == player]
shot_df = player_df[player_df['EVENTMSGTYPE'].isin([1, 2])]

made = (shot_df['EVENTMSGTYPE'] == 1).sum()
attempts = len(shot_df)
fg_pct = made / attempts if attempts > 0 else 0

st.subheader(f"📊 Clutch Stats for {player}")
st.metric("FG%", f"{fg_pct:.2%}")
st.metric("Shot Attempts", attempts)
st.metric("Shots Made", made)

st.markdown("### 📝 Shot Log")
st.dataframe(shot_df[['PCTIMESTRING', 'HOMEDESCRIPTION', 'VISITORDESCRIPTION']])

if attempts > 0:
    fig, ax = plt.subplots()
    ax.bar(["Shots Made", "Shots Missed"], [made, attempts - made], color=["green", "red"])
    st.pyplot(fig)
else:
    st.info("No shot data available for this player.")



🛠️ Tools Used
nba_api

Pandas

Streamlit

Matplotlib
