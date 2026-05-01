# 📘 Plan de Projet : Audit de performance d'un SSL VPN

**Nom du Projet :** Projet 9  
**Objet :** Mesure d'impact (Débit, Latence, CPU) d'un tunnel SSL VPN (SoftEther)  
**Outils :** GNS3, Ubuntu, SoftEther, iperf3, ping, htop, Wireshark  
**Date de début :** `[AAAA-MM-JJ]`  
**Auteur :** `[Ton Nom]`

---

## 1. Introduction et Objectifs

### 1.1 Contexte
Dans un environnement réseau moderne, l'utilisation de VPN permet de sécuriser les communications. Cependant, le chiffrement et le encapsulage ont un coût en termes de ressources système et de bande passante. Cet audit vise à quantifier ce coût.

### 1.2 Objectifs Spécifiques
*   [ ] Installer et configurer un serveur SoftEther sur une machine Ubuntu dans GNS3.
*   [ ] Établir une connexion tunnel SSL entre deux nœuds.
*   [ ] Mesurer le débit maximal sans VPN (Ligne directe).
*   [ ] Mesurer le débit maximal avec VPN (Tunnel chiffré).
*   [ ] Observer la latence (Ping) dans les deux scénarios.
*   [ ] Analyser la charge CPU lors des transferts de données.
*   [ ] Produire des graphiques comparatifs.

---

## 2. Topologie de Labo (GNS3)

### 2.1 Schéma Réseau
> **Consigne :** Dessinez ou décrivez ici votre topologie GNS3.
> *Exemple recommandé :*
> *   **Node A (Client) :** Ubuntu (Source du trafic)
> *   **Node B (Serveur VPN / Cible) :** Ubuntu (Hébergeant SoftEther Server)
> *   **Lien :** Câble direct ou via Switch générique dans GNS3 pour simuler le LAN.

