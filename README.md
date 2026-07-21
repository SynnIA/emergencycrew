<div align="center">

# 🚨 Emergency Crew

### La station coule. Vous êtes l'équipe de maintenance. **Un seul** sera Employé du Mois.

*Un jeu multijoueur coopératif — où la coopération est un piège.*

<br/>

![Node.js](https://img.shields.io/badge/Node.js-22-339933?logo=node.js&logoColor=white)
![Colyseus](https://img.shields.io/badge/Colyseus-0.15-2f2f2f?logo=colyseus&logoColor=white)
![Phaser](https://img.shields.io/badge/Phaser-3.60-8e44ad?logo=phaser)
![Express](https://img.shields.io/badge/Express-4-000000?logo=express&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Render%20%2B%20Vercel-2496ED?logo=docker&logoColor=white)
![Statut](https://img.shields.io/badge/statut-prototype%20jouable-orange)
![Tests](https://img.shields.io/badge/tests-0-red)

</div>

---

## La genèse — « Le Grand Dilemme »

Tout part d'une idée simple et un peu cruelle : **et si un jeu de coopération récompensait la trahison ?**

Dans *Emergency Crew*, une équipe de techniciens est coincée dans une station qui s'effondre — spatiale, sous-marine, souterraine, peu importe : elle tombe en ruines et le chrono ne s'arrête jamais. Fuite de gaz, court-circuit, surchauffe : les pannes s'enchaînent et la **jauge de Stabilité** descend. Si elle atteint zéro, la station explose et **tout le monde perd**.

Voilà le piège. Car à la fin, si la base est sauvée, il n'y a **qu'un seul gagnant** : celui qui a accumulé le plus de *Crédits de Maintenance*. L'« Employé du Mois ».

Alors chaque joueur vit la même tension permanente : *dois-je réparer avec les autres pour survivre, ou saboter mes collègues pour être le seul héros ?* On ne se bat pas pour tuer — on se bat pour **interrompre**. Un coup de clé bien placé réinitialise la réparation d'un rival, lui fait lâcher sa pièce, le laisse sonné pendant que vous prenez sa place. Mais passez trop de temps à vous entre-déchirer, et la station vous engloutit tous.

Ce point d'équilibre instable entre l'entraide et l'égoïsme, c'est **le Grand Dilemme** — et c'est tout le sel du jeu.

---

## Ce que ça fait

- 🎮 **Multijoueur temps réel** jusqu'à 4 joueurs par salle, avec code de salle à partager (créer / rejoindre).
- 🗺️ **Station isométrique 2D** : une carte de 30×24 tuiles, 9 salles, 16 machines à maintenir.
- 🔧 **Maintenance sous pression** : les urgences (fuite de gaz, court-circuit, surchauffe) apparaissent en continu ; il faut trouver le bon objet dans la Salle de Stockage et réparer avant l'explosion. Les objets sont volontairement **trop rares pour tout le monde** — le conflit est inévitable.
- 🥊 **Sabotage entre collègues** : coup de clé (éjection + interruption), dash-plaquage (renversement), vol de pièce, états *sonné* / *renversé*.
- ⚡ **Power-ups & armes spéciales** : vitesse, bouclier, réparation turbo, invisibilité, shockwave — plus quelques joujoux « admin » (laser, bombe nucléaire, gel de zone, téléportation).
- 🏆 **Écran de fin** avec podium : l'Employé du Mois, le Saboteur, le Stagiaire.
- 🎨 **Confort de jeu** : lobby, tutoriel, kill feed, émotes, cosmétiques (chapeaux/accessoires/personnages), minimap, ambiance sonore, interface **bilingue FR / EN**.

---

## Sous le capot

L'architecture repose sur une règle non négociable : **le serveur fait autorité**. Dans un jeu où l'on peut saboter les autres, hors de question de laisser un client mentir sur sa position ou ses coups.

**Serveur — Node.js + Colyseus (temps réel autoritaire)**
- `GameRoom.js` (~1 300 lignes) concentre **toute** la logique : physique, collisions, combat, réparations, urgences, power-ups et score. Une boucle de simulation tourne à **20 Hz** (`setSimulationInterval`), le client ne fait qu'envoyer ses intentions (`input`) et afficher l'état qu'on lui renvoie.
- L'état du monde est décrit avec **`@colyseus/schema`** : des classes typées (`Player`, `Machine`, `GameItem`, `Emergency`, `PowerUp`) rangées dans des `MapSchema`, plus un état racine (`stability`, `timer`, `phase`, `hostId`). Colyseus ne transmet que les **diffs binaires** à chaque tick — c'est ce qui rend la synchro fluide.
- `Express` sert les fichiers statiques du client et les assets, et expose un `/health` pour les plateformes d'hébergement. Transport WebSocket via `@colyseus/ws-transport`.

**Client — Phaser 3 + HTML/CSS + colyseus.js**
- Le **gameplay** (rendu isométrique, tuiles 64×32, caméra qui suit, minimap) tourne dans une scène **Phaser 3.60** (`GameScene`, `TutorialScene`).
- Toute l'**UI** (menus, lobby, join, podium) est en **HTML/CSS vanilla** pilotée par un `ScreenManager` — pas de framework front lourd.
- La connexion se fait via **`colyseus.js`** ; le client bascule automatiquement entre `ws://localhost:3000` en dev et le serveur de prod en `wss://`.
- Phaser et colyseus.js sont chargés **depuis CDN** (jsDelivr / unpkg), pas de bundler.

**Déploiement**
- `Dockerfile` (`node:22-slim`) qui empaquette serveur + client + assets.
- **Render** héberge le serveur temps réel (`render.yaml`, plan gratuit, `/health`).
- **Vercel** sert le client statique (`vercel.json`).
- Des scripts VPS alternatifs sont fournis (`deploy/` : `nginx.conf`, service systemd, `setup.sh`, `deploy.sh`).

**Compromis assumés**
- `GameRoom.js` est un **monolithe** : tout le cœur de jeu dans un seul fichier. C'est direct à lire d'un bout à l'autre, mais c'est la principale dette technique.
- **Dépendances front par CDN** : zéro étape de build côté client, au prix d'une dépendance à la disponibilité du CDN et à des versions figées à la main.
- **Plan Render gratuit** : le serveur s'endort après inactivité → premier chargement lent (cold start).
- **Aucun test** (voir ci-dessous) : la logique est validée à la main, en jouant.

---

## État

> **Prototype jouable, déjà déployé.** Honnête et sans enjoliver.

- ✅ **Ça tourne et ça se joue** en multijoueur : serveur + client déployés (Docker / Render / Vercel).
- ✅ ~**4 180 lignes** de JavaScript de jeu (≈ 1 720 côté serveur, ≈ 2 460 côté client), ~5 700 lignes en comptant HTML/CSS.
- ⚠️ **0 test automatisé.** Aucun framework de test, aucune CI. La validation est entièrement manuelle. C'est le premier chantier pour fiabiliser le projet.
- ⚠️ **Prototype**, pas un produit fini : équilibrage à affiner, cœur de jeu concentré dans un gros fichier, dépendances front non bundlées.

**Prochains chantiers naturels :** premiers tests (physique/combat/score côté serveur), découpe de `GameRoom.js`, bundling du client, équilibrage.

---

## Crédits

Un projet **d'équipe**, développé à plusieurs mains.

- **Nathan Fernandes** ([@SynnIA](https://github.com/SynnIA)) — auteur principal. Il signe **la totalité des commits** de ce dépôt : architecture serveur autoritaire, boucle de jeu, combat/sabotage, power-ups, UI, déploiement.
- **Arthur** (dépôt [S-Mopty/Emergency-Crew](https://github.com/S-Mopty)) — co-auteur. Sa version (`version_Arthur`) a été **fusionnée** dans ce projet : la **carte 30×24** (9 salles, 16 machines) et la base de **localisation FR/EN** en viennent. Sa contribution est tracée directement dans le code (`server/src/MapData.js`, `client/js/locale.js`, `client/js/scenes/GameScene.js`) et dans l'historique git (commit « merge Arthur »).
- **Claude Opus (Anthropic)** — assistant de développement, présent en `Co-Authored-By` sur la quasi-totalité des commits.

> Note d'honnêteté : l'historique git de **ce** dépôt est mono-auteur (tous les commits sont signés `synnheal`), mais le code porte explicitement le travail fusionné d'Arthur / S-Mopty. Les crédits ci-dessus reflètent cette réalité, pas seulement les métadonnées git.

---

## Lancer en local

**Prérequis :** Node.js 18+ (22 recommandé).

```bash
# 1. Récupérer le code
git clone https://github.com/SynnIA/emergencycrew.git
cd emergencycrew

# 2. Installer les dépendances du serveur
cd server
npm install

# 3. Démarrer le serveur (il sert AUSSI le client)
npm start          # équivaut à : node src/index.js
```

Puis ouvrez **http://localhost:3000** dans votre navigateur. Le serveur Express sert le client statique et le jeu se connecte automatiquement en `ws://localhost:3000`.

Pour jouer à plusieurs : ouvrez plusieurs onglets/navigateurs, **créez une partie** dans l'un pour obtenir un code de salle, puis **rejoignez** avec ce code dans les autres.

**Variante Docker :**

```bash
docker build -t emergency-crew .
docker run -p 3000:3000 emergency-crew
# → http://localhost:3000
```

---

<div align="center">
<sub>Emergency Crew — un prototype qui explore une seule question : <em>jusqu'où trahiriez-vous vos collègues pour un titre d'Employé du Mois ?</em></sub>
</div>
