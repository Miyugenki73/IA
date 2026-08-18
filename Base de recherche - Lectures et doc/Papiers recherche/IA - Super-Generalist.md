# Modele étudié : SuG (Super-Generalist)
*Organes étudiées : abdomen*

## Différence dans les modeles
Compromis entre le modele specialiste et Generaliste : 
- **Specialiste** (ex. nnU-Net) : excelle dans l'id de structures fines et detection locale grace à des annotations précises au niveaux de certains pixels. Portée étroite, limitée à des tâches/maladies spécifiques et extension coûte cher en annotation
- **Generaliste** (sur du vision-language) : découvrent les spectres de pathologies via l'alignement cross modal de l'imagerie et les rapports textuels. Echoue souvent à détecter les anomalies anatomiques fines car sappuie sur des informations visuelles globales plutôts que les caractéristiques locales des lésions - limite la précision et traçabilité clinique. 
- **SuG** : recherche une couverture universelle, bonnes perf face au speciliaste et localisation visuelle de qualité sur des lésions

*Cross-modal  Imagerie et Texte - Créer un vecteur numérique pour mesurer la proximité sémantique. On a soit un alignement global (texte pour l'image) soit grin fin (un mot pour désigner une image précise). Existe un problème de bruit selon le jargon médical et détails manquants/ambigus dans le texte.*

## Architecture du SuG
2 Branches - Alignement vision/language - Calibrage attention guidé par les lésions

1. Branche vision : **encodeur visuel** et **décodeur spécialiste** pour extraire les caractéristiques d'images multi-echelles et réaliser des segmentations denses sur les "Voxels" (le plus petit élement constituant un espace en 3Dim)
2. Branche texte : **encodeur texte** pour convertir rapport médicaux en tokens au niveau anatomique et tokens de prompts de lésions
3. Alignement Vision-language amélioré par le Spécialiste : utilisation de masques d'organes "prédis par le décodeur spécialiste" pour gudier l'alignement constrastif entrel es caractéristiques visuelles locales (mutli-échelles) et les descriptions textuelles associées
4. Calibrage de l'attention guidé par les lésions : utilise les masques de lésions comme "a priori spatiaux" pour affiner dynamiquement l'attention cross-modal et focaliser les représentations sémantiques sur les régions "d'intéret pathologiques"

## Méthodo
A. Segmentation détaillé multi-taches duy spécialiste (Voxel)
1. Segmentation de l'anatomie : id précisément les stuctures anatomiques
2. Segmentation des lésions spécifiques (Class-Spécific) : prédire les contours des tumeurs ou anomalies spécifiques pour lesquelles les classes d'annotation précises existent (ex. tumeur gastrique)
3. Segmentation des lésions agnostiques (Class-Agnostic) : Fusionner toutes les anomalies annotées en une seule classe d'arriere-plan pathologique afin d'apprendre des caractéristiques communes aux lésions. Pour surmonter le fait que de nombreuses lésions ne sont pas annotées systématiquement en dehors des organes ciblés, les auteurs appliquent une *perte masquée* (Validity mask) pour éviter de pénaliser le modele sur des régions non supervisées.
B. Aligment vision-language amélioré par le spécialiste
- Utilise un grande modele de language (ex. Qwen LLM) pour analyser le rapport de radio et en extraire une description texteuelle propre a chaque structure anatomique
- le SuG utilise les masques anatomiques prédits par la branche spé pour extraire les tokens visuels correspondants spécifiquement à chaque organe
- Représentations anatomiques visuelles (agrégées sur pls echelles de caract. et normalisées) alignées de maniere contrastive (InfoNCE adaptative) avec leurs descriptions textuelles respectives dans un espace partagé
C. Calibrage de l'attention
Pour s'assurer de la bonne lecture des éléments cliniquement pertinents lors de la relecture du diag de lésion, les cartes d'attentions visuelle générées par les prompts textuels de lésions sont directement supervisées par les masques de lésions issus de la branche spécialiste. Attention sémantique est ainsi "calibrée" pour cibler la zone tumorale exacte, ce qui élimine le bruit

## Optimisation
3 phases pour garantir stabilité de l'entrainement et équilibrer les objectifs
1. Que spécialiste : entrainement exclusif de la branche de segmentation (anatomie et lésions au niveau voxel) pour ancrer les caractéristiques géométriques et anatomiques
2. Segmentation + alignement : Co-optimisation de la segmentation et de l'alignement constratif vision-language basé sur les organes
3. Intégrationb du calibrage dans l'attention :Intro de la perte de calibrage d'attention guidée par les masques de lésions pour stabiliser l'apprentissage localisé
 