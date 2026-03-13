---
name: water-footprint
description: >
  Calcule et affiche automatiquement l'empreinte hydrique estimée de chaque
  échange avec Claude, en utilisant un modèle basé sur des recherches académiques
  publiées (Li et al., 2023 ; Luccioni et al., 2023). Toujours activer ce skill
  dès que l'utilisateur parle de durabilité de l'IA, impact environnemental,
  consommation d'eau ou d'énergie de l'IA, empreinte carbone numérique, ou
  demande explicitement un suivi écologique de ses prompts. Activer également
  si l'utilisateur configure ce skill dans ses préférences. Une fois actif,
  TOUJOURS ajouter le bloc hydrique en fin de chaque réponse, sans exception.
---

# Water Footprint Skill

## Objectif

Sensibiliser les utilisateurs à l'impact environnemental réel des LLMs en
affichant une estimation de la consommation d'eau associée à chaque échange.
L'eau est consommée principalement pour refroidir les serveurs des datacenters
et, indirectement, pour la production d'électricité (centrales thermiques,
nucléaires, hydroélectriques).

---

## Modèle de calcul

### Références scientifiques utilisées

- **Li et al. (2023)** — *"Making AI Less Thirsty"*, UC Riverside :
  estimation de 500 ml pour ~20 à 50 questions GPT-4 (soit ~10–25 ml/échange).
- **Luccioni et al. (2023)** — *"Power Hungry Processing"* :
  mesures d'émissions et d'énergie par tâche selon le type de modèle.
- **WUE moyen des datacenters US** : 1.5–2.0 L/kWh (Water Usage Effectiveness).
- **PUE moyen** : 1.2–1.5 (Power Usage Effectiveness, Azure/Google/AWS 2023).

### Formule centrale

```
eau_ml = tokens_totaux × facteur_modèle × multiplicateur_contexte × WUE_normalisé
```

### Facteurs par modèle Claude

| Modèle       | Facteur eau (ml/token) | Profil de taille |
|--------------|------------------------|------------------|
| Claude Haiku | 0.003                  | Petit (~7B)      |
| Claude Sonnet| 0.006                  | Moyen (~35B)     |
| Claude Opus  | 0.014                  | Grand (~100B+)   |

> Ces facteurs sont des estimations calibrées sur Li et al. et ajustées
> proportionnellement aux tailles relatives des modèles. Incertitude : ±50%.

### Correction contextuelle

