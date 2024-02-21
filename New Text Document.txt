import tkinter as tk
from tkinter import filedialog
import re
import requests

def nettoyer_et_reformater(fichier_entree, fichier_sortie):
    with open(fichier_entree, 'r', encoding='utf-8') as f_entree:
        contenu = f_entree.read()

    contenu_nettoye = re.sub(r'(https?://\S+\s*)|(\s*https?://\S+)', '', contenu)
    contenu_formate = re.sub(r',\s+', '\n', contenu_nettoye)
    lignes_uniques = list(dict.fromkeys(contenu_formate.split('\n')))
    lignes_filtrees = [ligne for ligne in lignes_uniques if len(ligne) <= 30]
    lignes_finales = [ligne for ligne in lignes_filtrees if '.txt' not in ligne]

    with open(fichier_sortie, 'w', encoding='utf-8') as f_sortie:
        for ligne in lignes_finales:
            f_sortie.write(f"{ligne}\n")

    return lignes_finales

def ouvrir_fichier():
    chemin_fichier = filedialog.askopenfilename()
    if chemin_fichier:
        entry_chemin.delete(0, tk.END)
        entry_chemin.insert(0, chemin_fichier)

def executer():
    fichier_entree = entry_chemin.get()
    if fichier_entree:
        resultats = nettoyer_et_reformater(fichier_entree, 'sortie.txt')
        label_statut.config(text="Traitement terminé !", fg="green")
        envoyer_discord(resultats)

def envoyer_discord(resultats):
    webhook_url = 'VOTRE_WEBHOOK'
    message = '\n'.join(resultats)
    fichier_sortie = 'resultats.txt'
    with open(fichier_sortie, 'w', encoding='utf-8') as f:
        f.write(message)
    files = {'file': open(fichier_sortie, 'rb')}
    requests.post(webhook_url, files=files)

# Création de la fenêtre principale
app = tk.Tk()
app.title("Combolist Creator By Wazzyx")
app.geometry("400x250")
app.configure(bg='black')
app.attributes('-alpha', 0.6)  # Opacité à 60%

# Éléments de l'interface
label_chemin = tk.Label(app, text="Chemin du fichier d'entrée:", font=("Arial", 12), bg='black', fg='white')
label_chemin.pack()

entry_chemin = tk.Entry(app, width=40, font=("Arial", 10))
entry_chemin.pack()

btn_ouvrir = tk.Button(app, text="Ouvrir", command=ouvrir_fichier, font=("Arial", 10))
btn_ouvrir.pack(pady=5)

btn_executer = tk.Button(app, text="Exécuter", command=executer, font=("Arial", 12))
btn_executer.pack(pady=5)

label_statut = tk.Label(app, text="", font=("Arial", 12), bg='black', fg='white')
label_statut.pack()

# Lancement de l'application
app.mainloop()
