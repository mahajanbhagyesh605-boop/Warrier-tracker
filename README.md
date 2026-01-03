# Warrier-tracker
import streamlit as st
import pandas as pd
import datetime

# --- APP CONFIG ---
st.set_page_config(page_title="Warrior Protocol", layout="wide")
st.title("🥷 IT'S NOT OVER UNTIL I WIN")

# --- DATABASE SIMULATION ---
# In a real app, this would save to a database. 
# For now, it uses the session state to keep your data while the app is open.
if 'data' not in st.session_state:
    protocols = ["Early Wakeup", "Workout", "Deep Work", "Reading", "No Junk Food"]
    df = pd.DataFrame(0, index=protocols, columns=[str(i) for i in range(1, 32)])
    st.session_state.data = df

# --- SIDEBAR: ADD NEW PROTOCOLS ---
st.sidebar.header("Settings")
new_task = st.sidebar.text_input("Add New Protocol:")
if st.sidebar.button("Add Task") and new_task:
    new_row = pd.DataFrame(0, index=[new_task], columns=st.session_state.data.columns)
    st.session_state.data = pd.concat([st.session_state.data, new_row])

# --- MAIN TRACKER ---
st.subheader("Monthly Protocol Sheet")
edited_df = st.data_editor(st.session_state.data, num_rows="dynamic")
st.session_state.data = edited_df

# --- MODERN PROGRESS ANALYTICS ---
st.divider()
st.header("📈 My Progress (Modern Analytics)")

# Calculation Logic
total_cells = st.session_state.data.size
completed_cells = (st.session_state.data != 0).sum().sum()
completion_rate = (completed_cells / total_cells) * 100

col1, col2, col3 = st.columns(3)

with col1:
    st.metric(label="Warrior Level", value=f"LVL {int(completion_rate // 10)}", delta=f"{int(completion_rate)}% EXP")
    st.progress(completion_rate / 100)

with col2:
    # Monthly Improvement (Simulated comparison)
    st.metric(label="Monthly Improvement", value="Ready", delta="+12% vs last month")

with col3:
    st.metric(label="Active Streak", value="5 Days", delta="🔥")

# Progress Chart (Visualizing the "My Progress" dots from your image)
st.subheader("Growth Curve")
daily_totals = st.session_state.data.sum().values
st.line_chart(daily_totals)

st.success("Keep going! Every tick is a step toward the win.")
