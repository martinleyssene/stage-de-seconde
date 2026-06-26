# CLAUDE.md

Ce fichier fournit des indications à Claude Code (claude.ai/code) pour travailler sur le code de ce dépôt.

## De quoi il s'agit

« Mon deuxième cerveau » — une application web de prise de notes / carte mentale tenant dans un seul fichier, réalisée pendant un *stage de seconde*. C'est un projet pédagogique : HTML + CSS + JavaScript vivent tous dans un seul fichier, **abondamment commenté en français** pour un public débutant. Il n'y a aucune étape de build, aucun serveur, aucune dépendance, aucun gestionnaire de paquets, aucun test.

## Comment le lancer

Ouvre `Mon deuxième cerveau.html` directement dans un navigateur (double-clic, ou `open "Mon deuxième cerveau.html"` sur macOS). C'est tout le déroulé — il n'y a rien à installer ni à compiler.

`index.html` n'est qu'une page d'accueil GitHub Pages qui redirige immédiatement vers le fichier principal. L'application est déployée à l'adresse `https://martinleyssene.github.io/mes-notes/` (la constante `LIEN_PARTAGE`).

## Travailler dans ce code

- **Tout est dans `Mon deuxième cerveau.html`** (~2800 lignes) : le `<style>` dans le `<head>`, le balisage dans le `<body>`, et un gros `<script>` tout en bas. Toutes les fonctions sont des variables globales de premier niveau, reliées aux éléments via des gestionnaires inline `onclick=`/`oninput=`.
- **Conserve les commentaires en français et le ton accessible aux débutants.** Les noms de variables et de fonctions sont en français (`ajouterNote`, `mesNotes`, `deplacerNoteVers`, `liens`). Respecte ça — ne renomme pas en anglais et ne supprime pas les commentaires explicatifs. La fin du fichier contient même une section « À TOI DE JOUER » d'exercices pour l'élève ; préserve cet esprit.
- Un nouveau comportement implique généralement : ajouter du balisage dans le `<body>`, le styler dans le bloc `<style>`, et ajouter une fonction globale dans le `<script>`, puis manipuler l'état via les variables `let` de niveau module déclarées vers le début du script (≈ lignes 660–727).

## Principes de développement

Ce projet vise un code **lisible par un débutant**. Trois principes guident chaque modification — applique-les avec discernement, jamais de façon dogmatique : la clarté pédagogique passe avant la pureté théorique.

- **DRY — *Don't Repeat Yourself* (ne te répète pas).** Si la même logique apparaît à deux endroits ou plus, factorise-la dans une fonction. Exemple concret ici : c'est exactement pourquoi on passe toujours par `sauvegarderNotesEnStockage()` au lieu de réécrire les appels `localStorage` partout, et par `escapeHtml()` plutôt que de ré-échapper le HTML à la main à chaque insertion. Avant d'écrire un bloc, vérifie qu'une fonction existante ne fait pas déjà le travail. *Nuance :* dupliquer une ligne ou deux peut rester plus clair qu'une abstraction tordue — ne crée pas une fonction « utilitaire » fourre-tout juste pour éviter deux lignes jumelles.

- **KISS — *Keep It Simple, Stupid* (reste simple).** Choisis toujours la solution la plus simple qui marche. Pas de framework, pas de système d'événements maison, pas d'astuce clever difficile à relire : du JavaScript direct, des fonctions courtes au nom explicite (`ajouterNote`, `deplacerNoteVers`). Si une fonction devient longue ou imbriquée, découpe-la. Le code doit pouvoir être lu et compris par l'élève à qui ce projet est destiné.

- **YAGNI — *You Aren't Gonna Need It* (tu n'en auras pas besoin).** N'ajoute que ce qui est demandé maintenant. Pas d'options « au cas où », pas de paramètres jamais utilisés, pas de généralisation prématurée pour des besoins hypothétiques. Une fonctionnalité non demandée, c'est du code à comprendre et à maintenir pour rien. Si un besoin futur se présente vraiment, on l'ajoutera à ce moment-là.

En cas de tension entre ces principes, **privilégie la lisibilité pour un débutant** (KISS) — quitte à tolérer une légère répétition (DRY) plutôt qu'une abstraction obscure.

## Principes de Clean Architecture

L'idée centrale de la *clean architecture* (Robert C. Martin) est de **séparer la logique métier des détails techniques** (affichage, stockage, navigateur) pour que le cœur de l'application reste compréhensible et indépendant. **Ici, on en applique l'esprit, PAS la mise en couches lourde** : un projet en un seul fichier destiné à un débutant ne doit pas être noyé sous des dossiers, des classes et des interfaces abstraites. On garde donc ces principes, traduits à notre échelle :

- **Séparer les responsabilités (separation of concerns).** Distingue clairement trois rôles, déjà présents dans le code :
  1. **les données / l'état** (`mesNotes`, `positionsNotes`, `liens`, `onglets`…) ;
  2. **la logique** qui les transforme (`ajouterNote`, `supprimerNotesParIndices`, `couleurValidee`…) ;
  3. **l'affichage et les entrées/sorties** (le DOM via `afficherLesNotes`/`dessinerLiens`, le stockage via `sauvegarderNotesEnStockage`/`chargerNotesDepuisStockage`).
  Une fonction qui modifie l'état ne devrait pas, en plus, fabriquer du HTML à la main : elle change les données puis appelle `afficherLesNotes()`.

