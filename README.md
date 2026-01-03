import streamlit as st
import pandas as pd
import datetime

# --- CONFIG & THEME ---
st.set_page_config(page_title="Warrior Protocol", layout="wide")
st.markdown("""
    <style>
    .main { background-color: #0e1117; color: white; }
    stProgress > div > div > div > div { background-color: #00ff00; }
    </style>
    """, unsafe_allow_html=True)

st.title("🥷 IT'S NOT OVER UNTIL I WIN")

# --- DATA INITIALIZATION ---
if 'protocol_data' not in st.session_state:
    tasks = ["Early Wakeup", "Workout", "Deep Work", "Reading", "Meditation"]
    st.session_state.protocol_data = pd.DataFrame(0, index=tasks, columns=[str(i) for i in range(1, 32)])
    st.session_state.mood_data = {}
    st.session_state.screen_time = [0] * 31

# --- SECTION 1: PROTOCOL TRACKER (Left Side) ---
st.header("📋 Daily Protocol")
edited_df = st.data_editor(st.session_state.protocol_data, use_container_width=True)
st.session_state.protocol_data = edited_df

# --- SECTION 2: MOOD & SCREEN TIME (Side by Side) ---
st.divider()
col_mood, col_screen = st.columns(2)

with col_mood:
    st.header("🎭 Mood Tracker")
    day = st.selectbox("Select Day", range(1, 32))
    mood = st.select_slider("How do you feel?", options=["😫", "😔", "😐", "🙂", "🔥"])
    if st.button("Save Mood"):
        st.session_state.mood_data[day] = mood
    st.write(f"Logged Moods: {st.session_state.mood_data}")

with col_screen:
    st.header("📱 Screen Time")
    current_day = st.number_input("Day Number", min_value=1, max_value=31, step=1)
    hours = st.number_input("Hours Spent", min_value=0.0, max_value=24.0, step=0.5)
    if st.button("Update Screen Time"):
        st.session_state.screen_time[current_day-1] = hours
    st.bar_chart(st.session_state.screen_time)

# --- SECTION 3: MODERN ANALYTICS (Progress Tracker) ---
st.divider()
st.header("📈 Growth Analytics")

# Calculations
total_tasks = st.session_state.protocol_data.size
done_tasks = (st.session_state.protocol_data != 0).sum().sum()
score = (done_tasks / total_tasks) * 100

c1, c2, c3 = st.columns(3)
with c1:
    st.metric("Warrior Rank", f"Level {int(score // 10)}", delta=f"{int(score)}% Total")
with c2:
    avg_screen = sum(st.session_state.screen_time) / 31
    st.metric("Avg Screen Time", f"{round(avg_screen, 1)} hrs", delta="-0.5 hrs", delta_color="inverse")
with c3:
    st.metric("Current Streak", "5 Days", "🔥")

# Growth Curve
st.subheader("Your Progress Curve (Weekly/Monthly)")
daily_progress = st.session_state.protocol_data.sum()
st.line_chart(daily_progress)
