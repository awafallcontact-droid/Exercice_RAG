# README — Mon RAG sur un document réel

## 1. Le projet

- **Auteur :** Awa Fall
- **Document choisi :** `Guide_agricole_pratique.docx` (guide agricole de 20 pages en texte natif).
- **Pourquoi ce document :** le guide réunit des repères agronomiques organisés par thème; il permet de vérifier si la recherche retrouve les consignes dans la bonne section.
- **Notebook :** [Notebook Colab RAG](https://colab.research.google.com/drive/1atOKTdmcML_-pEFTOVaoXHm2pKWV8nFJ?usp=sharing)

## 2. Les choix technologiques, brique par brique

| Brique du pipeline | Choix réalisé | Pourquoi ce choix | Alternative possible |
| --- | --- | --- | --- |
| **1. Lecture du document** | `python-docx` | Extrait le texte et les sauts de page du document Word natif. | `pypdf` pour un PDF texte, PyMuPDF |
| **2. Découpage** | Un chunk par page, borné par les sauts de page explicites | Chaque page du guide correspond à une section courte; les IDs restent faciles à retrouver (`P1-1` à `P20-1`). | Découpage par longueur fixe ou par paragraphes |
| **3. Embeddings** | Aucun | Le prototype utilise une recherche lexicale; cela évite un modèle ou un service externe. | `sentence-transformers` multilingue |
| **4. Stockage** | Liste de dictionnaires Python en mémoire | Le corpus est petit (20 chunks) et n’exige pas de base vectorielle. | NumPy, FAISS, ChromaDB |
| **5. Recherche** | Couverture pondérée des termes par fréquence documentaire | Les termes rares pèsent davantage; un seuil de 0,40 permet de refuser les questions trop éloignées du corpus. | BM25, recherche dense, approche hybride |
| **6. Réponse** | Extraction de jusqu’à deux phrases du chunk classé premier | La réponse reste directement vérifiable dans la source et n’est pas reformulée par un modèle génératif. | Modèle de langage avec contexte et citations contrôlées |
| **7. Contrôle et évaluation** | Fonctions maison `verifier_citations` et `evaluer` | Les tests vérifient la source attendue et signalent tout ID de citation inconnu. | Ragas, TruLens |

## 3. Jeu de test et résultats

Les cinq questions ci-dessous ont été exécutées avec `evaluer(MON_JEU_DE_TEST)`. Dans chaque cas, le chunk attendu a été retrouvé et aucune citation inventée n’a été détectée.

| Question testée | Chunk attendu | Résultat |
| --- | --- | --- |
| Quels éléments le diagnostic initial demande-t-il de relever sur chaque parcelle ? | `P3-1` | Source retrouvée; citation valide |
| Quels sont les principes complémentaires de l’agriculture de conservation cités dans le guide ? | `P5-1` | Source retrouvée; citation valide |
| Selon le guide, sur quels paramètres l’eau doit-elle être gérée ? | `P7-1` | Source retrouvée; citation valide |
| Que faut-il vérifier avant d’utiliser un produit phytopharmaceutique ? | `P9-1` | Source retrouvée; citation valide |
| Dans le carnet de suivi, que faut-il inscrire à la fin du cycle ? | `P19-1` | Source retrouvée; citation valide |

**Question hors document :** « Quel est le prix du blé en dollars cette année ? »

**Résultat obtenu :** `INFORMATION ABSENTE`.

## 4. Résultats des fonctions d’évaluation

- `evaluer(MON_JEU_DE_TEST)` : 5 sources attendues retrouvées sur 5.
- `verifier_citations(...)` : aucune citation inventée (`[]` pour chacune des cinq réponses).
- La question hors document renvoie `INFORMATION ABSENTE`.

Le prototype utilise une seule stratégie de recherche lexicale. Il ne produit donc pas de scores comparatifs BM25, dense ou hybride.


