# MVP (Minimum Viable Product) - AI Ops Copilot

## Définition du MVP

Le **MVP d'AI Ops Copilot** est une version simplifiée mais fonctionnelle du système qui démontre la valeur core : **détection automatique d'erreurs dans les logs et génération de diagnostics IA**.

## 🎯 Objectif MVP

Prouver que le système peut :
1. **Ingérer** des logs en temps réel
2. **Détecter** automatiquement les erreurs
3. **Diagnostiquer** via IA les causes probables
4. **Notifier** les équipes avec des solutions

## 📋 Fonctionnalités Core MVP

### 1. Ingestion Basique ✅
- **Source unique** : Un fichier de log local
- **Format** : Logs texte standard (Apache/Nginx style)
- **Surveillance** : Monitoring temps réel avec `watchdog`
- **Volume** : Support 100 logs/minute minimum

**Exemple de log supporté :**
```
2025-01-07 14:30:15 [ERROR] api.users.create - Database connection failed: Connection timeout after 30s
2025-01-07 14:30:16 [INFO] api.users.list - Retrieved 150 users successfully
2025-01-07 14:30:17 [ERROR] api.orders.process - Payment gateway error: Invalid API key
```

### 2. Parsing Simple ✅
- **Extraction** : timestamp, level, message
- **Patterns** : Regex configurables
- **Normalisation** : Format uniforme
- **Classification** : ERROR, WARN, INFO, DEBUG

**Structure de sortie :**
```python
{
    "timestamp": "2025-01-07T14:30:15",
    "level": "ERROR",
    "component": "api.users.create",
    "message": "Database connection failed: Connection timeout after 30s",
    "raw_log": "2025-01-07 14:30:15 [ERROR] api.users.create - Database connection failed: Connection timeout after 30s"
}
```

### 3. Détection d'Erreur ✅
- **Patterns** : Reconnaissance ERROR/EXCEPTION/FATAL
- **Mots-clés** : "failed", "error", "exception", "timeout"
- **Seuils** : Configurable par niveau
- **Filtrage** : Éviter les faux positifs

**Règles de détection :**
```python
ERROR_PATTERNS = [
    r'\[ERROR\]',
    r'Exception:',
    r'failed',
    r'timeout',
    r'connection.*error',
    r'database.*error'
]
```

### 4. Agent IA Unique ✅
- **Modèle** : Llama 3.1 7B via Ollama
- **Fonction** : Diagnostic + Solution combinés
- **Timeout** : 2 minutes maximum
- **Fallback** : Solution générique si échec

**Prompt template :**
```
Analyse ce log d'erreur et fournis :
1. Cause probable
2. Solution recommandée
3. Niveau de criticité (1-5)

Log: {log_message}
Contexte: {context}
```

### 5. Stockage Minimal ✅
- **Base** : SQLite uniquement
- **Tables** : logs, errors, diagnostics
- **Rétention** : 30 jours par défaut
- **Index** : Sur timestamp et level

