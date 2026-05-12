# Quick Test IA — IALS

Formulaire d'évaluation du niveau en IA générative. 10 questions, score sur 10, message + CTA selon le niveau.

## Mise en ligne

1. Pousse `index.html` à la racine du repo.
2. Active GitHub Pages : `Settings → Pages → Source : main → /(root)`.
3. L'URL publique sera `https://animalcreatif.github.io/quicktestniveau/`.

## Configuration du webhook Make → Notion

Le formulaire envoie un POST JSON vers une URL Make. GitHub Pages étant un site statique, on ne peut pas appeler l'API Notion directement (clé exposée, CORS). Make fait le pont.

### Étape 1 — Crée le scénario Make

**Module 1 : Webhook (Custom webhook)**
- Récupère l'URL générée, exemple : `https://hook.eu1.make.com/abc123...`

**Module 2 : Notion → Create a Database Item**
- Base : `Leads Quiz IA`
- Mapping des champs (à créer dans Notion si absents) :

| Champ Notion        | Type       | Source webhook              |
|---------------------|------------|-----------------------------|
| Prénom              | Title      | `{{prenom}}`                |
| Email               | Email      | `{{email}}`                 |
| Profil              | Select     | `{{profil}}`                |
| Score               | Number     | `{{score}}` (sur 10)        |
| Score brut          | Number     | `{{score_brut}}` (sur 30)   |
| Niveau              | Select     | `{{niveau}}`                |
| Date                | Date       | `{{timestamp}}`             |
| Source              | Select     | `{{source}}`                |
| Détail réponses     | Text       | `{{reponses}}` (JSON brut)  |

Options Select à créer dans Notion :
- `Profil` : `creatif`, `non-creatif`
- `Niveau` : `Débutant`, `Intermédiaire`, `Expert`
- `Source` : `quicktestniveau`

### Étape 2 — Colle l'URL du webhook dans le fichier

Dans `index.html`, cherche `WEBHOOK_URL` et remplace :

```js
const WEBHOOK_URL = "https://hook.eu1.make.com/abc123...";
```

### Étape 3 — Active le scénario

Scénario sur `ON`. Teste avec une vraie soumission. Le lead doit apparaître dans Notion en quelques secondes.

## Payload envoyé

```json
{
  "timestamp": "2026-05-12T14:23:00.000Z",
  "source": "quicktestniveau",
  "prenom": "Jane",
  "email": "jane@example.com",
  "profil": "creatif",
  "score": 6,
  "score_max": 10,
  "score_brut": 18,
  "score_brut_max": 30,
  "niveau": "Intermédiaire",
  "reponses": {
    "Q1": { "label": "Cette semaine.", "pts": 2 },
    "Q2": { "label": "Une instruction structurée avec contexte, rôle, contraintes.", "pts": 2 }
  }
}
```

## Scoring

Score brut sur 30, converti sur 10 via `Math.floor((scoreBrut / 30) * 10)`.

Conséquence : pour décrocher 10/10, il faut littéralement 30/30 sur les questions, donc cocher uniquement les réponses à 3 pts. C'est volontaire.

Seuils :
- **0 à 4/10** → Débutant → CTA formations
- **5 à 9/10** → Intermédiaire → CTA formations
- **10/10** → Expert → CTA LinkedIn

## Personnalisation

Toutes les variables sont en haut du `<script>` dans `index.html` :
- `WEBHOOK_URL` : URL du webhook Make
- `URL_FORMATIONS` : page d'accueil IALS
- `URL_LINKEDIN` : profil LinkedIn Nathalie

Charte graphique en variables CSS dans le `:root` du `<style>` :
- `--noir`, `--noir-soft` : fond
- `--rose`, `--rose-dim` : accent (rose poudré)
- `--blanc-casse` : texte
