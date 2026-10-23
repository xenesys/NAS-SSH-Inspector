# NAS-SSH-Inspector
A travers une connexion SSH, vous accédez à quelques outils d'inspection de votre NAS en local.
Interface graphique simple, cadre pour lire les résultats des commandes.

Voici la liste des outils et commandes intégrés dans "NAS SSH Inspector" :

### Outils de diagnostic et de monitoring rapide (Menu déroulant)

* **Espace disque (`df -h`)** : Visualisation en temps réel de l'occupation des volumes, des partitions et des points de montage.
* **Liste détaillée des disques (`lsblk`)** : Affichage structuré des disques connectés (nom, modèle, numéro de série, taille) et peuplement automatique du sélecteur de disques.
* **Gestion acoustique (`hdparm`)** : Interrogation des paramètres de performance/bruit d'un disque spécifique sélectionné dans l'interface.
* **Statut de santé SMART (`smartctl -H`)** : Vérification rapide de l'état de santé global d'un disque cible (Passed/Failed).
* **Température des disques (`smartctl -A`)** : Relevé thermique ciblé pour les disques NVMe et SATA.
* **Mémoire & RAM (`free -h`)** : Suivi de l'utilisation de la mémoire vive (RAM) et de l'espace d'échange (Swap).
* **Charge CPU & Uptime (`uptime` + `top`)** : Analyse de la charge processeur, du temps de fonctionnement de la machine et aperçu des processus actifs.
* **Services actifs (`systemctl`)** : Liste de l'ensemble des services système en cours d'exécution.

### Fonctionnalités de l'interface

* **Champ de commande personnalisée** : Permet de saisir et d'exécuter à la volée n'importe quelle commande Linux en ligne de texte libre.
* **Gestion automatisée des privilèges (`sudo -S`)** : Sécurisation de l'exécution des commandes bas niveau nécessitant des droits administrateur, avec injection transparente du mot de passe.
* **Console de sortie dédiée** : Fenêtre de type terminal intégrée avec coloration syntaxique (fond sombre, texte vert) et retour visuel des erreurs SSH filtrées.

Testé sur un NAS Ugreen DXP6800 Plus & UGOS PRO (v1.20).

<img width="846" height="698" alt="image" src="https://github.com/user-attachments/assets/d3aa9947-83e0-45cd-a8ed-dfff63cce9ba" />
