# CLAUDE.md — Projet « Guide » (jeu Unity, chien guide)

Ce fichier est lu par Claude Code à chaque session. Il décrit le projet, les règles et les conventions.

## Le projet en bref

Jeu 3D Unity où l'on incarne surtout une personne aveugle guidée par son chien dans la rue :
- **Mode maître (principal)** : monde sombre ; chaque source de bruit porte un outline animé et émet une onde qui révèle les arêtes du décor. Le joueur écoute, lit les signaux du chien (arrêt, assis, traction ; rendus par vibration de la manette et indicateur) et donne les ordres.
- **Mode chien (secondaire, rôle à décider)** : troisième personne, obéir ou refuser les ordres (désobéissance intelligente), signaler les dangers.

Un seul monde, deux couches de perception. Assets : packs Kenney (CC0) — Mini Characters, Cube Pets, City Kit (Roads), City Kit (Commercial).

## Environnement

- Unity : 6000.3.27f1
- Pipeline de rendu : URP
- Input : nouveau Input System (`UnityEngine.InputSystem`)
- IDE : Visual Studio ; Claude Code tourne dans un terminal à la racine du projet.

## Ce que Claude Code fait et ne fait pas

Claude Code **écrit et modifie uniquement des scripts C# et des shaders** (`Assets/_Project/Scripts/`, `Assets/_Project/Shaders/`).

Ne jamais :
- modifier des fichiers `.unity`, `.prefab`, `.asset`, `.mat`, `.meta` ou `ProjectSettings/` : la scène, les prefabs et les réglages se font à la main dans l'éditeur ;
- toucher aux dossiers `Library/`, `Temp/`, `Logs/`, `UserSettings/` ni aux packs importés (`Assets/Kenney/`) ;
- ajouter un package (manifest) sans le demander.

Quand un script demande un réglage dans l'éditeur (composant à ajouter, référence à glisser dans l'inspecteur, layer, tag), **le dire explicitement à la fin de la réponse**, sous forme de liste d'étapes.

## Arborescence

```
Assets/
  _Project/
    Scripts/
      Core/          # bus d'événements sonores, utilitaires
      Dog/           # DogController, harnais
      Handler/       # HandlerFollower, CommandSystem, TrustMeter
      Hazards/       # HazardDefinition (ScriptableObject), HazardSpawner
      Street/        # TrafficLight, circulation
      QTE/           # QTEManager
      Echo/          # EchoPulseManager (mode maître)
      UI/
    Shaders/         # shader d'écho (HLSL ou Shader Graph)
    ScriptableObjects/
  Kenney/            # packs importés, ne pas modifier
```

## Architecture

- **Bus d'événements sonores** (`SoundEvent` : position, intensité, type) : tout ce qui fait du bruit le publie. Il déclenche l'audio et, en mode maître, une onde visuelle (outline sonore).
- **Shader d'outline sonore** : détection d'arêtes en post-traitement (profondeur + normales), affichée seulement là où passe une onde ; outline animé permanent sur les sources qui émettent.
- **Données dans des ScriptableObjects** (types de dangers, ordres, réglages de difficulté) plutôt que codées en dur.
- **Machines à états simples** (enum + switch, ou classes d'état) pour le chien, le maître et les feux.
- Composants petits, une responsabilité chacun ; communication par événements (C# `event` / `Action`) plutôt que par `FindObjectOfType` ou `GameObject.Find`.

## Conventions C#

- Namespaces : `Guide.Core`, `Guide.Dog`, `Guide.Handler`, `Guide.Hazards`, `Guide.Street`, `Guide.QTE`, `Guide.Echo`, `Guide.UI`.
- Champs sérialisés : `[SerializeField] private` + `camelCase` ; propriétés et méthodes publiques en `PascalCase`.
- Pas de `public` sur les champs juste pour l'inspecteur.
- `[RequireComponent]` quand un composant en dépend d'un autre ; `[Tooltip]` sur les réglages non évidents.
- Mettre en cache les `GetComponent` dans `Awake` ; aucune allocation dans `Update`.
- Commentaires XML (`///`) sur les classes et méthodes publiques, en français.

## Façon de travailler

- Avant une fonctionnalité importante : proposer d'abord un plan (mode Plan), attendre la validation.
- Changements petits et ciblés ; ne pas réécrire un fichier entier pour une modification locale.
- Après une modification : indiquer les fichiers touchés et comment tester dans le Play Mode.
- En cas de doute sur une API Unity ou sur la version : le dire plutôt que deviner.

## Jalons

1. Socle : Git, `.gitignore` Unity, ce fichier, import Kenney.
2. Prototype du shader d'outline sonore (scène vide, sources qui émettent au clic, onde qui se propage sur les arêtes).
3. Mode maître jouable : une rue, sons spatialisés, signaux du chien, ordres et refus, 4–5 dangers, un feu, jauge de confiance.
4. QTE et score.
5. Mode chien sur le même trajet (tutoriel ou chapitre bonus).
6. Finitions : mixage audio, menus, accessibilité.
