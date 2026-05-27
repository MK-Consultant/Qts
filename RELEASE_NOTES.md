# Notes de version

Ce dépôt ne dispose pas encore de numérotation d’application formelle dans les fichiers. Les livraisons sont désignées ici par **version livrable** ; la **version du référentiel** reste celle indiquée dans `referentiel.json` et le pied de page.

---

## 1.0.3 — 2026-05-21

### Contenu et données

- Référentiel régénéré depuis **`questionnaire_v3.14.xlsx`** (version référentiel **3.1.4**).
- Import : fusion par défaut **`--merge title`** (ordre Excel respecté ; rattachement `id`/criticité par titre après normalisation). Option **`--merge positional`** pour l’ancien mode par rang.
- Métadonnées **`navigation.json`** alignées sur la version du référentiel ; bloc embarqué dans **`index.html`** mis à jour.

---

## 1.0.2 — 2026-05-06

### Contenu et données

- Référentiel régénéré depuis **`Questionnaire IA v312.xlsx`** (version référentiel **3.1.2**).
- Script d’import documenté : `scripts/xlsx_to_referentiel.py` (nécessite **openpyxl**).

### Interface et PDF

- Contours des blocs de réponse A/B plus visibles ; premier paragraphe des réponses et des mesures en **gras** selon les sauts de paragraphe du tableur (double saut de ligne).
- Distinction explicite entre **En savoir plus** (colonne Contexte) et **Référence juridique** (lien) dans le bilan HTML, le volet « Je ne sais pas » et l’export PDF ; détail des mesures en section séparée « Détail des mesures ».
- L’**étape** du questionnaire est affichée sous le titre de la question (et sous le titre dans le PDF du plan d’action) pour éviter la confusion avec un titre long.

### Fichiers de configuration

- Alignement des métadonnées `navigation.json` sur la version **3.1.2** du référentiel.

---

## 1.0.1 — 2026-05-05

Version livrable axée sur la sécurité frontale, la cohérence des dépendances et la documentation de conformité.

### Sécurité

- Ajout de l’attribut **Subresource Integrity (SRI)** sur les feuilles de style DSFR (`dsfr.min.css`, `icons.min.css`) servies depuis jsDelivr.
- Correction du chargement **jsPDF** : passage à une URL CDNjs **fonctionnelle** (`2.5.1` au lieu d’un fichier `2.5.2` inexistant sur CDNjs).
- Ajout du **SRI** sur le script jsPDF.
- Suppression du **fallback dynamique** vers `unpkg.com`, qui chargeait une dépendance sans contrôle d’intégrité depuis une deuxième origine.
- Durcissement partiel de la **Content-Security-Policy** : directives `base-uri 'none'` et `object-src 'none'` ; retrait de `https://unpkg.com` de `script-src`.

### Interface et formulation

- Ajustement des mentions du **pied de page** pour refléter l’état réel du service : formulation plus prudente sur l’accessibilité et les données, mention du **SRI** aux côtés de la CSP.

### Documentation

- Ajout de `SECURITY.md` : périmètre technique, mesures présentes dans le front et chantiers NIS2 hors code.
- Ajout de `DECLARATION-RGESN.md` : première déclaration d’écoconception et liste d’actions restantes.
- Ajout du présent fichier `RELEASE_NOTES.md` et de `README.md`.

### Référentiel

- Aucun changement de contenu réglementaire dans cette livraison : référentiel inchangé **v3.1.1** (date `2026-05-01` dans les données).

### Limitations connues (inchangées ou à poursuivre)

- La CSP autorise encore `'unsafe-inline'` pour les styles et scripts contenus dans `index.html`.
- Les ressources DSFR et jsPDF restent chargées depuis des **CDN** : mitigation renforcée par le SRI, mais pas encore **self-hosting**.
- Le référentiel complet peut être dupliqué entre `referentiel.json` et le bloc embarqué dans `index.html`, ce qui alourdit la page initiale.

---

## Historique synthétique (avant 1.0.1)

Les modifications antérieures ne sont pas tracées dans ce dépôt. La base fonctionnelle existante avant cette livraison comportait notamment :

- Application statique `index.html` avec questionnaire guidé, bilan et export PDF.
- Données `referentiel.json`, `navigation.json`, `config.json`.
- Schéma DSFR visuel, en-têtes de sécurité HTTP équivalents (meta), gestion d’accessibilité partielle (skiplinks, rôles ARIA, focus).