- **La règle de dépendance : les détails dépendent de la logique, jamais l'inverse.** Le « quoi » (ajouter une note, relier deux notes) ne doit pas être écrit en fonction du « comment c'est affiché ou stocké ». Concrètement : ne dissémine pas d'appels directs à `localStorage` ou de `document.getElementById(...)` au milieu de la logique métier — passe par les fonctions dédiées (`sauvegarderNotesEnStockage`, `afficherLesNotes`). Ainsi, si on change un jour la façon d'afficher ou de stocker, la logique ne bouge pas.

- **Isoler les effets de bord (frontière entrée/sortie).** Tout ce qui touche au monde extérieur — `localStorage`, lecture de fichiers, insertion dans le DOM — est regroupé dans des fonctions identifiables, pas éparpillé. C'est aussi une frontière de **sécurité** : tout contenu venant de l'extérieur (fichier `.json` importé, notes partagées) passe par `escapeHtml()` / `couleurValidee()` avant d'entrer dans le DOM (voir « Note de sécurité »).

- **Responsabilité unique (le « S » de SOLID).** Chaque fonction fait **une** chose et porte un nom qui le dit. Si une fonction « ajoute une note **et** redessine **et** sauvegarde **et** recentre », c'est qu'elle mélange les rôles : découpe-la et fais-la appeler les fonctions spécialisées.

Règle d'arbitrage (cohérente avec ci-dessus) : applique ces principes **tant qu'ils rendent le code plus clair**. Si respecter une frontière demande une abstraction que l'élève ne comprendrait pas, **la lisibilité (KISS) l'emporte** — documente alors le petit écart en commentaire.

## Architecture (les parties qui s'étendent sur tout le fichier)

L'état vit dans des variables de niveau module et est répliqué dans `localStorage` à chaque modification. Concepts clés :

- **Clés de persistance.** Toutes les clés de stockage dérivent d'une base comme `mon-deuxieme-cerveau-notes` plus deux suffixes : `SUFFIXE_PARTAGE` (défini quand la page a été ouverte comme copie *partagée*, afin que les modifications d'un destinataire n'entrent jamais en collision avec ses propres notes personnelles) et le suffixe par onglet recalculé par `appliquerClesOnglet()`. Les sous-clés ajoutent `-positions`, `-liens`, `-reduites` à la `CLE_STOCKAGE` active. Quand tu touches au stockage, passe par `sauvegarderNotesEnStockage()` / `chargerNotesDepuisStockage()` plutôt que d'appeler `localStorage` directement, pour que toutes les clés liées restent synchronisées.
- **Onglets.** Chaque onglet est un jeu de notes indépendant avec son propre suffixe de stockage ; le premier onglet réutilise les clés héritées sans suffixe pour la rétrocompatibilité. Géré par `chargerOnglets` / `changerOnglet` / `appliquerClesOnglet`.
- **Notes, positions, liens.** `mesNotes[]` contient le contenu des notes ; `positionsNotes` contient la position x/y en glissé-libre de chaque note ; `liens[]` contient les connexions sous forme de paires d'index `{a, b}`, dessinées en SVG par `dessinerLiens()`. Comme les liens et les positions sont indexés par l'**index** du tableau, supprimer une note oblige à les ré-associer — voir `supprimerNotesParIndices()`.
- **Interaction sur le canevas.** La zone de travail gère le déplacement/zoom (`appliquerTransform`, `appliquerZoom`, `fixerZoomAutourDe`), le glissé-libre des notes (`demarrerDeplacement` → `deplacerNoteVers` → `terminerDeplacement`), la multi-sélection au rectangle (`demarrerSelectionRect` …), et une cible « glisser vers la corbeille ».
- **Les modes sont des booléens globaux** — `modeRelier`, `modeSupprimerLiens`, `modeSelection` — basculés par les fonctions `basculer…` ; ils changent ce que fait un clic sur une note. Vérifie/réinitialise ces modes quand tu ajoutes un nouveau comportement de clic.
- **Partage** (`partagerNotes`) affiche simplement l'URL de l'app déployée à copier. Quand une page est ouverte en portant une charge utile `window.__NOTES_PARTAGEES__`, `installerNotesPartageesUneFois()` initialise le stockage une seule fois à la première ouverture, puis laisse la copie du destinataire tranquille.
- **Import/export.** `telechargerNotes()` écrit un fichier JSON (`format: "mon-deuxieme-cerveau"`, contient les notes, positions, liens, état réduit, titre, description) ; `appliquerNotesImportees()` le relit et tolère un ancien format dépourvu de positions/liens.

## Note de sécurité

Les fichiers de notes importés et les charges utiles partagées sont des entrées non fiables rendues dans le DOM. `escapeHtml()` existe spécifiquement pour empêcher les attaques XSS via les notes importées (voir le commit « Sécurité : empêche l'injection de code (XSS) »). Échappe toujours le contenu utilisateur/importé avant de l'insérer en tant que HTML.
