# IA
## Ludo - Travail

## Pierre - Travail
En cours // Lecture de doc : 
- A survey on deep learning in medical image analysis (Litjens et al.)
- Machine learning methods for histopathological image analysis: Updates in 2024
- RadCLIP: Enhancing Radiologic Image Analysis through Contrastive Language-Image Pre-training
- Analyse du model DINO
- Vérif si indic de qualité en *train.csv* disponible 

Fait :
- Super-Generalist: Towards Comprehensive and Accurate Medical Image Understanding via Generalist–Specialist Synergy
- recherche de la struct possible 
### Ouvrages disponibles
- Mathematics for Machine Learning de Deisenroth, Faisal & Ong
- MONAI (arxiv 2211.02701)
### Modeles disponibles
- CNN 3D natifs (DenseNet3D, EfficientNet3D, ResNet3D)
- nnU-Net
- Vision Transformers (ViT) et hybrides CNN-ViT
- Transformers hierarchiques (Swin, etc.)
- Modeles a espace d'etat (Mamba)
- Pré-entrainement auto-supervisé (Masked Autoencoders, contrastif type Barlow Twins/DINO)
### Struct possible
1. BackBone image - DINO V2 + LoRa avec pooling pondéré de série + prétrait DDICOM
2. Backbone texte - utilisé pour entrainement du model teacher multimodal (predict 12 labels puis distil prédict dans le student "Label only". utilise le student a la fin pour le test ayant hérité du teacher sans avoir besoin de texte) // point de vigilance : peu de cas on de bonnes annotations, pondéré selon la qualité de l'annotation (si gold, % important, sinon faible => utiliser comme soft distill)
3. Fusion image/texte avec les 12 patho - Que branche image (25 sec par étude)