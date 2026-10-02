# lgps_pdf — Contexte projet

> Fichier tenu à jour au fil des discussions. Toute décision importante (stack,
> architecture, contrainte, design, feuille de route) doit être ajoutée ici.

## Objectif
App mobile (Android + iOS) de lecture de livres PDF, centrée sur une ergonomie
de lecture adaptée et des fonctionnalités qui améliorent l'expérience de lecture.
Appareil de test principal : téléphone Android Xiaomi (HyperOS).

## Stack (versions réelles du pubspec)
- Flutter / Dart SDK ^3.13
- Rendu PDF : pdfrx ^2.6 (PDFium)
- État : Riverpod 3 — flutter_riverpod ^3.4, riverpod_annotation ^4, riverpod_generator ^4
- Navigation : go_router ^18
- Modèles : freezed ^4 / freezed_annotation ^3 + json_serializable
- Base locale : Drift ^2.35 + drift_flutter (SQLite)
- Préférences simples : shared_preferences
- Fichiers : file_picker, path_provider, path, crypto (hash anti-doublon)
- Confort de lecture : wakelock_plus, screen_brightness, flutter_tts
- Utilitaires : intl, uuid, collection
- Tests : flutter_test + mocktail

## Lints
- riverpod_lint passe par le système de plugins officiel de Dart :
  bloc `plugins:` de PREMIER NIVEAU dans analysis_options.yaml.
- NE PAS ajouter custom_lint : incompatible avec riverpod_lint >= 3.1.4
  (conflit de versions d'analyzer, testé et confirmé).
- Fichiers générés (*.g.dart, *.freezed.dart) exclus de l'analyse.

## Architecture (feature-first)
```
lib/
  main.dart, app.dart          # MaterialApp.router + thème
  core/
    database/                  # app_database.dart (@DriftDatabase) + tables/
                               #   books, bookmarks, highlights, reading_sessions
    router/                    # app_router.dart (go_router)
    theme/                     # app_theme.dart, reading_themes.dart (clair/sépia/sombre)
    services/                  # pdf_service.dart, file_storage_service.dart
    constants/  utils/  widgets/
  features/
    library/  reader/  annotations/  search/  tts/  stats/
      chacune : data/ domain/ presentation/(providers, screens, widgets)
test/
  fixtures/                    # PDF de test : lourd, scanné, protégé, images, paysage
  helpers/  features/
```
- domain : modèles freezed + interfaces de repository
- data : implémentations des repositories + sources Drift
- presentation : providers Riverpod, écrans, widgets

## Règles de code
- Toujours passer par `core/services/pdf_service.dart` : ne jamais appeler pdfrx
  directement depuis l'UI (permet de changer de lib PDF sans tout réécrire).
- Ne jamais modifier à la main les fichiers générés.
- Une seule solution de gestion d'état : Riverpod (pas de Bloc, pas de setState global).

## Contraintes et décisions de conception
- Un PDF a une mise en page fixe : pas de « changement de police » façon EPUB.
  L'ergonomie passe par : ajustement à la largeur / zoom, recadrage des marges,
  mode « texte extrait » optionnel (perd images et mise en page), thèmes de lecture.
- Thèmes clair / sépia / sombre via ColorFiltered ; garder un mode « original »
  car l'inversion de couleurs abîme les images.
- Performance : rendu des pages à la demande, miniatures basse résolution,
  jamais tout le document en mémoire. Tester avec un PDF de 500+ pages.
- Import : le PDF est COPIÉ dans le dossier de l'app (scoped storage Android),
  doublons détectés par hash.
- Surlignages / annotations : coordonnées RELATIVES à la page (0..1), stockées
  en base Drift ; le fichier PDF n'est jamais modifié.
- Android : intent-filter `application/pdf` pour apparaître dans « Ouvrir avec… »
  (à tester spécifiquement sur HyperOS).
- PDF protégés par mot de passe : prévoir un écran de saisie.
- PDF scannés (sans texte) : recherche et TTS indisponibles ; OCR
  (google_mlkit_text_recognition) envisagé plus tard.
- Accessibilité dès le départ : Semantics, contrastes, taille du texte de l'UI.

## Feuille de route
1. Import + bibliothèque avec miniature de la 1re page
2. Lecture + reprise automatique à la dernière page lue
3. Modes de lecture : défilement vertical / page par page, thèmes, luminosité,
   verrouillage rotation, écran toujours allumé
4. Marque-pages + recherche dans le texte
5. Surlignages + notes
6. TTS, statistiques de lecture, défilement automatique, double page sur tablette
7. (Plus tard) synchronisation cloud — attention au coût de stockage des fichiers

## Commandes
- Génération de code (à laisser tourner) : `dart run build_runner watch -d`
- Lancer sur le téléphone : `flutter run`

## Git
- Claude (Cowork) met à jour ce fichier mais ne fait jamais de commit ni de push.
- Relire `git diff CLAUDE.md` avant de commiter si plusieurs outils l'ont modifié.