**Schéma SQLite :**
```sql
CREATE TABLE logs (
    id INTEGER PRIMARY KEY,
    timestamp DATETIME,
    level VARCHAR(10),
    component VARCHAR(100),
    message TEXT,
    raw_log TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE diagnostics (
    id INTEGER PRIMARY KEY,
    log_id INTEGER REFERENCES logs(id),
    cause TEXT,
    solution TEXT,
    criticality INTEGER,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 6. Interface Basique ✅
- **Framework** : Streamlit simple
- **Pages** : Dashboard, Logs, Diagnostics
- **Temps réel** : Rafraîchissement auto
- **Filtres** : Par date, niveau, composant

**Pages MVP :**
- **Dashboard** : Métriques de base (erreurs/heure, top erreurs)
- **Logs** : Liste des logs avec filtres
- **Diagnostics** : Analyses IA avec solutions

### 7. Notification Email ✅
- **Protocole** : SMTP simple
- **Déclencheur** : Erreur critique détectée
- **Template** : Email HTML basique
- **Configuration** : Variables d'environnement

**Template email :**
```html
<h2>🚨 Erreur Critique Détectée</h2>
<p><strong>Timestamp:</strong> {timestamp}</p>
<p><strong>Composant:</strong> {component}</p>
<p><strong>Message:</strong> {message}</p>
<p><strong>Diagnostic IA:</strong> {diagnosis}</p>
<p><strong>Solution:</strong> {solution}</p>
```

## 🚫 Limitations MVP

### Fonctionnalités Exclues
- ❌ **Multi-sources** : Un seul fichier supporté
- ❌ **Recherche sémantique** : Pas de Qdrant
- ❌ **Multi-agents** : Un seul agent simplifié
- ❌ **Corrélation temporelle** : Pas d'analyse de patterns
- ❌ **Chat IA** : Interface basique uniquement
- ❌ **API REST** : Interface Streamlit seulement
- ❌ **Clustering** : Pas de regroupement d'erreurs
- ❌ **Métriques avancées** : Monitoring basique

### Contraintes Techniques
- **Volume** : Max 1000 logs/jour
- **Modèle** : Llama 3.1 7B uniquement
- **Stockage** : SQLite local seulement
- **Déploiement** : Local uniquement

## ✅ Critères d'Acceptation MVP

### Critères Fonctionnels
- [ ] **Ingestion** : Traite 100 logs/minute sans perte
- [ ] **Détection** : Identifie 90% des erreurs évidentes
- [ ] **Diagnostic** : Génère un diagnostic en < 2 minutes
- [ ] **Interface** : Dashboard fonctionnel avec données temps réel
- [ ] **Email** : Envoie alerte automatique pour erreurs critiques
- [ ] **Stockage** : Persiste tous les logs et diagnostics

### Critères Techniques
- [ ] **Performance** : Latence < 5 secondes par log
- [ ] **Fiabilité** : Fonctionne 8h continues sans crash
- [ ] **Installation** : Setup en < 30 minutes
- [ ] **Documentation** : Guide utilisateur complet

### Critères Qualité
- [ ] **Tests** : Coverage > 70%
- [ ] **Logs** : Logging structuré complet
- [ ] **Erreurs** : Gestion gracieuse des exceptions
- [ ] **Configuration** : Paramètres externalisés

## 🛠️ Architecture MVP Simplifiée

```
┌─────────────────┐
│   Log File      │
│  (local.log)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│  File Watcher   │
│   (watchdog)    │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│  Simple Parser  │
│   (regex)       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│ Error Detector  │
│  (patterns)     │
└─────────┬───────┘
          │
     [IF ERROR]
          │
          ▼
┌─────────────────┐
│  Single Agent   │
│   (Ollama)      │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐
│    SQLite       │
│   Storage       │
└─────────┬───────┘
          │
          ▼
┌─────────────────┐    ┌─────────────────┐
│  Streamlit UI   │    │  Email Alert    │
│   Dashboard     │    │   (SMTP)        │
└─────────────────┘    └─────────────────┘
```

## 📊 Métriques de Succès MVP

### Métriques Techniques
- **Uptime** : > 95%
- **Latence moyenne** : < 3 secondes
- **Précision détection** : > 85%
- **Faux positifs** : < 10%

### Métriques Utilisateur
- **Installation** : < 30 minutes
- **Première détection** : < 5 minutes
- **Satisfaction** : Score > 3/5
- **Utilisation** : > 1 semaine continue

## 🚀 Plan de Validation MVP

### Phase 1 : Tests Internes (Semaine 1)
1. **Tests unitaires** : Tous les modules
2. **Tests d'intégration** : Flux complet
3. **Tests de charge** : 100 logs/minute
4. **Tests de fiabilité** : 8h continues

### Phase 2 : Tests Utilisateur (Semaine 2)
1. **Installation** : 3 utilisateurs différents
2. **Utilisation** : 1 semaine en conditions réelles
3. **Feedback** : Questionnaire détaillé
4. **Métriques** : Collecte automatique

### Phase 3 : Validation Métier (Semaine 3)
1. **Démonstration** : Présentation stakeholders
2. **ROI** : Calcul temps économisé
3. **Roadmap** : Validation prochaines étapes
4. **Go/No-Go** : Décision continuation

## 📝 Livrables MVP

### Code Source
- [ ] Module collector basique
- [ ] Parser regex simple
- [ ] Agent IA unique
- [ ] Interface Streamlit
- [ ] Système notification

### Documentation
- [ ] Guide d'installation
- [ ] Manuel utilisateur
- [ ] Documentation technique
- [ ] Tests et validation

### Configuration
- [ ] requirements.txt
- [ ] config.yaml
- [ ] Scripts de déploiement
- [ ] Variables d'environnement

---

**🎯 Objectif MVP : Démontrer la valeur en 4 semaines avec un système simple mais fonctionnel.**

*Dernière mise à jour : 07/01/2025*