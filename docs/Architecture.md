# Architecture - AI Ops Copilot

## Vue d'Ensemble

**AI Ops Copilot** utilise une architecture multi-agents modulaire conçue pour la scalabilité et la maintenabilité.

## Diagramme ASCII - Architecture Globale

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           AI OPS COPILOT ARCHITECTURE                      │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Log Sources   │    │   Log Sources   │    │   Log Sources   │
│  (Cloudflared)  │    │ (Local Files)   │    │  (API Streams)  │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      LOG COLLECTOR      │
                    │   - File Watcher        │
                    │   - Stream Parser       │
                    │   - Rate Limiting       │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │      LOG PARSER         │
                    │   - Regex Engine        │
                    │   - JSON Extractor      │
                    │   - Field Normalizer    │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │    ANOMALY DETECTOR     │
                    │   - Pattern Analysis    │
                    │   - Threshold Monitor   │
                    │   - ML Classification   │
                    └────────────┬────────────┘
                                 │
                         ┌───────▼───────┐
                         │ ERROR DETECTED │
                         └───────┬───────┘
                                 │
                    ┌────────────▼────────────┐
                    │   MULTI-AGENT SYSTEM   │
                    │                         │
                    │  ┌─────────────────┐   │
                    │  │ Context Agent   │   │
                    │  │ - Log Enricher  │   │
                    │  │ - App Context   │   │
                    │  └─────────┬───────┘   │
                    │            │           │
                    │  ┌─────────▼───────┐   │
                    │  │ Diagnosis Agent │   │
                    │  │ - Root Cause    │   │
                    │  │ - Error Class   │   │
                    │  └─────────┬───────┘   │
                    │            │           │
                    │  ┌─────────▼───────┐   │
                    │  │  Fixer Agent    │   │
                    │  │ - Solution Gen  │   │
                    │  │ - Code Patches  │   │
                    │  └─────────┬───────┘   │
                    │            │           │
                    │  ┌─────────▼───────┐   │
                    │  │ Reporter Agent  │   │
                    │  │ - Summary Gen   │   │
                    │  │ - Notification  │   │
                    │  └─────────┬───────┘   │
                    └────────────┼───────────┘
                                 │
                    ┌────────────▼────────────┐
                    │    STORAGE LAYER        │
                    │                         │
                    │  ┌─────────────────┐   │
                    │  │ Qdrant Vector   │   │
                    │  │ - Embeddings    │   │
                    │  │ - Similarity    │   │
                    │  └─────────────────┘   │
                    │                         │
                    │  ┌─────────────────┐   │
                    │  │ SQLite History  │   │
                    │  │ - Metadata      │   │
                    │  │ - Relations     │   │
                    │  └─────────────────┘   │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   INTERFACE LAYER       │
                    │                         │
                    │  ┌─────────────────┐   │
                    │  │ Streamlit UI    │   │
                    │  │ - Dashboard     │   │
                    │  │ - Chat Bot      │   │
                    │  │ - Search        │   │
                    │  └─────────────────┘   │
                    │                         │
                    │  ┌─────────────────┐   │
                    │  │ Email Notifier  │   │
                    │  │ - SMTP Client   │   │
                    │  │ - Templates     │   │
                    │  └─────────────────┘   │
                    └─────────────────────────┘
```

## Flux de Données Détaillé

```
Log Entry → Collector → Parser → Anomaly Detection
                                        │
                                   [IF ERROR]
                                        │
                                        ▼
                              ┌─────────────────┐
                              │ Context Agent   │
                              │ Enriches log    │
                              │ with metadata   │
                              └─────────┬───────┘
                                        │
                                        ▼
                              ┌─────────────────┐
                              │ Diagnosis Agent │
                              │ Identifies      │
                              │ root cause      │
                              └─────────┬───────┘
                                        │
                                        ▼
                              ┌─────────────────┐
                              │ Fixer Agent     │
                              │ Generates       │
                              │ solution        │
                              └─────────┬───────┘
                                        │
                                        ▼
                              ┌─────────────────┐
                              │ Reporter Agent  │
                              │ Creates summary │
                              │ & notifications │
                              └─────────┬───────┘
                                        │
                                        ▼
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
            ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
            │   Qdrant    │    │   SQLite    │    │    Email    │
            │  (Vectors)  │    │ (Metadata)  │    │ Notification│
            └─────────────┘    └─────────────┘    └─────────────┘
                    │                   │                   
                    └───────────────────┼───────────────────┘
                                        │
                                        ▼
                              ┌─────────────────┐
                              │ Streamlit UI    │
                              │ - Dashboard     │
                              │ - Search        │
                              │ - Chat          │
                              └─────────────────┘
