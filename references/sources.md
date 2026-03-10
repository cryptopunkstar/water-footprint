# Sources & Références — Water Footprint Skill

## Publications scientifiques clés

### Li et al. (2023) — Référence principale
**"Making AI Less Thirsty: Uncovering and Addressing the Secret Water Footprint
of AI Models"**
- Auteurs : Pengfei Li, Jianyi Yang, Mohammad A. Islam, Shaolei Ren (UC Riverside)
- ArXiv : https://arxiv.org/abs/2304.03271
- Résultats clés :
  - GPT-3 (175B) : 700 ml d'eau pour ~10–50 questions
  - GPT-4 : estimé à 500 ml pour ~20–50 questions
  - L'eau est consommée pour le refroidissement direct ET la production d'énergie
  - Les datacenters en zones chaudes ou sèches consomment jusqu'à 3× plus

### Luccioni et al. (2023)
**"Power Hungry Processing: Watts Driving the Cost of AI Deployment?"**
- Auteurs : Alexandra Sasha Luccioni, Yacine Jernite, Emma Strubell
- Résultats clés :
  - Consommation énergie par tâche varie de 10× à 100× selon le modèle
  - Les tâches génératives consomment ~10× plus que les tâches de classification
  - Benchmarks sur BLOOM, OPT, T5, GPT-2

### Patterson et al. (2022) — Google
**"Carbon Footprint of Machine Learning Training"**
- Mesures internes Google/TPU
- PUE moyen Google datacenter : 1.10 (meilleur du secteur)
- WUE Google 2022 : 1.10 L/kWh

---

## Données WUE/PUE par fournisseur (2022-2023)

| Fournisseur | PUE moyen | WUE moyen (L/kWh) | Source          |
|-------------|-----------|-------------------|-----------------|
| Google      | 1.10      | 1.10              | Rapport annuel  |
| Microsoft   | 1.18      | 1.53              | Rapport annuel  |
| Amazon AWS  | 1.20      | ~1.80 (estimé)    | Rapport partiel |
| Meta        | 1.10      | ~1.40 (estimé)    | Rapport annuel  |
| Moyenne US  | 1.55      | 1.80              | Uptime Institute|

**Note** : Anthropic n'a pas publié ses métriques WUE/PUE à la date de rédaction.
Le skill utilise la moyenne industrie US comme approximation conservatrice.

---

## Comparatif d'empreinte par type de tâche

| Type de tâche               | Tokens typiques | Eau estimée (Sonnet) |
|-----------------------------|-----------------|----------------------|
| Question courte / factuelle | 150–400         | 0.9–2.4 ml           |
| Résumé d'un texte           | 800–2 000       | 4.8–12 ml            |
| Génération de code (~50 lignes) | 1 500–4 000 | 9–24 ml              |
| Rédaction longue (article)  | 3 000–8 000     | 18–48 ml             |
| Analyse de document PDF     | 5 000–15 000    | 30–90 ml             |
| Conversation multi-tour (10 échanges) | 8 000–20 000 | 48–120 ml   |

---

## Calibration des facteurs ml/token

### Méthodologie de dérivation

1. Li et al. (2023) mesure ~500 ml pour ~35 échanges GPT-4 moyens
2. Échange moyen GPT-4 ≈ 300 tokens input + 400 tokens output = 700 tokens/échange
3. 35 échanges × 700 tokens = 24 500 tokens total pour 500 ml
4. **Facteur GPT-4 brut** : 500 / 24 500 ≈ **0.020 ml/token**
5. GPT-4 est estimé ~1 700B paramètres (sparse MoE)
6. Claude Opus (~100B dense) est ~17× plus petit → facteur ÷ 1.4 (économies d'échelle) ≈ **0.014**
7. Claude Sonnet (~35B) : facteur × 0.43 de Opus ≈ **0.006**
8. Claude Haiku (~7B) : facteur × 0.22 de Opus ≈ **0.003**

> Ces dérivations sont approximatives. Les tailles de modèles Claude ne sont
> pas officiellement publiées ; elles sont estimées par la communauté.

---

## Limites connues du modèle

1. **Taille des modèles non confirmée** : Anthropic ne publie pas les tailles exactes
2. **Infrastructure inconnue** : localisation des datacenters Anthropic non publique
3. **Variabilité saisonnière** : WUE peut doubler en été (chaleur = plus de refroidissement)
4. **Cache KV** : les prompts avec cache consomment moins ; non modélisé
5. **Batching** : l'inférence en batch réduit la consommation par token ; non modélisé
6. **Eau grise** : la fabrication des serveurs (eau grise) n'est pas incluse

---

## Liens utiles

- https://arxiv.org/abs/2304.03271 — Li et al. 2023 (référence principale)
- https://arxiv.org/abs/2311.16863 — Luccioni et al. 2023
- https://sustainability.google/reports/ — Rapports Google
- https://aka.ms/AIsustainability — Microsoft AI sustainability
- https://uptimeinstitute.com/resources/research-and-reports — PUE industrie
