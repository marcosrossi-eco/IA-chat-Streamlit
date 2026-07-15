# IA-chat-Streamlit
import os
import streamlit as st
from dotenv import load_dotenv
from openai import OpenAI

# Carrega .env
load_dotenv()

api_key = os.getenv("OPENROUTER_API_KEY")

# Configuração da página
st.set_page_config(
    page_title="Chatbot Especialista",
    page_icon="🤖",
    layout="centered"
)

st.title("🤖 Chatbot com IA Generativa")
st.write("Digite uma pergunta e receba uma resposta.")

# Verifica chave
if not api_key:
    st.error("OPENROUTER_API_KEY não encontrada no .env")
    st.stop()

# Cliente
client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=api_key
)

# Histórico
if "mensagens" not in st.session_state:
    st.session_state.mensagens = [
        {
            "role": "system",
            "content": "Você é um chatbot educacional, claro e didático."
        }
    ]

# Exibe histórico
for mensagem in st.session_state.mensagens:
    if mensagem["role"] != "system":
        with st.chat_message(mensagem["role"]):
            st.write(mensagem["content"])

# Entrada
pergunta_usuario = st.chat_input("Digite sua pergunta")

if pergunta_usuario:

    st.session_state.mensagens.append({
        "role": "user",
        "content": pergunta_usuario
    })

    with st.chat_message("user"):
        st.write(pergunta_usuario)

    try:
        with st.chat_message("assistant"):

            resposta = client.chat.completions.create(
                model="gpt-4o-mini",
                messages=st.session_state.mensagens
            )

            resposta_ia = resposta.choices[0].message.content

            st.write(resposta_ia)

            st.session_state.mensagens.append({
                "role": "assistant",
                "content": resposta_ia
            })

    except Exception as erro:
        st.error(str(erro))

# Limpar
if st.button("Limpar conversa"):
    del st.session_state["mensagens"]
    st.rerun()

