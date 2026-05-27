# Documentation technique — mise en production

Ce document décrit **l’architecture**, les **mécanismes de sécurité côté client** et le **pipeline d’export** de l’application « Questionnaire conformité IA ». Il complète les commentaires retirés du fichier `index.html` pour permettre une **revue sécurité** et une **passation exploitation** sans parcourir le code ligne à ligne.

---

## 1. Architecture

### 1.1 Nature du livrable

- Application **entièrement statique** : pas de backend, pas d’API applicative dans ce dépôt.
- Une seule « page SPA légère » : plusieurs `<section class="screen">` masquées via CSS (`display: none`) sauf `.screen.active`.
- Données métier chargées par `fetch()` vers `referentiel.json`, `navigation.json`, `config.json`, avec **repli automatique** sur des constantes embarquées (`EMBEDDED_*`) dans `index.html` lorsque les fichiers absents ou l’usage en `file://` l’exigent.

### 1.2 Persistance locale

Les réponses et l’état de session sont conservés **en mémoire JavaScript uniquement** (objet `state`). Aucun `localStorage` / `IndexedDB` n’est utilisé : fermer l’onglet efface les réponses sauf comportement navigateur hors périmètre.

### 1.3 En-têtes et composition

En-tête de site : titre fixe ; lien d’accueil pointant vers `index.html`. Étapes macro : composant monté sous `#flow-stepper-mount` avec le Design System de l’État (DSFR) chargé en module ES depuis jsDelivr.

### 1.4 Outils développement

Le script [`tools/strip_inline_comments.py`](./tools/strip_inline_comments.py) peut être réexécuté pour régénérer une version sans commentaires (HTML `<!-- -->`, CSS `/* */`, commentaires JS) **sans casser les URL** contenues dans les littéraux chaîne ni les expressions rationnelles. À utiliser après éditions manuelles importantes uniquement si la politique « sans commentaires » doit être reappliquée.

---

## 2. Sécurité applicative (navigateur)

### 2.1 Content Security Policy (CSP)

Définie dans une balise `<meta http-equiv="Content-Security-Policy">` :

| Directive      | Valeur indicative |
|----------------|-------------------|
| `default-src`  | `'self'` |
| `script-src`   | `'self' 'unsafe-inline' https://cdn.jsdelivr.net` |
| `style-src`    | `'self' 'unsafe-inline' https://cdn.jsdelivr.net` |
| `font-src`     | `https://cdn.jsdelivr.net` |
| `img-src`      | `'self' data: blob:` |
| `connect-src`  | `'self'` |
| `object-src`   | `'none'` |
| `base-uri`     | `'none'` |

**Point de revue** : présence de `'unsafe-inline'` pour scripts et styles est **volontaire** (mono-fichier, pas de pipeline de build). Pour durcir en production : envisager désaccord nonces/hashés via build (Hugo, Eleventy, etc.) et hébergement local des JS/CSS DSFR.

### 2.2 Autres en-têtes équivalents meta

- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: no-referrer`
- `Permissions-Policy` limitant caméra, micro, géolocalisation, paiement

Pour un hébergement HTTP réel, **dupliquer** ces politiques en en-têtes HTTP serveur (prioritaires sur les meta).

### 2.3 Ressources externes et SRI

- DSFR et icônes : jsDelivr, avec **intégrité SRI** sur les feuilles de style.
- jsPDF : bundle **local** `vendor/jspdf.umd.min.js` (v2.5.1, aligné sur le paquet npm / cdnjs) — plus de chargement réseau pour l’export PDF.
- Module DSFR (`type="module"`) : jsDelivr **sans SRI** dans la version actuelle — **piste d’amélioration** pour la prod (self-host + SRI ou politique d’approvisionnement documentée).

### 2.4 Sanitisation des contenus injectés

| Contexte | Mécanisme |
|----------|-----------|
| Texte utilisateur (« Je ne sais pas ») | `sanitizeFreeText` + `escapeHtml` / `nlToBrEscaped` selon le cas |
| Référentiel (champs texte, listes) | `prepareReferentielPlain` → `safeHtml` (balises `<b>` uniquement, équilibrées) → `formatCellHtml` pour le HTML affiché |
| PDF | `clean()` pour texte plat ; `pdfRichRaw` + `wRich` pour le gras ; `sanitizeFreeText` sur les réponses libres exportées |

**Hypothèse** : le référentiel JSON est une **donnée de confiance** (fournie par l’éditeur de l’outil). Une modification hostile du JSON pourrait contourner partiellement les garde-fous — la CSP limite l’exécution de script injecté.

### 2.5 Liens externes

- Formulaire satisfaction (Galileo) : URL centralisée en variable `QC_FEEDBACK_SURVEY_URL` dans le script ; le lien HTML du bilan duplique la même URL pour rester utilisable sans JavaScript active.

---

## 3. Accessibilité et impression

### 3.1 Réduction des animations

Les transitions d’activation d’écran respectent `prefers-reduced-motion` : animations désactivées pour les utilisateurs qui le demandent au niveau système.

### 3.2 Feuille « print »

En impression (Ctrl+P) :

- Masqués : en-tête, pied de page, skiplinks, toast, `.no-print`, écrans inactifs.
- Visible : uniquement `.screen.active` (un seul écran à la fois).

---

## 4. Export PDF (jsPDF)

### 4.1 Contenu et structure

- **Pages principales** : corps aligné sur le bilan à l’écran — titre « Bilan de conformité », sections **Synthèse et rapport** (deux colonnes : jauge circulaire, libellé `bilanLibelleConformite(pct)` ; carte **Informations projet**, séparateur, **Résultats** en trois tuiles chiffrées), **Points de conformité**, **Plan d’action**, puis détail des actions avec **`wRich`**. Les **titres de section** sont précédés d’un trait séparateur avec espacement réservé au titre ; les **corps de texte** utilisent une largeur maximale égale aux marges (pas de débordement) **et justification** sauf **dernière ligne** et titres centrés de la synthèse. La **conformité par volet** n’est pas dans l’export PDF.
- **Annexe finale** : ancienne « page de garde » : titre application, avertissement, limites de responsabilité, bloc secrets CRPA, contribution (`QC_FEEDBACK_SURVEY_URL`).

### 4.2 Horodatage pied de page

`Généré le …` utilise `toLocaleString('fr-FR')` pour limiter les collisions de fichiers téléchargés le même jour.

### 4.3 Nom du fichier

`bilan-{slug projet}.pdf` avec repli **`evaluation`** si le slug du nom de projet est vide.

---

## 5. Liste de contrôle hébergement (hors dépôt)

- [ ] Servir uniquement en **HTTPS** avec certificats valides et configuration TLS à jour  
- [ ] Rejouer **CSP** et politiques équivalentes en **en-têtes HTTP**  
- [ ] Journalisation / supervision des erreurs HTTP (CDN, origin) selon vos exigences NIS2 / internes  
- [ ] Processus de **mise à jour** DSFR et jsPDF (voir aussi [SECURITY.md](./SECURITY.md))  
- [ ] Décision sur **auto-hébergement** des assets CDN pour réduire la dépendance réseau et permettre **SRI** systématique  

---

## 6. Fichiers de données versionnés

- `referentiel.json`, `navigation.json`, `config.json` sont validés au chargement par des fonctions de validation structurelle (règles de navigation résolues, questions référencées, etc.).

---

Pour le volet conformité réglementaire NIS2 et organisationnel, se reporter à **[SECURITY.md](./SECURITY.md)** et aux procédures internes du service hébergeur.
