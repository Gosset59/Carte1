# Carte1
import streamlit as st
import pandas as pd
import folium
from folium.plugins import MarkerCluster
from colorama import init, Fore, Style

# Initialisation de Colorama pour la coloration du texte
init(autoreset=True)

# Titre de l'application
st.title("Carte des taux de réussite au brevet avec mention très bien")

# Upload du fichier CSV
uploaded_file = st.file_uploader("Téléchargez un fichier CSV", type=["csv"])

if uploaded_file is not None:
    # Lecture du fichier CSV en utilisant pandas
    df = pd.read_csv(uploaded_file)

    # Création d'une carte Folium avec un fond de plan OpenStreetMap
    m = folium.Map(location=[46.603354, 1.888334], zoom_start=6, tiles='Stamen Terrain')

    # Ajout des points avec un dégradé de couleur en fonction du taux de réussite
    marker_cluster = MarkerCluster().add_to(m)
    max_taux = df['Taux de Mention TB'].max()
    for index, row in df.iterrows():
        taux = row['Taux de Mention TB']
        taux_normalized = taux / max_taux
        color = f"#{int(255*(1-taux_normalized)):02X}{int(255*taux_normalized):02X}00"
        marker = folium.Marker(location=[row['Latitude'], row['Longitude']], icon=folium.Icon(color=color))
        marker.add_to(marker_cluster)

        # Affichage des informations au survol du point
        marker.add_child(folium.Popup(f"{Fore.GREEN}Collègue: {row['Collègue']}{Style.RESET_ALL}\n"
                                     f"Taux de Mention TB: {Fore.RED if taux == 0 else Fore.GREEN}{taux}%{Style.RESET_ALL}"))

    # Affichage de la carte
    st.subheader("Carte des taux de réussite au brevet avec mention très bien")
    st.write(f"Dégradé de couleur : {Fore.RED}0%{Style.RESET_ALL} à {Fore.GREEN}100%{Style.RESET_ALL}")
    st.write(f"Max Taux de Mention TB : {max_taux}")
    folium_static(m)
