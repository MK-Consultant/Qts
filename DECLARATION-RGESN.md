# Declaration d'ecoconception RGESN

## Perimetre

Service audite : questionnaire statique de conformite IA.

Le service fonctionne cote navigateur, sans backend applicatif visible dans ce depot. Les reponses sont conservees en memoire le temps de la session et l'export PDF est genere localement dans le navigateur.

## Objectif du service

Le service vise a aider les porteurs de projets IA a realiser une premiere auto-evaluation declarative sur les volets IA Act, RGPD, AFNOR et RGESN.

Cette auto-evaluation ne remplace pas un audit juridique, un audit RGAA, une analyse d'impact ou une qualification formelle de conformite.

## Mesures deja en place

- Application statique, limitant les traitements serveur.
- Absence de cookies applicatifs detectes dans le code.
- Absence de stockage persistant detecte dans le code (`localStorage` et `sessionStorage` non utilises).
- Chargement de donnees locales (`referentiel.json`, `navigation.json`, `config.json`) avec repli embarque.
- Parcours limite a une finalite principale : produire un bilan declaratif et un plan d'action.

## Ecarts identifies

- Le referentiel est duplique entre `referentiel.json` et le bloc embarque dans `index.html`, ce qui augmente le poids initial.
- Les dependances DSFR et jsPDF sont encore chargees depuis des CDN, meme si elles sont maintenant protegees par SRI.
- Aucune mesure automatisee du poids de page, du nombre de requetes, du temps de chargement ou de l'impact environnemental n'est fournie.
- Aucune declaration publique complete RGESN n'est encore reliee depuis l'interface.

## Actions a realiser

1. Choisir une source de donnees principale et supprimer la duplication embarquee si le mode hors ligne strict n'est pas requis.
2. Servir les dependances tierces localement ou les integrer dans un processus de build versionne.
3. Mesurer regulierement le poids total charge, le nombre de requetes et le comportement sur terminal ancien.
4. Documenter les criteres RGESN applicables, non applicables et non conformes avec justification.
5. Ajouter une revue d'utilite du service et des alternatives non numeriques ou plus sobres.
6. Definir une politique de maintenance du referentiel et de suppression des fonctionnalites non utilisees.

## Indicateurs a suivre

- Poids total de la page et des ressources chargees.
- Nombre de requetes reseau au chargement initial.
- Nombre de dependances tierces.
- Frequence de mise a jour du referentiel.
- Resultat d'un test clavier et lecteur d'ecran sur le parcours principal.