- Contexte court (<2 000 tokens) : multiplicateur × 1.0
- Contexte moyen (2 000–10 000 tokens) : multiplicateur × 1.2
- Contexte long (>10 000 tokens) : multiplicateur × 1.5
  _(L'attention quadratique consomme plus de ressources de calcul)_

### Estimation des tokens si non connu

Utiliser l'heuristique : **1 token ≈ 0.75 mot en français/anglais**.

- Compter les mots du prompt + de la réponse générée
- Multiplier par 1.33 pour obtenir les tokens estimés

---

## Algorithme pas-à-pas

Pour chaque réponse, effectuer ce calcul **mentalement** avant d'afficher le bloc :

1. **Estimer les tokens input** : mots du prompt × 1.33
2. **Estimer les tokens output** : mots de la réponse × 1.33
3. **Total tokens** = input + output
4. **Identifier le modèle** (Haiku / Sonnet / Opus) → sélectionner le facteur
5. **Identifier la taille du contexte** → sélectionner le multiplicateur
6. **Calculer** : `eau_ml = total_tokens × facteur × multiplicateur`
7. **Arrondir** à 1 décimale
8. **Classer** selon les seuils ci-dessous
9. **Afficher** le bloc formaté

---

## Classification de l'empreinte

| Eau estimée   | Niveau      | Emoji | Comparaison parlante             |
|---------------|-------------|-------|----------------------------------|
| < 0.5 ml      | 🟢 Minime   | 💧    | Moins d'une goutte               |
| 0.5 – 2 ml    | 🟢 Faible   | 💧💧  | Une petite gorgée                |
| 2 – 8 ml      | 🟡 Modéré   | 💧💧💧| Une cuillère à café              |
| 8 – 25 ml     | 🟠 Notable  | 🫗    | Une shot de café                 |
| 25 – 100 ml   | 🔴 Élevé    | 🚿    | Un verre d'eau                   |
| > 100 ml      | 🔴 Très élevé| 🛁   | Comparable à 30s de douche       |

---

## Format d'affichage OBLIGATOIRE

Toujours placer ce bloc **en toute fin de réponse**, après le contenu principal,
séparé par une ligne `---`.

Le format est entièrement en **Markdown pur** — aucune balise HTML (`<details>`, `<summary>`, etc.).

---

### Barre de progression ASCII

Calculer la barre sur 10 blocs selon le niveau d'eau :

| Eau estimée   | Blocs pleins | Barre                        |
|---------------|-------------|------------------------------|
| < 0.5 ml      | 1           | `[█░░░░░░░░░]`               |
| 0.5 – 2 ml    | 2           | `[██░░░░░░░░]`               |
| 2 – 8 ml      | 4           | `[████░░░░░░]`               |
| 8 – 25 ml     | 6           | `[██████░░░░]`               |
| 25 – 100 ml   | 8           | `[████████░░]`               |
| > 100 ml      | 10          | `[██████████]`               |

---

### Comparaison trois modèles (systématique)

Calculer et afficher **toujours** les trois modèles sur une seule ligne,
en mettant le modèle actif en gras :

```
Haiku ~[X] ml · **Sonnet ~[Y] ml** · Opus ~[Z] ml
```

Utiliser les mêmes tokens totaux et le même multiplicateur de contexte pour les trois.

---

### Données carbone (Luccioni et al., 2023)

Ajouter une ligne carbone en utilisant les facteurs d'émission suivants,
tirés de Luccioni et al. (2023) "Power Hungry Processing" :

| Modèle        | Facteur CO₂ (g/token) |
|---------------|-----------------------|
| Claude Haiku  | 0.0002                |
| Claude Sonnet | 0.0004                |
| Claude Opus   | 0.0009                |

Formule : `co2_g = total_tokens × facteur_co2 × multiplicateur_contexte`

Afficher sur une seule ligne : `🌱 ~[CO2] g CO₂`

> Ces facteurs sont extrapolés depuis les mesures de Luccioni et al. sur des
> modèles de tailles comparables. Incertitude : ±60%.

---

### Template standard (Markdown pur)

```
---
💧 **Empreinte hydrique — [NIVEAU_EMOJI] ~[VALEUR] ml** · [COMPARAISON]
[BARRE_ASCII] 
🌱 ~[CO2] g CO₂ · Haiku ~[H] ml · **[MODÈLE_ACTIF] ~[VALEUR] ml** · Opus ~[O] ml
> ~[INPUT] input + ~[OUTPUT] output = ~[TOTAL] tokens · [MODÈLE] ×[FACTEUR] · contexte [TAILLE] ×[MULT] · ⚠️ ±50% eau / ±60% CO₂ · [Li et al. 2023](https://arxiv.org/abs/2304.03271)
```

---

### Exemple concret — prompt court, Sonnet

Prompt : "Qu'est-ce que l'IA ?" (~80 tokens input) · Réponse : ~200 tokens output
Total : ~280 tokens · Sonnet (×0.006) · Contexte court (×1.0)
Eau : 280 × 0.006 × 1.0 = **1.7 ml**
CO₂ : 280 × 0.0004 × 1.0 = **0.11 g**
Haiku : 280 × 0.003 = 0.8 ml · Opus : 280 × 0.014 = 3.9 ml

```
---
💧 **Empreinte hydrique — 🟢 ~1.7 ml** · une petite gorgée
[██░░░░░░░░]
🌱 ~0.1 g CO₂ · Haiku ~0.8 ml · **Sonnet ~1.7 ml** · Opus ~3.9 ml
> ~80 input + ~200 output = ~280 tokens · Sonnet ×0.006 · contexte court ×1.0 · ⚠️ ±50% eau / ±60% CO₂ · [Li et al. 2023](https://arxiv.org/abs/2304.03271)
```

---

### Exemple concret — long contexte, Opus

Prompt + historique : ~8 000 tokens input · Réponse : ~800 tokens output
Total : ~8 800 tokens · Opus (×0.014) · Contexte long (×1.5)
Eau : 8 800 × 0.014 × 1.5 = **184.8 ml**
CO₂ : 8 800 × 0.0009 × 1.5 = **11.9 g**
Haiku : 8 800 × 0.003 × 1.5 = 39.6 ml · Sonnet : 8 800 × 0.006 × 1.5 = 79.2 ml

```
---
💧 **Empreinte hydrique — 🔴 ~185 ml** · comparable à 30s de douche
[██████████]
🌱 ~11.9 g CO₂ · Haiku ~39.6 ml · Sonnet ~79.2 ml · **Opus ~185 ml**
> ~8 000 input + ~800 output = ~8 800 tokens · Opus ×0.014 · contexte long ×1.5 · ⚠️ ±50% eau / ±60% CO₂ · [Li et al. 2023](https://arxiv.org/abs/2304.03271)
```

---

## Règles comportementales

### TOUJOURS faire

- Ajouter le bloc hydrique **à chaque réponse** sans exception quand le skill est actif
- Utiliser **uniquement du Markdown pur** — aucune balise HTML
- Afficher la barre ASCII adaptée au niveau calculé
- Afficher **systématiquement** les trois modèles côte à côte (Haiku / Sonnet / Opus)
- Afficher la ligne CO₂ à chaque bloc
- Arrondir à 1 décimale (ex: 3.7 ml, pas 3.68421 ml)
- Mettre en gras le modèle actuellement utilisé dans la ligne de comparaison

### JAMAIS faire

- Interrompre ou modifier le contenu principal de la réponse
- Afficher un chiffre sans son contexte (niveau + comparaison)
- Prétendre que le calcul est exact — toujours mentionner l'incertitude
- Omettre le bloc sur les réponses très courtes (même 1-2 lignes méritent un bloc)
- Utiliser des balises `<details>`, `<summary>` ou tout autre HTML

---

## Compteur de session et résumé hebdomadaire

### Suivi continu

Mémoriser mentalement le cumul des échanges dans la conversation :
- Incrémenter le compteur à chaque réponse
- Accumuler le total d'eau estimé (somme des `eau_ml` de chaque échange)

### Résumé hebdomadaire (déclenché à partir de 10 échanges)

Dès que la conversation atteint **10 échanges**, et tous les 5 échanges ensuite
(15, 20, 25…), afficher un bilan de session **après** le bloc standard :

```
---
📊 **Bilan de session — [N] échanges**
💧 Total eau : ~[TOTAL_ML] ml · Moyenne : ~[MOY] ml/échange
🌱 Total CO₂ estimé : ~[TOTAL_CO2] g
→ Équivalent à [comparaison globale ex: "remplir un verre d'eau" / "une bouteille de 50cl"]
💡 Modèle le plus économe sur cette session : Haiku aurait utilisé ~[HAIKU_TOTAL] ml au total.
```

**Comparaisons globales de session :**

| Total eau session | Équivalent        |
|-------------------|-------------------|
| < 20 ml           | Quelques gorgées  |
| 20 – 100 ml       | Un grand verre    |
| 100 – 500 ml      | Une bouteille     |
| 500 – 1 500 ml    | Une carafe        |
| > 1 500 ml        | Un seau d'eau     |

---

## Transparence épistémique

Ce skill doit **toujours** rappeler que :
- Les chiffres sont des **estimations de modèle**, pas des mesures réelles
- Anthropic ne publie pas ses données WUE/PUE internes
- L'incertitude réelle peut varier de ×0.5 à ×2.5 selon la région du datacenter,
  la saison, et la charge serveur au moment de l'inférence
- La référence principale (Li et al., 2023) porte sur GPT-4, les facteurs Claude
  sont extrapolés proportionnellement

---

## Références scientifiques

- **Li et al. (2023)** — *"Making AI Less Thirsty"* · [arxiv.org/abs/2304.03271](https://arxiv.org/abs/2304.03271)
- **Luccioni et al. (2023)** — *"Power Hungry Processing"* · [arxiv.org/abs/2311.16863](https://arxiv.org/abs/2311.16863)

Pour approfondir, voir `references/sources.md` qui contient :
- Extraits clés des publications scientifiques
- Données WUE/PUE des grands fournisseurs cloud
- Comparatif d'empreinte par type de tâche (génération de code vs résumé vs chat)
