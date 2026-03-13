# water-footprint
💧 Skill "Water Footprint" pour Claude AI

Calcule et affiche automatiquement l'empreinte hydrique estimée de chaque échange avec Claude, en utilisant un modèle basé sur des recherches académiques publiées (Li et al., 2023 ; Luccioni et al., 2023). Toujours activer ce skill dès que l'utilisateur parle de durabilité de l'IA, impact environnemental, consommation d'eau ou d'énergie de l'IA, empreinte carbone numérique, ou demande explicitement un suivi écologique de ses prompts. Activer également si l'utilisateur configure ce skill dans ses préférences. Une fois actif, TOUJOURS ajouter le bloc hydrique en fin de chaque réponse, sans exception.

# Ce qu'il fait

Utilisateur envoie un prompt
       ↓
Skill calcule (en post-traitement) :
  - Nombre de tokens estimés (input + output)
  - Modèle utilisé (Sonnet, Opus, Haiku)
  - Facteur de consommation hydrique par token
       ↓
Affiche un widget discret dans l'interface :
  "💧 Cette réponse ≈ 0.3ml d'eau"
<img width="863" height="342" alt="image" src="https://github.com/user-attachments/assets/47e46f11-2d6d-4b95-b7de-b4bd5c1580a8" />