```

## Modules Détaillés

### 1. Log Collector Module
**Localisation :** `src/collector/`

**Responsabilités :**
- Surveillance temps réel des sources de logs
- Ingestion multi-format (JSON, plain text, structured)
- Gestion de la charge et rate limiting
- Récupération automatique en cas de panne

**Composants :**
- `FileWatcher` : Surveillance des fichiers locaux
- `CloudflaredCollector` : Ingestion logs Cloudflared
- `StreamProcessor` : Traitement des flux temps réel
- `BufferManager` : Gestion des buffers et backpressure

### 2. Log Parser Module
**Localisation :** `src/parser/`

**Responsabilités :**
- Extraction des champs structurés
- Normalisation des formats
- Classification des niveaux de log
- Détection des patterns d'erreur

**Composants :**
- `RegexEngine` : Moteur de parsing regex
- `JSONExtractor` : Extraction de données JSON
- `FieldNormalizer` : Standardisation des champs
- `PatternMatcher` : Reconnaissance de patterns

### 3. Storage Module
**Localisation :** `src/storage/`

**Responsabilités :**
- Stockage vectoriel dans Qdrant
- Persistance métadonnées SQLite
- Gestion des embeddings
- Indexation pour recherche rapide

**Composants :**
- `QdrantClient` : Interface Qdrant
- `EmbeddingGenerator` : Génération d'embeddings
- `SQLiteManager` : Gestion base relationnelle
- `IndexManager` : Optimisation des index

### 4. Multi-Agent System
**Localisation :** `src/agents/`

#### Context Agent
**Responsabilités :**
- Enrichissement contextuel des logs
- Récupération métadonnées applicatives
- Corrélation avec événements système
- Historique des déploiements

**Inputs :** Log brut, métadonnées système
**Outputs :** Log enrichi avec contexte applicatif

#### Diagnosis Agent
**Responsabilités :**
- Analyse des causes racines
- Classification des types d'erreur
- Corrélation avec incidents passés
- Évaluation de la criticité

**Inputs :** Log enrichi, historique similaire
**Outputs :** Diagnostic avec cause probable et criticité

#### Fixer Agent
**Responsabilités :**
- Génération de solutions concrètes
- Propositions de patches de code
- Recommandations de configuration
- Actions de mitigation

**Inputs :** Diagnostic, contexte technique
**Outputs :** Solutions structurées avec étapes d'implémentation

#### Reporter Agent
**Responsabilités :**
- Synthèse des analyses
- Génération de rapports
- Préparation des notifications
- Mise à jour du dashboard

**Inputs :** Diagnostic + Solutions
**Outputs :** Rapport structuré, notifications

### 5. Interface Layer
**Localisation :** `src/ui/`

**Composants :**
- `StreamlitDashboard` : Interface web principale
- `ChatBot` : Assistant IA interactif
- `SearchEngine` : Recherche sémantique
- `NotificationManager` : Gestion des alertes

## Pipeline Multi-Agents

### Orchestration CrewAI
```python
# Exemple de configuration du pipeline
crew = Crew(
    agents=[context_agent, diagnosis_agent, fixer_agent, reporter_agent],
    tasks=[
        enrich_context_task,
        diagnose_error_task,
        generate_solution_task,
        create_report_task
    ],
    process=Process.sequential,
    memory=True,
    cache=True,
    max_execution_time=300
)
```

### Flux d'Exécution
1. **Trigger** : Détection d'erreur par l'Anomaly Detector
2. **Context Agent** : Enrichissement (30s max)
3. **Diagnosis Agent** : Analyse cause racine (60s max)
4. **Fixer Agent** : Génération solution (90s max)
5. **Reporter Agent** : Synthèse et notification (30s max)
6. **Storage** : Persistance résultats
7. **Notification** : Envoi alertes

### Gestion des Erreurs
- **Timeout** : Chaque agent a un timeout configuré
- **Retry** : 3 tentatives avec backoff exponentiel
- **Fallback** : Solutions génériques si agents échouent
- **Circuit Breaker** : Protection contre les cascades d'erreurs

## Patterns Architecturaux

### 1. Event-Driven Architecture
- Communication asynchrone entre modules
- Event bus pour découplage
- Résilience aux pannes

### 2. Plugin Architecture
- Modules interchangeables
- Configuration dynamique
- Extensibilité

### 3. CQRS (Command Query Responsibility Segregation)
- Séparation lecture/écriture
- Optimisation des performances
- Scalabilité

## Considérations de Performance

### Métriques Cibles
- **Latence** : < 100ms par log
- **Throughput** : > 1000 logs/sec
- **Disponibilité** : 99.9%
- **Précision** : > 85% diagnostics corrects

### Optimisations
- Cache multi-niveaux
- Indexation intelligente
- Parallélisation des agents
- Compression des données

---
*Dernière mise à jour : 07/01/2025*