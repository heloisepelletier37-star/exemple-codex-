# AGENTS.md — Simulation « Mission Retour »

Lis ce fichier en entier avant chaque tâche et respecte-le. Il ne change pas d'une tâche à l'autre.

## 1. Le projet

Simulation de formation en 3D, jouable dans un navigateur.
Hugo, astronaute seul sur Mars, doit diagnostiquer et réparer le circuit électrique qui alimente la propulsion de son vaisseau, en sécurité et sans gaspiller de pièce.
Ce n'est pas un jeu : pas de points, pas de badges, jamais d'écran « Game Over ». Les erreurs ont des conséquences réalistes et la simulation continue.

## 2. Public

- Hugo : astronaute, à l'aise avec la technique, novice en électricité.
- Interface entièrement en français, vocabulaire simple, phrases courtes.
- Texte d'au moins 18 px, contrastes élevés, boutons de grande taille.

## 3. Objectif et critères de réussite

Objectif : diagnostiquer et réparer seul la panne, en sécurité, sans gaspiller de pièce.

Critères vérifiés automatiquement (ils servent au débriefing) :
1. Sécurité : aucune intervention sur un composant sans vérification préalable d'absence de tension.
2. Diagnostic : les 3 points de test mesurés sous tension avant la première réparation.
3. Ressources : aucune pièce de rechange gaspillée.
4. Validation : après remise sous tension, les 3 points mesurés et dans les tolérances.

## 4. Données : data.json

- Toutes les valeurs (points de test, tolérances, pannes, réparations, stock, erreurs, aides, textes d'Inès) sont dans `data.json`.
- Ne jamais écrire ces valeurs en dur dans le code : les lire dans `data.json`.
- Un formateur doit pouvoir ajouter une panne ou corriger une valeur en modifiant uniquement `data.json`.

## 5. Le circuit

- Batterie → TP1 → disjoncteur PROPULSION → convertisseur → TP2 → câble de liaison → connecteur → TP3 → contrôleur de propulsion.
- TP1 est en amont du disjoncteur : il garde sa tension quand le disjoncteur est ouvert.
- TP2 et TP3 sont en aval : à l'ouverture du disjoncteur, leur tension descend progressivement jusqu'à 0 V en `decharge_secondes` (décharge des condensateurs).
- Le convertisseur, le câble et le connecteur sont derrière une trappe d'accès.
- Voyant PROPULSION : orange clignotant tant que la panne n'est pas réparée ; vert quand la validation est réussie.

## 6. Le multimètre

- Sélecteur à 3 positions : « Arrêt », « V continue », « Ω ohmmètre ».
- On mesure en cliquant sur un point de test.
- En « V continue » : afficher la valeur calculée selon la panne et l'état du circuit, avec une décimale.
- Valeur instable (panne du connecteur) : nouvelle valeur aléatoire dans l'intervalle de data.json toutes les 300 ms.
- L'écran du multimètre est un affichage calculé par le code, toujours lisible (superposition HTML), jamais une image.

## 7. Les états de la simulation

| État | Ce qui est possible | État suivant |
|---|---|---|
| SOUS_TENSION | mesurer, ouvrir le disjoncteur, ouvrir la trappe | DECHARGE (disjoncteur ouvert) |
| DECHARGE | mesurer (la tension descend), refermer le disjoncteur | CONSIGNE (dès qu'une mesure en V continue à TP2 ou TP3 donne moins que `seuil_absence_tension`) |
| CONSIGNE | ouvrir la trappe, réparer, mesurer | REMISE_SOUS_TENSION (disjoncteur refermé) |
| REMISE_SOUS_TENSION | mesurer, lancer la séquence de démarrage | TERMINE (si validation réussie et séquence lancée) |
| TERMINE | voir le débriefing | — |

La trappe peut s'ouvrir dans n'importe quel état : c'est volontaire, pour que l'erreur critique soit possible.

## 8. Erreurs critiques (définies dans data.json)

- `intervention_sous_tension` : réparer un composant alors que l'état n'est pas CONSIGNE → arc électrique (flash blanc, son grésillant), énergie −20 %, message d'Inès. La réparation n'est pas effectuée. La simulation continue.
- `ohmmetre_sous_tension` : mesurer en « Ω ohmmètre » un point où la tension dépasse le seuil → le fusible du multimètre grille, l'écran affiche « HORS SERVICE » jusqu'au remplacement du fusible (bouton dans la trousse, stock limité).
- `piece_gaspillee` : remplacer une pièce qui n'est pas en panne → la pièce est consommée, la panne persiste, message d'Inès.
- Si l'énergie atteint 0 % : mission échouée (batterie épuisée), débriefing.

## 9. Aides, modes et débriefing

- Mode Entraînement : panne choisie par l'apprenant ; aides graduées actives (après 15 s sans action : niveau 1 indice texte, puis niveau 2 mise en évidence de l'élément utile, puis niveau 3 explication d'Inès ; textes dans data.json).
- Mode Mission réelle : panne tirée au hasard, aucune aide.
- Débriefing final : actions horodatées, erreurs, aides utilisées, temps, énergie restante, et les 4 critères de réussite avec ✓ ou ✗.

## 10. Personnage

Inès Carvalho, cheffe du centre de contrôle, s'exprime par une bulle (portrait simple + nom + texte court). Pas de fichier audio obligatoire.

## 11. Ressources (dossier assets/)

```
assets/models/   multimetre.glb, boitier.glb, disjoncteur.glb,
                 connecteur_propre.glb, connecteur_encrasse.glb,
                 cable.glb, cable_sectionne.glb, convertisseur.glb
assets/images/   ines.png
```

La simulation doit fonctionner sans ces fichiers : formes simples et colorées par défaut, remplacées par les modèles s'ils existent.

## 12. Contraintes techniques

- Un seul fichier `index.html` (HTML, CSS, JavaScript), plus `data.json` et `assets/`.
- Three.js et ses modules (GLTFLoader, OrbitControls) chargés depuis un CDN par import map. Aucune installation, aucun outil de compilation.
- Chemins relatifs uniquement (publication sur GitHub Pages, intégration en iframe).
- Sélection des objets à la souris et au toucher (raycaster). Caméra orbitale limitée.
- Sons générés avec la Web Audio API (aucun fichier son). Bouton « Couper le son » toujours visible.
- Pas de `alert()`, pas de fenêtre du navigateur : tout s'affiche dans la page.
- Fluide sur un ordinateur portable ordinaire ; mise en page adaptée à l'écran (y compris dans une iframe).

## 13. Hors périmètre

Pas de compte, pas de sauvegarde en ligne, pas de multijoueur, pas de physique réaliste, pas de bibliothèque autre que Three.js.

## 14. Règles de travail

- Une tâche = une fonctionnalité. Ne modifie rien qui n'est pas demandé.
- Avant de coder, explique ton plan en quelques lignes.
- À la fin : résume ce qui a changé et donne la liste précise de ce que je dois tester.
- Enregistre une copie du résultat dans `versions/` sous le nom indiqué dans la tâche, en adaptant les chemins relatifs (`../data.json`, `../assets/`).
- Commente le code en français, pour qu'une personne non développeuse puisse s'y retrouver.