*(Insérer ici une capture d'écran de la topologie GNS3)*

### 2.2 Adressage IP Prévu
| Nœud | Rôle | Interface | Adresse IP | Sous-masque |
| :--- | :--- | :--- | :--- | :--- |
| Node A | Client | eth0 | `192.168.10.10` | `/24` |
| Node B | Serveur Physique | eth0 | `192.168.10.20` | `/24` |
| Node B | Interface VPN Virtuelle | tun/vtun | `10.0.0.1` | `/24` |
| Node A | Client après VPN | tun/vtun | `10.0.0.2` | `/24` |

---

## 3. Déploiement et Configuration

### 3.1 Installation de SoftEther (Node B)
*   [ ] Mettre à jour les paquets : `sudo apt update && sudo apt upgrade -y`
*   [ ] Télécharger les binaires SoftEther depuis le site officiel ou utiliser `git clone` si disponible.
*   [ ] Compiler/Installer selon la documentation officielle.
*   [ ] Lancer le service : `./vpnserver start`
*   [ ] Configuration initiale (`vpncmd`) :
    *   Créer un HUB virtuel.
    *   Configurer le port SSL (default 443).
    *   Activer la fonctionnalité "SecureNAT" ou définir un DHCP virtuel si nécessaire.

### 3.2 Configuration du Client (Node A)
*   [ ] Installer le client SoftEther (ou `openconnect` compatible).
*   [ ] Enregistrer les identifiants pour la connexion automatique.
*   [ ] Vérifier la connectivité physique avant activation du VPN : `ping 192.168.10.20`

---

## 4. Protocole de Test (Méthodologie)

> **Important :** Exécutez les tests dans l'ordre et notez toutes les valeurs brutes immédiatement.

### Phase 1 : Scénario Baseline (Sans VPN)
*L'objectif est d'avoir une référence maximale du réseau.*

1.  **Tests de Latence :**
    *   Commande : `ping -c 100 192.168.10.20`
    *   Données à relever : Moyenne (ms), Jitter, Packet Loss (%).
2.  **Tests de Débit (Bandwidth) :**
    *   Sur Node B (Serveur) : `iperf3 -s`
    *   Sur Node A (Client) : `iperf3 -c 192.168.10.20 -t 60 -P 4` (4 threads pour saturer)
    *   Durée : 60 secondes.
    *   Répéter : 3 fois pour moyenne.
3.  **Observation Système :**
    *   Ouvrir `htop` sur Node A et Node B.
    *   Prendre une capture d'écran pendant le test iperf montrant le %CPU global.

### Phase 2 : Scénario VPN (Avec Tunnel)
*Configuration active du VPN SSL.*

1.  **Connecter le Tunnel :**
    *   Lancer la connexion VPN depuis Node A vers Node B.
    *   Vérifier l'IP virtuelle attribuée (ex: `10.0.0.2`).
2.  **Tests de Latence (à travers le tunnel) :**
    *   Commande : `ping -c 100 10.0.0.1` (Interface Gateway VPN du serveur)
    *   Données à relever : Moyenne, Jitter, Packet Loss.
3.  **Tests de Débit (à travers le tunnel) :**
    *   Sur Node B (Écoute sur l'interface virtuelle) : `iperf3 -s -B 10.0.0.1`
    *   Sur Node A : `iperf3 -c 10.0.0.1 -t 60 -P 4`
    *   Durée : 60 secondes.
    *   Répéter : 3 fois pour moyenne.
4.  **Observation Système (CPU) :**
    *   Noter le pic d'utilisation CPU (%) durant le transfert chiffré.

### Phase 3 : Analyse Fine (Wireshark)
1.  Lancer Wireshark sur Node A.
2.  Capturer le trafic sur l'interface physique ET l'interface VPN.
3.  Filtrer par protocole SSL/TLS ou UDP (selon configuration).
4.  Comparer la taille des trames : La trame physique est plus grande à cause de l'en-tête VPN ?

---

## 5. Résultats et Collecte de Données

> **Instructions :** Remplissez les tableaux ci-dessous avec vos mesures réelles.

### Tableau 1 : Synthèse des Performances
| Métrique | Sans VPN (Direct) | Avec SSL VPN | Écart (%) | Observation |
| :--- | :--- | :--- | :--- | :--- |
| **Débit Moyen** (Mbps) | ... | ... | ... | ... |
| **Latence Moyenne** (ms) | ... | ... | ... | ... |
| **Jitter** (ms) | ... | ... | ... | ... |
| **Packet Loss** (%) | ... | ... | ... | ... |
| **Charge CPU (Serveur)** (%) | ... | ... | ... | ... |
| **Charge CPU (Client)** (%) | ... | ... | ... | ... |

### Graphiques Requis
*(Insérer ici des captures d'écran générées ou des graphes Excel/Python créés à partir des logs iperf)*

1.  **Graphique 1 :** Comparaison Barres (Débit Sans vs Avec).
2.  **Graphique 2 :** Courbe de Latence au cours du temps (si analyse temporelle faite).
3.  **Capture Wireshark :** Montrer la différence de structure de paquet (optionnel mais bonus).

---

## 6. Analyse et Interprétation

### 6.1 Analyse des écarts de débit
> **Guide d'écriture :** Pourquoi le débit a-t-il baissé ?
> *   Le chiffrement (AES, etc.) prend du temps processeur.
> *   L'encapsulation ajoute des octets (Overhead) aux paquets.
> *   Est-ce acceptable pour un usage entreprise (email, web) ? Pour du streaming vidéo ?

### 6.2 Impact CPU
> **Guide d'écriture :** Quel est le ratio coût/bénéfice ?
> *   Si le CPU dépasse X%, cela peut devenir un goulot d'étranglement.
> *   La sécurité a un prix : valider cette perte.

### 6.3 Analyse Wireshark (Optionnel)
> Avez-vous remarqué une fragmentation des paquets ? L'estimation de la fenêtre TCP change-t-elle ?

---

## 7. Conclusion Argumentée

### 7.1 Résumé des performances
En quelques lignes, résumez si SoftEther tient ses promesses en terme de performance brute dans ce scénario simulé.

### 7.2 Recommandations
Sur la base de vos résultats, que recommanderiez-vous à un administrateur ?
*   *Exemple : "Utiliser ce VPN pour l'accès bureautique, mais envisager IPsec Hardware pour du transfert de gros fichiers."*

### 7.3 Limitations de l'étude
*   Le simulateur GNS3 influence-t-il les résultats (virtualisation imbriquée) ?
*   Le manque de variables aléatoires (réseau réel) ?

---

## 8. Annexes

*   **Commandes utilisées :** Copiez-collez ici les commandes exactes qui ont servi.
*   **Fichiers Logs :** Indiquer où sont sauvegardés les logs `iperf` (`results_baseline.txt`, `results_vpn.txt`).
*   **Références :** Liens vers la doc SoftEther, documentation iperf, etc.

---

### 💡 Conseils du guide pour réussir ce projet :

1.  **Prépare ton script :** Ne tape pas les commandes à la main pendant le test `iperf`. Utilise un petit script Bash qui lance le test, attend la fin, et sauvegarde le résultat dans un fichier CSV. Cela t'évitera des erreurs humaines de saisie.
2.  **Isolation :** Assure-toi qu'aucune autre machine ne consomme de la bande passante dans ton labo GNS3 pendant les tests.
3.  **Visualisation :** Pour les graphiques, tu peux extraire les données de `iperf3` et les mettre dans Excel, ou utiliser Python (matplotlib) si tu veux impressionner le correcteur.
4.  **SoftEther :** Attention, SoftEther utilise souvent HTTPS (port 443) par défaut pour masquer le flux. Vérifie bien que ton pare-feu (iptables) laisse passer le port 443 entre les deux VMs GNS3 sinon le VPN ne se connectera pas.
5.  **Sécurité :** Dans l'analyse, mentionne toujours que la baisse de perf s'accompagne d'une augmentation significative de la **confidentialité** et de la **confiance**, c'est le compromis inhérent à la cybersécurité.
