# Questionnaire conformité IA

Application web **statique** d’auto-évaluation pour les projets d’intelligence artificielle : périmètres **IA Act**, **RGPD**, **AFNOR / RGESN** et volet transverse. Elle propose un assistant pas à pas, un bilan synthétique et un **export PDF** généré dans le navigateur.

Ce dépôt ne contient pas de serveur applicatif ni de base de données. Les réponses restent **en mémoire dans le navigateur** pendant la session ; aucune transmission à un backend n’est implémentée dans ce code.

**Versions utiles à distinguer**

- **Référentiel** : version affichée en pied de page, définie dans `referentiel.json` (champs `version` et `date`).
- **Livrable application** : voir [RELEASE_NOTES.md](./RELEASE_NOTES.md).

---

## Contenu du dépôt

| Fichier | Rôle |
|--------|------|
| [index.html](./index.html) | Interface complète (HTML, CSS intégré, logique JavaScript). |
| [referentiel.json](./referentiel.json) | Liste des questions, réponses A/B, contextes, mesures, criticités, applicabilité par phase. |
| [navigation.json](./navigation.json) | Règles de navigation (`next`, `goto`, `skip_section`, `stop`, etc.). |
| [config.json](./config.json) | Libellés, phases, pondération des criticités, paramètres d’interface. |
| [RELEASE_NOTES.md](./RELEASE_NOTES.md) | Notes de version. |
| [SECURITY.md](./SECURITY.md) | Pistes sécurité et compléments NIS2 hors code. |
| [DOCUMENTATION_PRODUCTION.md](./DOCUMENTATION_PRODUCTION.md) | Architecture, sécurité applicative, export PDF et impression pour revue mise en production. |

---

## Prérequis

- Un navigateur récent (Chrome, Firefox, Safari ou Edge).
- Pour charger les JSON depuis `fetch()`, ouvrir le site via **`http://` ou `https://`** (et non uniquement `file://`), sauf si vous utilisez exclusivement le référentiel embarqué dans `index.html`.

---

## Installation et exécution locale

Aucune installation Node ou Composer n’est requise.

### Option A — Serveur HTTP minimal

À la racine du dossier du projet :

```bash
python3 -m http.server 8080
```

Puis ouvrir `http://localhost:8080/` dans le navigateur.

### Option B — Hébergement statique

Copier l’ensemble des fichiers (`index.html`, `*.json`) sur tout hébergement ne servant que des fichiers statiques (objet de stockage + CDN, GitHub Pages, etc.). Veiller à ce que les quatre fichiers ci-dessus restent au **même niveau de répertoire** que `index.html`.

---

## Utilisation (utilisateur final)

1. **Accueil** : cocher les prérequis, saisir le nom du projet, choisir une phase du cycle de vie, puis lancer l’évaluation.
2. **Questionnaire** : répondre par bloc A, B ou « Je ne sais pas » ; suivre les messages de navigation ou d’arrêt selon `navigation.json`.
3. **Bilan** : consulter le score, le détail par volet, les points conformes et le plan d’action.
4. **Export PDF** : document **texte sélectionnable** (jsPDF / Helvetica). Le **bilan** commence le document (synthèse, points conformes, plan d’action comme à l’écran) ; les **mentions légales / CRPA / contribution** sont en **dernière section**. Texte cadré à la largeur utile (césure) et **justifié** sur les paragraphes (dernière ligne alignée à gauche quand le moteur fournit plusieurs lignes). Rappels encadrés dans le plan d’action ; gras `<b>` via **`wRich`**. jsPDF **local** (`vendor/jspdf.umd.min.js` v2.5.1).

Les limitations légales et la valeur indicative des résultats sont rappelées en **fin de PDF** et dans l’interface. Voir [DOCUMENTATION_PRODUCTION.md](./DOCUMENTATION_PRODUCTION.md) pour l’audit d’architecture et les points d’hébergement.

---

## Configuration (`config.json`)

Principaux blocs :

- `application` : titre, sous-titre, organisme, mentions.
- `interface.boutons` et `interface.labels` : libellés réutilisables par la logique (certaines clés peuvent ne pas être toutes exploitées dans `index.html`).
- `interface.phases` et `interface.messages` : phases et messages génériques.
- `volets` : couleurs et descriptions par volet.
- `ponderation` : poids et couleurs par niveau de criticité.
- `resultatsApiUrl` : chaîne vide dans cette version ; aucun envoi automatique des résultats.

