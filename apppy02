import streamlit as st
import pandas as pd
import google.generativeai as genai

st.set_page_config(page_title="Chat with Your CSV using Gemini", layout="wide")
st.title("💬 Chat with CSV using Gemini Pro")
st.caption("Powered by Streamlit + Google Generative AI")

# -------- API Key -------- #
api_key = st.text_input("🔑 Enter your Gemini API Key", type="password")
model = None
if api_key:
    try:
        genai.configure(api_key=api_key)
        model = genai.GenerativeModel("gemini-pro")
        st.success("✅ Gemini model is ready!")
    except Exception as e:
        st.error(f"Failed to configure Gemini: {e}")

# -------- Session State -------- #
if "chat_history" not in st.session_state:
    st.session_state.chat_history = []
if "csv_data" not in st.session_state:
    st.session_state.csv_data = None
if "data_dict" not in st.session_state:
    st.session_state.data_dict = None

# -------- Upload Section -------- #
st.subheader("📁 Upload CSV Files")
col1, col2 = st.columns(2)

with col1:
    data_file = st.file_uploader("Upload your main dataset (CSV)", type="csv")
    if data_file:
        try:
            st.session_state.csv_data = pd.read_csv(data_file)
            st.success("✅ Dataset loaded")
            st.dataframe(st.session_state.csv_data.head())
        except Exception as e:
            st.error(f"Failed to read CSV: {e}")

with col2:
    dict_file = st.file_uploader("Upload Data Dictionary (optional)", type="csv")
    if dict_file:
        try:
            st.session_state.data_dict = pd.read_csv(dict_file)
            st.success("✅ Data Dictionary loaded")
            st.dataframe(st.session_state.data_dict.head())
        except Exception as e:
            st.error(f"Failed to read data dictionary: {e}")

# -------- Chat History -------- #
for role, msg in st.session_state.chat_history:
    st.chat_message(role).markdown(msg)

# -------- Chat Input -------- #
if user_input := st.chat_input("Type your question about the data..."):

    st.chat_message("user").markdown(user_input)
    st.session_state.chat_history.append(("user", user_input))

    if not model:
        st.warning("Please enter your Gemini API Key.")
    elif st.session_state.csv_data is None:
        st.warning("Please upload a dataset first.")
    else:
        try:
            # สร้าง prompt
            prompt = user_input
            if any(word in user_input.lower() for word in ["analyze", "insight", "summary", "describe"]):
                summary = st.session_state.csv_data.describe(include="all").to_string()
                prompt = f"""Here is a summary of the dataset:
{summary}

Data dictionary (if any):
{st.session_state.data_dict.to_string() if st.session_state.data_dict is not None else 'None'}

Now: {user_input}
"""

            response = model.generate_content(prompt)
            bot_response = response.text
            st.chat_message("assistant").markdown(bot_response)
            st.session_state.chat_history.append(("assistant", bot_response))

        except Exception as e:
            st.error(f"❌ Error generating response: {e}")
