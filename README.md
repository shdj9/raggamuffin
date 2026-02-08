# Projet RAG à Muffin 🧁

Bienvenue dans l'atelier de Winston Muffin, un assistant culinaire intelligent spécialisé dans l'art du muffin. 

# 🛠️ Architecture du Projet
# 1. Collecte des Données
Le dataset repose sur l'extraction automatisée de 574 recettes via la recherche "muffin" sur Marmiton, qui contient des recettes en français.

Contenu : Titre, ingrédients, instructions, durées, portions et URLs sources.

Qualité des données : Le dataset reflète la réalité du web (langue naturelle, fautes d'orthographe, imprécisions des utilisateurs), que l'on retrouve plus ou moins dans les réponses du chef...

Fichier : muffins_marmiton_VF.json (pas nécessaire de relancer le scraping, les données sont prêtes à l'emploi).

# 2. Embedding et stockage des vecteurs mathématiques
Pour transformer les recettes en "vecteurs mathématiques", j'ai utilisé :

Modèle : paraphrase-multilingual-MiniLM-L12-v2, modèle optimisé pour le français, comme préconisé

Stratégie d'Indexation : Seuls le titre et les ingrédients ont été vectorisés.
L'inclusion des instructions ajoutait trop de "bruit" sémantique (verbes d'action, termes techniques), qui "diluait" les infos ingrédients principaux.

# 3. Le Moteur RAG & Génération

Le système suit une architecture Retrieval-Augmented Generation :

Retrieval : Extraction des 5 recettes les plus proches dans la base vectorielle ChromaDB. Des tests à $k=10$ ont été effectués, mais n'ont pas montré d'amélioration significative de la pertinence. 
J’ai affiché systématiquement pour mes tests le « podium » des 5 recettes ; par contre, dans cette version finale, pas de podium retourné sur la page pour l’utilisateur. 

Génération : Orchestration via Mistral AI avec un prompt définissant le persona de Winston Muffin. Il s'efforce de renvoyer les recettes les plus pertinentes parmi celles du contexte. 

### Observations : 

Au départ, l'objectif était de fixer les garde-fous, pour que le chef comprenne qu'il ne cuisinait que des muffins. À cause des refus stricts que j'avais mis, j'ai obtenu des réponses trop rigides. Si mes prompts invitant le chef à la rigueur étaient efficaces pour la précision, ils étaient frustrants pour l'utilisateur, le Chef perdait son côté sympathique en devenant trop binaire. À l'inverse, après des modifications, le chef a eu tendance à proposer des recettes pour "consoler" l'utilisateur, qui étaient trop éloignées de la demande. 

Le prompt final combine la vibe Reggae pour le persona et une logique de sélection selon la présence des ingrédients demandés dans les recettes obtenues par le RAG. Les réponses ont dues être formatées pour corriger les problèmes d'affichage (notamment pour forcer des sauts de ligne).

# 4. Alternative détaillée à la fin de la partie Optimisation

# ↗️ Optimisation 
Le modèle est concluant quand on donne des ingrédients fréquents, plutôt sucrés mais pas que. 
Le système a montré des limites sur certaines requêtes, avec un biais de répétition sur quelques recettes spécifiques lorsque le RAG ne trouvait pas de match évident. De plus, les résultats se sont montrés décevants pour des ingrédients plus rares. Plusieurs solutions ont été testées pour contrer ce phénomène :

1. Le filtre par distance sémantique : J'ai tenté de définir un seuil basé sur la distance calculée par ChromaDB pour exclure les recettes trop éloignées de la requête. Cette piste s'est révélée non concluante : les distances restaient mathématiquement proches, que la recette soit sémantiquement cohérente ou totalement hors-sujet...

2. Le tri mécanique post-retrieval : Un algorithme de tri par mots-clés après la récupération a été testé pour réorganiser les 10 meilleurs résultats. Cette méthode n'a pas fonctionné car si la phase de RAG initiale échoue avec 10 recettes qui ne correspondent pas à la query, un tri algorithmique simple ne peut pas compenser la faiblesse du contexte.

### 3. Alternative : Pondération par "Ingrédients principaux"
Pour améliorer les résultats du RAG, j'ai essayé une autre approche : l'utilisation de Mistral en amont pour identifier les 2 ou 3 ingrédients principaux de chaque recette. Le fichier "muffins_enriched_dataset.json" contient ces ingrédients principaux et l'embedding associé. 

Dans l'embedding, ces ingrédients ont été répétés trois fois afin d'augmenter leur poids vectoriel. Bien que les résultats soient globalement équivalents à la version standard, cette méthode a montré une légère amélioration sur des requêtes très spécifiques dont les ingrédients étaient auparavant noyés dans la liste globale. 

Les fonctions liées à cette expérimentation sont conservées en fin de notebook à titre de documentation technique. 
Vous pouvez les exécuter à la suite des autres pour tester la performance de cette alternative.