Après modification de `config.json`, recharger la page ; aucune étape de compilation n’est nécessaire.

---

## Référentiel (`referentiel.json`)

Chaque entrée du tableau `questions` doit au minimum inclure les champs contrôlés par la validation JavaScript : `id`, `volet`, titre ou question, `mesures.resume`, et la structure attendue pour les phases (`applicable`).

### Import depuis Excel (`questionnaire_v3.14.xlsx` ou équivalent)

Le classeur source attendu comporte une feuille nommée **`Questionnaire simplifié`**, avec les colonnes : Volet, Titre / Sujet, Réponses A et B, Contexte, Mesures (réponse B), Info (détail), Étapes, Lien, puis les colonnes par phase (OUI/non).

Pour régénérer `referentiel.json`, le script lit les cellules telles qu’elles sont stockées en OOXML (texte enrichi inclus, sans troncature du contenu brut). Les **identifiants** et la **criticité** viennent du JSON actuel. Par défaut (`--merge title`), chaque ligne Excel est reliée à la question ayant **le même titre** (sans balises, après normalisation légère), ce qui conserve **l’ordre du classeur**. L’association stricte par rang (`--merge positional`) reste disponible comme option.

```bash
python3 scripts/xlsx_to_referentiel.py \
  --xlsx "questionnaire_v3.14.xlsx" \
  --referentiel referentiel.json \
  --out referentiel.json \
  --version "3.1.4" \
  --date "2026-05-21"
```

La dépendance `openpyxl` (`pip install openpyxl`) est optionnelle pour le script ; après import, resynchroniser la copie embarquée dans `index.html` (constantes `EMBEDDED_REFERENTIEL` / `EMBEDDED_NAVIGATION`) comme pour cette livraison.

Pour une mise à jour **manuelle** du contenu réglementaire :

1. Éditer `referentiel.json` (et ajuster la `version` / la `date`).
2. Vérifier la cohérence des identifiants avec `navigation.json`.
3. Si vous maintenez une copie embarquée dans `index.html`, alignez-la sur le même contenu ou supprimez-la pour éviter la duplication (voir [DECLARATION-RGESN.md](./DECLARATION-RGESN.md)).

---

## Navigation (`navigation.json`)

Le tableau `regles` associe chaque `question_id` à des actions selon la réponse (`a`, `b`, `je_ne_sais_pas`, ou clés héritées `oui`/`non`). Les cibles `goto` et les sections ignorées doivent référencer des ids ou intitulés de volet présents dans le référentiel.

---

## Confidentialité et données

Le pied de page et la logique actuelle indiquent **aucune transmission serveur applicative** dans ce dépôt. Les tiers suivants peuvent toutefois être contactés lors de l’usage normal :

- **CDN** (jsDelivr) pour **DSFR** (CSS, module JS, polices) — charges réseau externes. **jsPDF** est en **fichier local** sous `vendor/`.
- Les liens externes (par exemple formulaire « Donner mon avis », références juridiques) ouvrent dans un nouvel onglet selon le balisage `target="_blank"` et `rel="noopener noreferrer"`.

Pour une analyse RGPD complète, croiser avec votre politique d’hébergement, vos journaux serveur et votre CMP si vous ajoutez un backend ou des traceurs.

---

## Sécurité

Voir [SECURITY.md](./SECURITY.md) pour la liste des en-têtes, du SRI et des mesures organisationnelles à documenter (dont les attendus de type NIS2 hors périmètre du seul fichier HTML).

À chaque changement de version des **fichiers DSFR** chargés depuis jsDelivr, **recalculer les hashes SRI** et mettre à jour les attributs `integrity` dans `index.html`. Pour **jsPDF** en local, vérifier que `vendor/jspdf.umd.min.js` correspond à une version attestée lors des mises à jour.

---

## Écoconception

Voir [DECLARATION-RGESN.md](./DECLARATION-RGESN.md). Points sensibles : duplication référentiel / page, dépendances externes, absence de mesures d’impact environnemental automatisées dans ce dépôt.

---

## Accessibilité et DSFR

L’interface reprend des composants et classes du [Système de design de l’État (DSFR)](https://www.systeme-de-design.gouv.fr/). Une revue RGAA complète reste à planifier ; les mentions du pied de page reflètent cet état.

---

## Support et évolution

- Suivi des livraisons : [RELEASE_NOTES.md](./RELEASE_NOTES.md).
- Pour signaler une vulnérabilité : renseigner la section « Contact sécurité » dans [SECURITY.md](./SECURITY.md).
