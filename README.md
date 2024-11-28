%%writefile app.py
import streamlit as st
import requests
import pandas as pd
import matplotlib.pyplot as plt

# URL de la API
API_URL = "https://restcountries.com/v3.1/all"

# Obtener datos de la API
response = requests.get(API_URL)
if response.status_code == 200:
    data = response.json()
else:
    st.error(f"Error al conectar con la API: {response.status_code}")

# Procesar datos en un DataFrame
countries = []
for country in data:
    countries.append({
        "Nombre": country.get("name", {}).get("common", "N/A"),
        "Región": country.get("region", "N/A"),
        "Población": country.get("population", 0),
        "Área": country.get("area", 0),
        "Fronteras": len(country.get("borders", [])),
        "Idiomas": ", ".join(country.get("languages", {}).values()) if "languages" in country else "N/A",
        "Bandera": country.get("flags", {}).get("png", "")
    })

df = pd.DataFrame(countries)

# Configuración de la aplicación
st.set_page_config(page_title="Visualización de Países", layout="wide")

# Título y descripción
st.title("Visualización Interactiva de Países")
st.write("Explora datos de países usando la API REST Countries.")

# Menú de selección
page = st.sidebar.selectbox("Selecciona una opción", ["Resumen", "Gráficos", "Detalles"])

# Resumen de datos
if page == "Resumen":
    st.subheader("Datos Generales de los Países")
    st.dataframe(df)

# Gráficos interactivos
elif page == "Gráficos":
    st.subheader("Gráficos Interactivos")

    # Densidad poblacional
    df["Densidad"] = df["Población"] / df["Área"]
    fig, ax = plt.subplots()
    ax.scatter(df["Área"], df["Población"], alpha=0.5)
    ax.set_xlabel("Área (km²)")
    ax.set_ylabel("Población")
    ax.set_title("Relación entre Área y Población")
    st.pyplot(fig)

# Detalles específicos
elif page == "Detalles":
    st.subheader("Información Detallada")
    pais = st.selectbox("Selecciona un país", df["Nombre"])
    detalles = df[df["Nombre"] == pais].iloc[0]
    st.write(f"**Región:** {detalles['Región']}")
    st.write(f"**Población:** {detalles['Población']}")
    st.write(f"**Área:** {detalles['Área']} km²")
    st.write(f"**Fronteras:** {detalles['Fronteras']}")
    st.write(f"**Idiomas:** {detalles['Idiomas']}")
    st.image(detalles["Bandera"], caption=f"Bandera de {detalles['Nombre']}")
