# 🏀 NBA Clutch Performance Dashboard

An interactive dashboard that visualizes NBA player performance in the **final 5 minutes of close games**. Built with **Python**, **Streamlit**, and the **NBA API**.

🔗 **Live Demo**: [(https://nba-clutch-dashboard-cswnnumnydavxpggtt3vtw.streamlit.app/) 
*(Replace with your actual Streamlit Cloud URL)*

---

## ✅ Features

- Collects clutch play-by-play data (4th quarter, last 5 minutes)
- Streamlit dashboard for player-specific analysis
- Visualizes FG%, attempts, and shot logs
- Easy deployment with GitHub + Streamlit Cloud

---

## 📁 Files

nba-clutch-dashboard/
├── app.py # Streamlit app
├── clutch_play_by_play.csv # Clutch game data
└── README.md # Project overview

## 🧱 Step-by-Step Instructions

### Collect Clutch Data (Google Colab)

!pip install nba_api

## Then run this code to generate clutch_play_by_play.csv: 

from nba_api.stats.endpoints import leaguegamefinder, playbyplayv2
import pandas as pd
import time

gamefinder = leaguegamefinder.LeagueGameFinder(season_nullable='2023-24', league_id_nullable='00')
games_df = gamefinder.get_data_frames()[0]
game_ids = games_df['GAME_ID'].drop_duplicates().head(20).tolist()

def convert_time(t):
    try:
        m, s = map(int, t.split(":"))
        return m * 60 + s
    except:
        return 9999

clutch_plays = []

for game_id in game_ids:
    try:
        pbp = playbyplayv2.PlayByPlayV2(game_id=game_id)
        df = pbp.get_data_frames()[0]
        df['SECONDS_REMAINING'] = df['PCTIMESTRING'].apply(convert_time)
        clutch_df = df[(df['PERIOD'] == 4) & (df['SECONDS_REMAINING'] <= 300)].copy()
        clutch_df['GAME_ID'] = game_id
        clutch_plays.append(clutch_df)
        time.sleep(1)
    except Exception as e:
        print(f"Failed game {game_id}: {e}")

clutch_data = pd.concat(clutch_plays, ignore_index=True)
clutch_data.to_csv("clutch_play_by_play.csv", index=False) 

## Create Dashboard: 
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
