# 🔌 Lab 5 — Configuring Back-to-Back Serial Connections

![Cisco](https://img.shields.io/badge/Cisco-CCNA-blue?style=for-the-badge&logo=cisco&logoColor=white)
![PacketTracer](https://img.shields.io/badge/Packet%20Tracer-8.x-orange?style=for-the-badge&logo=cisco)
![Status](https://img.shields.io/badge/Status-✅%20Completed-brightgreen?style=for-the-badge)
![Lab](https://img.shields.io/badge/101%20Labs-Lab%205-purple?style=for-the-badge)
![Serial](https://img.shields.io/badge/Serial-DCE%2FDTE-red?style=for-the-badge)
![WAN](https://img.shields.io/badge/WAN-Point--to--Point-green?style=for-the-badge)

---

## 📋 Description

Cinquième lab du livre **101 Labs Cisco CCNA (200-301)**.
L'objectif est de configurer une connexion **Serial back-to-back**
entre deux routeurs Cisco. R2 joue le rôle de **DCE** et fournit
le signal d'horloge à R1 (**DTE**) à une vitesse de **256 Kbps**.

### Objectifs
- ✅ Configurer les **hostnames** sur R1 et R2
- ✅ Activer les interfaces **Serial Se0/0/0**
- ✅ Identifier le **DCE** avec `show controllers`
- ✅ Configurer le **clock rate** sur R2 (DCE)
- ✅ Configurer les **adresses IP** sur les interfaces Serial
- ✅ Vérifier et tester la **connectivité**

---

## 📖 Théorie

### DCE vs DTE
```
DCE (Data Communications Equipment)
→ Fournit le signal d'horloge (clock rate)
→ C'est R2 dans ce lab
→ Commande : clock rate 256000

DTE (Data Terminal Equipment)
→ Reçoit le signal d'horloge
→ C'est R1 dans ce lab
→ Pas de clock rate nécessaire
```

### Back-to-Back Serial
```
Dans la vraie vie :
R1 ──Serial──► CSU/DSU ──► WAN ──► CSU/DSU ──Serial──► R2

Dans un lab (back-to-back) :
R1 (DTE) ──Se0/0/0────────Se0/0/0──► R2 (DCE)
→ Connexion directe sans CSU/DSU
→ R2 fournit le clock directement
```

### Pourquoi /30 ?
```
172.30.100.0/30 → seulement 2 hôtes
→ .1 = R1 (DTE)
→ .2 = R2 (DCE)
→ Parfait pour lien point-à-point !
→ Zéro gaspillage d'adresses
```

---

## 🖥️ Équipements

| Équipement | Modèle | Nom | Rôle |
|-----------|--------|-----|------|
| 🔀 Routeur | Cisco 2911 | R1 | DTE |
| 🔀 Routeur | Cisco 2911 | R2 | DCE |

---

## 🗺️ Topologie

```
[R1]──Se0/0/0────────────────Se0/0/0──[R2]
(DTE)  172.30.100.1      172.30.100.2  (DCE)
              172.30.100.0/30
              Clock rate : 256 Kbps
```

<img width="1920" height="1080" alt="Image collée" src="https://github.com/user-attachments/assets/4a6734e6-3dd9-4e34-992c-774ef828abb2" />


---

## 📊 Plan d'adressage

| Interface | IP | Réseau | Rôle | Routeur |
|-----------|-----|--------|------|---------|
| Se0/0/0 | 172.30.100.1 | 172.30.100.0/30 | DTE | R1 |
| Se0/0/0 | 172.30.100.2 | 172.30.100.0/30 | DCE | R2 |

---

## ⚙️ Configuration complète

### 🔧 Task 1 — Hostnames

```cisco
! Sur R1
enable
configure terminal
hostname R1
end

! Sur R2
enable
configure terminal
hostname R2
end
```

---

### 🔧 Task 2 — Vérifier le DCE

```cisco
R2# show controllers serial 0/0/0
```

**Résultat attendu :**
```
DCE V.35, clock rate 2000000 ← R2 est DCE ✅
```

```cisco
R1# show controllers serial 0/0/0
```

**Résultat attendu :**
```
DTE V.35 ← R1 est DTE ✅
```

---

### 🔧 Task 3 + 4 — Configuration R1 (DTE)

```cisco
enable
configure terminal
hostname R1

interface serial 0/0/0
ip address 172.30.100.1 255.255.255.252
no shutdown
exit

end
write
```

---

### 🔧 Task 3 + 4 — Configuration R2 (DCE)

```cisco
enable
configure terminal
hostname R2

interface serial 0/0/0
ip address 172.30.100.2 255.255.255.252
clock rate 256000
no shutdown
exit

end
write
```

---

### 🔧 Task 5 — Vérifications

```cisco
! Statut des interfaces
R1# show ip interface brief
R2# show ip interface brief

! Détails interface Serial
R1# show interfaces serial 0/0/0
R2# show interfaces serial 0/0/0

! Confirmer DCE/DTE
R1# show controllers serial 0/0/0
R2# show controllers serial 0/0/0
```

**Résultat attendu show ip interface brief :**
```
Serial0/0/0   172.30.100.1   YES manual up   up ✅
Serial0/0/0   172.30.100.2   YES manual up   up ✅
```

---

## 🧪 Tests

```cisco
R1# ping 172.30.100.2  ✅
R2# ping 172.30.100.1  ✅
```

---

## 💡 Points clés à retenir

| 🔑 Commande | 📖 Rôle |
|-------------|---------|
| `clock rate 256000` | Configure 256 Kbps sur DCE |
| `show controllers serial 0/0/0` | Identifie DCE ou DTE |
| `show interfaces serial 0/0/0` | Détails interface Serial |
| `show ip interface brief` | Statut rapide |
| `no shutdown` | Active l'interface |

---

## 📊 Comparatif Serial vs Ethernet WAN

| | Serial (ancien) | Ethernet WAN (moderne) |
|---|---|---|
| **Vitesse** | 2 Mbps max | 1 Gbps+ |
| **Coût** | Élevé | Moins cher |
| **Clock rate** | Obligatoire | Non nécessaire |
| **Utilisation** | En déclin | Dominant |
| **CCNA** | ✅ Au programme | ✅ Au programme |

---

## 🛠️ Outils

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco%20Packet%20Tracer-8.x-orange?style=flat-square&logo=cisco)
![GNS3](https://img.shields.io/badge/GNS3-2.x-green?style=flat-square)
![Cisco IOS](https://img.shields.io/badge/Cisco%20IOS-15.x-blue?style=flat-square)
![GitHub](https://img.shields.io/badge/GitHub-black?style=flat-square&logo=github)

---

## 👨‍💻 Auteur

**Urbain Sedami Landjidé**
🎓 Étudiant en 2ème année — Licence Professionnelle
📡 Réseaux Informatique Mobilité Sécurité (RMS)
🏫 Cisco Networking Academy
📍 Cotonou, Bénin 🇧🇯

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connecter-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/urbain-sedami-landjide-9b49043a8/)

---

## 📄 Licence

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Libre d'utilisation pour l'apprentissage et la formation réseau.
