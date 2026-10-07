# Veille IA Souveraine : Assistant RAG Local & Pipeline ETL

Système autonome, souverain et hautement sécurisé de veille technologique automatisée et d'interrogation augmentée par la recherche (RAG), conçu spécifiquement pour s'exécuter localement sous Windows/WSL2.

L'architecture transforme des flux documentaires non structurés en une base de connaissances pérenne (Markdown + YAML Frontmatter) et vectorielle (ChromaDB), sans jamais exposer la moindre donnée ou requête à des services cloud tiers (coût d'exploitation externe maintenu à 0 €).

---

## 🎯 Piliers de Qualité & Invariants d'Architecture

1. **Exactitude & Intégrité Sémantique**
   - **Grammaire coercitive** : utilisation de l'API des sorties structurées (*Structured Outputs*) d'Ollama adossée aux modèles de données Pydantic v2. Élimination totale des hallucinations de format.
   - **Attribution stricte** : 100 % des synthèses générées par l'agent interactif incluent une citation formelle et traçable pointant vers le fichier source Markdown.
   - **Typage statique absolu** : base de code Python analysée avec **Mypy en mode strict** (`--strict`), avec **interdiction formelle et définitive du type `Any`**.

2. **Sécurité native (*Secured by Design*)**
   - **Immunité sémantique** : encapsulation stricte du texte brut collecté entre les balises `[DÉBUT DU CONTEXTE]` et `[FIN DU CONTEXTE]` avec interdiction d'exécution transmise au LLM pour neutraliser les injections de prompt indirectes.
   - **Filtre anti-bruit** : classification préalable par Llama 3 émettant un statut `REJECT` pour écarter immédiatement pages de connexion, cookies et contenus publicitaires avant vectorisation.
   - **Principe du moindre privilège** : exécution non-root (`USER 1000:1000`) dans des conteneurs isolés sur l'interface de bouclage `127.0.0.1`. Ségrégation stricte des volumes : accès lecture/écriture (`rw`) pour l'ETL, accès restreint en lecture seule (`:ro`) pour le RAG.

3. **Optimisation Matérielle AMD & Sanctuarisation WSL2**
   - **Exécution locale** : optimisée pour le processeur **AMD Ryzen AI 9** et ses 64 Go de mémoire unifiée (exclusion formelle des pilotes et outils NVIDIA).
   - **Déport asymétrique** : vectorisation (`nomic-embed-text`) déportée sur le CPU afin de libérer les ressources pour la génération de texte (`Llama 3`).
   - **Verrouillage hyperviseur** : sanctuarisation obligatoire de l'hôte Windows via `.wslconfig` (`memory=40GB`, `processors=16`, `swap=0`) pour éviter la saturation de la RAM hôte et la dégradation des entrées/sorties disque.
   - **Stockage natif Linux** : persistance assurée exclusivement par des volumes Docker nommés (`veille_knowledge_data`) sous ext4, excluant tout montage direct (*bind mount* sur `/mnt/c/`) pour préserver les performances I/O.

---

## 🏗️ Architecture des Composants

| Composant | Rôle technique | Privilèges & Confinement |
| :--- | :--- | :--- |
| **Pipeline ETL** | Collecte (flux RSS, scraping furtif Playwright), filtrage Pydantic, dédoublonnage sémantique et écriture atomique | Volume en lecture/écriture (`rw`), `USER 1000:1000` |
| **Queue SQLite** | Mémorisation transactionnelle des états et ordonnancement résilient (APScheduler) face aux instabilités réseau | Localisé sur volume nommé |
| **Ollama** | Inférence locale (`Llama 3 8B`) et génération vectorielle (`nomic-embed-text`) | Confiné sur `127.0.0.1:11434` |
| **ChromaDB** | Magasin vectoriel persistant | Isolé dans le volume Docker |
| **Agent RAG (FastAPI)** | Interrogation en langage naturel, troncature dynamique du contexte (`tiktoken`) et restitution sourcée | Confiné sur `127.0.0.1:8000`, volume en lecture seule (`:ro`) |

---

## 📋 Prérequis & Configuration Hôte Obligatoire

Avant tout déploiement, vous devez sanctuariser votre hôte Windows en créant le fichier `%USERPROFILE%\.wslconfig` :

```ini
[wsl2]
memory=40GB
processors=16
swap=0
