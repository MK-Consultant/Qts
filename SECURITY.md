# Politique de securite et points NIS2

Voir aussi la fiche **[DOCUMENTATION_PRODUCTION.md](./DOCUMENTATION_PRODUCTION.md)** (architecture, CSP, export PDF, impression) pour les revues combinées sécurité / exploitation.

## Perimetre technique visible

Ce depot contient une application statique executee dans le navigateur. Aucun backend, compte utilisateur, base de donnees, journal serveur ou mecanisme d'administration n'est present dans le code audite.

Les mesures ci-dessous ameliorent la posture de securite du service, mais ne suffisent pas a etablir seules une conformite NIS2. La directive NIS2 depend aussi de l'entite, de son secteur, de son systeme d'information, de son hebergement et de ses processus organisationnels.

## Mesures deja appliquees dans le front

- Politique CSP declaree dans `index.html`.
- `X-Content-Type-Options: nosniff` via meta-equivalent.
- `Referrer-Policy: no-referrer` via meta-equivalent.
- `Permissions-Policy` limitant camera, microphone, geolocalisation et paiement.
- Integrite SRI ajoutee aux ressources CDN DSFR et jsPDF.
- Suppression du fallback jsPDF dynamique vers un second CDN.
- Echappement HTML centralise pour les contenus injectes depuis le referentiel.

## Mesures NIS2 a documenter hors code

1. Analyse des risques cyber liee au service et a son hebergement.
2. Procedure de gestion des incidents, avec roles, delais, escalade et notification.
3. Continuite d'activite : sauvegardes, reprise, mode degrade et tests.
4. Securite de la chaine d'approvisionnement : inventaire des dependances, suivi des versions, politique de mise a jour.
5. Gestion des vulnerabilites : veille, priorisation, correction et verification.
6. Controle d'acces aux environnements de publication et depot source.
7. Journalisation et supervision des acces a l'hebergement si le service est publie.
8. Chiffrement des communications en production via HTTPS.
9. Authentification multifacteur pour les comptes d'administration, de depot et d'hebergement.
10. Formation minimale des personnes qui maintiennent ou publient le service.

## Recommandations de maintenance

- Verifier mensuellement les versions DSFR et jsPDF.
- Preferer des ressources servies localement pour limiter le risque CDN.
- Regenerer les hash SRI a chaque changement de version.
- Tester le parcours principal apres chaque mise a jour de dependance.
- Conserver une trace des versions publiees du referentiel et de leurs dates de validation.

## Contact securite

dntum.donnees.conformiteødgfip.finances.gouv.fr