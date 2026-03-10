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

| Eau estimée   | Niveau        | Emoji | Comparaison parlante             |
|---------------|---------------|-------|----------------------------------|
| < 0.5 ml      | 🟢 Minime     | 💧    | Moins d'une goutte               |
| 0.5 – 2 ml    | 🟢 Faible     | 💧💧  | Une petite gorgée                |
| 2 – 8 ml      | 🟡 Modéré     | 💧💧💧| Une cuillère à café              |
| 8 – 25 ml     | 🟠 Notable    | 🫗    | Une shot de café                 |
| 25 – 100 ml   | 🔴 Élevé      | 🚿    | Un verre d'eau                   |
| > 100 ml      | 🔴 Très élevé | 🛁    | Comparable à 30s de douche       |

---

## Format d'affichage OBLIGATOIRE

Toujours placer ce bloc **en toute fin de réponse**, après le contenu principal,
séparé par une ligne `---`.

Ne JAMAIS utiliser de balises HTML (`<details>`, `<summary>`, etc.).
Utiliser uniquement du Markdown pur.

### Template standard

```
---
💧 **Empreinte hydrique estimée — [NIVEAU_EMOJI] ~[VALEUR] ml**

- Tokens estimés : ~[INPUT] input + ~[OUTPUT] output = ~[TOTAL] tokens
- Modèle : [NOM_MODÈLE] · Facteur : [FACTEUR] ml/token
- Contexte : [TAILLE_CONTEXTE] · Multiplicateur : ×[MULT]
- **Résultat : ~[VALEUR] ml** [COMPARAISON]

*Les datacenters consomment de l'eau pour refroidir leurs serveurs et produire l'électricité nécessaire au calcul.*
⚠️ *Estimation basée sur [Li et al. (2023)](https://arxiv.org/abs/2304.03271) · Incertitude : ±50%*
```

### Exemple concret — prompt court, Sonnet

Prompt : "Qu'est-ce que l'IA ?" (~60 mots → ~80 tokens input)
Réponse : ~150 mots → ~200 tokens output
Total : ~280 tokens · Sonnet (×0.006) · Contexte court (×1.0)
**Résultat : 280 × 0.006 × 1.0 = 1.68 ml**

```
---
💧 **Empreinte hydrique estimée — 🟢 ~1.7 ml**

- Tokens estimés : ~80 input + ~200 output = ~280 tokens
- Modèle : Claude Sonnet · Facteur : 0.006 ml/token
- Contexte : court · Multiplicateur : ×1.0
- **Résultat : ~1.7 ml** ≈ une petite gorgée d'eau 💧💧

*Les datacenters consomment de l'eau pour refroidir leurs serveurs et produire l'électricité nécessaire au calcul.*
⚠️ *Estimation basée sur [Li et al. (2023)](https://arxiv.org/abs/2304.03271) · Incertitude : ±50%*
```

### Exemple concret — long contexte, Opus

Prompt + historique : ~8 000 tokens input · Réponse : ~600 mots → ~800 tokens
Total : ~8 800 tokens · Opus (×0.014) · Contexte long (×1.5)
**Résultat : 8 800 × 0.014 × 1.5 = 184.8 ml**

```
---
💧 **Empreinte hydrique estimée — 🔴 ~185 ml**

- Tokens estimés : ~8 000 input + ~800 output = ~8 800 tokens
- Modèle : Claude Opus · Facteur : 0.014 ml/token
- Contexte : long (>10k tokens) · Multiplicateur : ×1.5
- **Résultat : ~185 ml** ≈ un grand verre d'eau 🛁

💡 *Astuce : utiliser Claude Sonnet sur cette tâche aurait estimé ~74 ml (-60%).*

*Les datacenters consomment de l'eau pour refroidir leurs serveurs et produire l'électricité nécessaire au calcul.*
⚠️ *Estimation basée sur [Li et al. (2023)](https://arxiv.org/abs/2304.03271) · Incertitude : ±50%*
```

---

## Règles comportementales

### TOUJOURS faire

- Ajouter le bloc hydrique **à chaque réponse** sans exception quand le skill est actif
- Utiliser uniquement du **Markdown pur** — aucune balise HTML
- Afficher la comparaison parlante correspondant au niveau calculé
- Arrondir à 1 décimale (ex: 3.7 ml, pas 3.68421 ml)
- Afficher l'astuce d'optimisation si le résultat dépasse 50 ml

### JAMAIS faire

- Utiliser des balises HTML comme `<details>`, `<summary>`, `<div>`, etc.
- Interrompre ou modifier le contenu principal de la réponse
- Afficher un chiffre sans son contexte (niveau + comparaison)
- Prétendre que le calcul est exact — toujours mentionner l'incertitude ±50%
- Omettre le bloc sur les réponses très courtes (même 1-2 lignes méritent un bloc)

### Astuce optimisation (si eau > 50 ml)

Calculer ce que coûterait la même requête sur Sonnet si Opus est utilisé,
ou sur Haiku si Sonnet est utilisé, et afficher la ligne :
> 💡 *Astuce : [modèle alternatif] aurait estimé ~[X] ml (-[Y]%).*

---

## Compteur de session (optionnel)

Si l'utilisateur demande un récapitulatif ou si la conversation dépasse 10 échanges,
proposer un bilan en mentionnant la somme cumulée estimée depuis le début de la session.

Format :
```
💧 **Bilan de session** : ~[TOTAL] ml consommés sur [N] échanges
  → Équivalent à [comparaison globale]
```

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

## Ressources complémentaires

Pour approfondir, voir `references/sources.md` qui contient :
- Extraits clés des publications scientifiques
- Données WUE/PUE des grands fournisseurs cloud
- Comparatif d'empreinte par type de tâche (génération de code vs résumé vs chat)
