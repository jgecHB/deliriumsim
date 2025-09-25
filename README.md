import streamlit as st
import openai

# --- Streamlit page setup ---
st.set_page_config(page_title="DeliriumSim Chatbot", layout="centered")
st.title("🧠 DeliriumSim: Gesprek met een delirante patiënt")
st.markdown("""
Welkom bij DeliriumSim. Je voert een gesprek met **Meneer Jansen**, 78 jaar, postoperatief na een heupoperatie. 
Hij is verward, denkt dat hij thuis is, wil weg, en beschuldigt de verpleegkundige van diefstal.

Gebruik dit gesprek om je communicatieve vaardigheden te oefenen.
""")

# --- API key input ---
api_key = st.text_input("🔑 Voer je OpenAI API-sleutel in", type="password")
if api_key:
    openai.api_key = api_key

# --- Initialize session state ---
if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "system", "content": (
            "Je bent Meneer Jansen, 78 jaar, postoperatief na een heupoperatie. "
            "Je bent verward, denkt dat je thuis bent, wil weg, en beschuldigt de verpleegkundige van diefstal. "
            "Je gedrag is wisselend: soms boos, soms verdrietig, soms achterdochtig. "
            "Reageer realistisch en emotioneel zoals een delirante patiënt zou doen."
        )},
        {"role": "assistant", "content": "Waar ben ik? Dit is niet mijn huis... Waarom hebben jullie mijn horloge gepakt?"}
    ]

# --- Chat input ---
user_input = st.chat_input("Typ hier je reactie als verpleegkundige...")
if user_input and api_key:
    st.session_state.messages.append({"role": "user", "content": user_input})
    try:
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=st.session_state.messages
        )
        reply = response.choices[0].message.content
        st.session_state.messages.append({"role": "assistant", "content": reply})
    except Exception as e:
        st.error(f"Fout bij ophalen van reactie: {e}")

# --- Display chat history ---
for msg in st.session_state.messages[1:]:
    if msg["role"] == "user":
        st.chat_message("user").markdown(msg["content"])
    elif msg["role"] == "assistant":
        st.chat_message("assistant").markdown(msg["content"])

