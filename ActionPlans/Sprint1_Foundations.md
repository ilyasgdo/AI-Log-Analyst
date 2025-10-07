# Sprint 1 : Foundations - Plan d'Action Détaillé

## 📋 Vue d'Ensemble

**Durée** : 2 semaines  
**Objectif** : Établir les fondations solides du projet AI Ops Copilot  
**Équipe** : 1-2 développeurs  
**Priorité** : Critique

## 🎯 Objectifs du Sprint

### Objectifs Principaux
1. **Infrastructure de base** : Configuration environnement de développement
2. **Architecture modulaire** : Structure de projet extensible
3. **Logging centralisé** : Système de logs structurés
4. **Configuration flexible** : Gestion des paramètres externalisés
5. **Tests automatisés** : Framework de tests unitaires

### Objectifs Secondaires
- Documentation technique de base
- Scripts d'installation automatisés
- Monitoring basique des performances

## 📦 Livrables

### Code Source
- [ ] Structure de projet modulaire
- [ ] Système de configuration centralisé
- [ ] Framework de logging structuré
- [ ] Tests unitaires de base
- [ ] Scripts d'installation

### Documentation
- [ ] Guide d'installation développeur
- [ ] Documentation architecture
- [ ] Standards de codage
- [ ] Guide de contribution

### Configuration
- [ ] requirements.txt complet
- [ ] Configuration Docker
- [ ] Variables d'environnement
- [ ] Scripts de déploiement

## 🗓️ Planning Détaillé

### Semaine 1 : Infrastructure

#### Jour 1-2 : Setup Projet
**Tâches :**
- [ ] Initialiser repository Git avec structure modulaire
- [ ] Configurer environnement virtuel Python 3.11+
- [ ] Installer et configurer Ollama localement
- [ ] Créer structure de dossiers standardisée

**Structure de dossiers :**
```
ai-ops-copilot/
├── src/
│   ├── core/           # Modules core
│   ├── agents/         # Agents IA
│   ├── collectors/     # Collecteurs de logs
│   ├── parsers/        # Parseurs de logs
│   ├── storage/        # Couche de stockage
│   └── interfaces/     # Interfaces utilisateur
├── config/             # Configurations
├── tests/              # Tests automatisés
├── docs/               # Documentation
├── scripts/            # Scripts utilitaires
└── data/               # Données de test
```

**Critères d'acceptation :**
- Repository Git initialisé avec .gitignore Python
- Environnement virtuel fonctionnel
- Ollama installé et modèle Llama 3.1 téléchargé
- Structure de dossiers créée

#### Jour 3-4 : Configuration Système
**Tâches :**
- [ ] Créer système de configuration centralisé (config.yaml)
- [ ] Implémenter chargement variables d'environnement
- [ ] Configurer logging structuré avec Loguru
- [ ] Créer utilitaires de base (helpers, constants)

**Configuration type :**
```yaml
# config/config.yaml
app:
  name: "AI Ops Copilot"
  version: "0.1.0"
  debug: true

logging:
  level: "INFO"
  format: "json"
  file: "logs/app.log"
  rotation: "1 day"
  retention: "30 days"

ollama:
  host: "localhost"
  port: 11434
  model: "llama3.1:7b"
  timeout: 120

storage:
  type: "sqlite"
  path: "data/logs.db"
  pool_size: 10
```

**Critères d'acceptation :**
- Configuration YAML chargée dynamiquement
- Variables d'environnement supportées
- Logging structuré fonctionnel
- Utilitaires de base testés

#### Jour 5 : Tests et Qualité
**Tâches :**
- [ ] Configurer pytest avec coverage
- [ ] Créer tests unitaires pour modules de base
- [ ] Configurer pre-commit hooks
- [ ] Implémenter linting (black, flake8, mypy)

**Configuration pytest :**
```ini
# pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    --cov=src
    --cov-report=html
    --cov-report=term-missing
    --strict-markers
```

**Critères d'acceptation :**
- Tests unitaires > 80% coverage
- Pre-commit hooks fonctionnels
- Code formaté automatiquement
- Type checking activé

### Semaine 2 : Modules Core

#### Jour 6-7 : Module Core
**Tâches :**
- [ ] Créer classes de base abstraites
- [ ] Implémenter gestionnaire d'événements
- [ ] Créer système de plugins
- [ ] Développer utilitaires de validation

**Classes de base :**
```python
# src/core/base.py
from abc import ABC, abstractmethod
from typing import Any, Dict, Optional

class BaseCollector(ABC):
    """Classe de base pour tous les collecteurs"""
    
    @abstractmethod
    async def collect(self) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    async def start(self) -> None:
        pass
    
    @abstractmethod
    async def stop(self) -> None:
        pass

class BaseParser(ABC):
    """Classe de base pour tous les parseurs"""
    
    @abstractmethod
    def parse(self, raw_log: str) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def validate(self, parsed_log: Dict[str, Any]) -> bool:
        pass
```

**Critères d'acceptation :**
- Classes abstraites définies
- Système d'événements fonctionnel
- Architecture plugin implémentée
- Validation des données active

#### Jour 8-9 : Storage Layer
**Tâches :**
- [ ] Implémenter couche d'abstraction base de données
- [ ] Créer modèles SQLAlchemy
- [ ] Configurer migrations Alembic
- [ ] Développer repository pattern

**Modèles de données :**
```python
# src/storage/models.py
from sqlalchemy import Column, Integer, String, DateTime, Text, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

class LogEntry(Base):
    __tablename__ = 'log_entries'
    
    id = Column(Integer, primary_key=True)
    timestamp = Column(DateTime, nullable=False)
    level = Column(String(10), nullable=False)
    component = Column(String(100))
    message = Column(Text, nullable=False)
    raw_log = Column(Text, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relations
    diagnostics = relationship("Diagnostic", back_populates="log_entry")

class Diagnostic(Base):
    __tablename__ = 'diagnostics'
    
    id = Column(Integer, primary_key=True)
    log_entry_id = Column(Integer, ForeignKey('log_entries.id'))
    agent_type = Column(String(50), nullable=False)
    diagnosis = Column(Text)
    confidence = Column(Integer)  # 1-100
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relations
    log_entry = relationship("LogEntry", back_populates="diagnostics")
```

**Critères d'acceptation :**
- Base de données SQLite configurée
- Modèles SQLAlchemy fonctionnels
- Migrations Alembic opérationnelles
- Repository pattern implémenté

#### Jour 10 : Intégration et Tests
**Tâches :**
- [ ] Tests d'intégration modules core
- [ ] Validation performance de base
- [ ] Documentation technique
- [ ] Scripts de déploiement

**Tests d'intégration :**
```python
# tests/integration/test_core_integration.py
import pytest
from src.core.config import ConfigManager
from src.storage.database import DatabaseManager

@pytest.mark.asyncio
async def test_full_stack_integration():
    """Test intégration complète des modules core"""
    # Setup
    config = ConfigManager()
    db = DatabaseManager(config)
    
    # Test connexion base
    await db.connect()
    assert db.is_connected()
    
    # Test création tables
    await db.create_tables()
    
    # Cleanup
    await db.disconnect()
```

**Critères d'acceptation :**
- Tests d'intégration passent
- Performance baseline établie
- Documentation à jour
- Scripts de déploiement testés

## 🔧 Configuration Technique

### Environnement de Développement
```bash
# Installation
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate     # Windows

pip install -r requirements-dev.txt

# Configuration Ollama
ollama pull llama3.1:7b
ollama serve
```

### Variables d'Environnement
```bash
# .env
APP_ENV=development
LOG_LEVEL=DEBUG
DATABASE_URL=sqlite:///data/logs.db
OLLAMA_HOST=localhost
OLLAMA_PORT=11434
OLLAMA_MODEL=llama3.1:7b
```

### Docker Configuration
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY src/ src/
COPY config/ config/

CMD ["python", "-m", "src.main"]
```

## 📊 Métriques de Succès

### Métriques Techniques
- **Coverage tests** : > 80%
- **Performance** : Startup < 5 secondes
- **Mémoire** : < 100MB au démarrage
- **Logs** : Structured logging fonctionnel

### Métriques Qualité
- **Linting** : 0 erreurs flake8
- **Type checking** : 0 erreurs mypy
- **Documentation** : Tous modules documentés
- **Tests** : Tous modules testés

### Métriques Fonctionnelles
- **Configuration** : Chargement dynamique OK
- **Base de données** : Connexion et migrations OK
- **Logging** : Rotation et rétention OK
- **Plugins** : Architecture extensible OK

## 🚨 Risques et Mitigation

### Risques Techniques
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Problème installation Ollama | Moyenne | Élevé | Documentation détaillée + scripts automatisés |
| Performance SQLite | Faible | Moyen | Tests de charge + optimisation requêtes |
| Complexité architecture | Moyenne | Moyen | Revue code + simplification si nécessaire |

### Risques Projet
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Retard planning | Moyenne | Moyen | Buffer temps + priorisation features |
| Scope creep | Élevée | Élevé | Définition claire MVP + revues régulières |

## 📝 Checklist de Fin de Sprint

### Code
- [ ] Tous les modules core implémentés
- [ ] Tests unitaires > 80% coverage
- [ ] Tests d'intégration passent
- [ ] Code review effectuée
- [ ] Documentation technique complète

### Infrastructure
- [ ] Environnement de développement documenté
- [ ] Scripts d'installation testés
- [ ] Configuration Docker fonctionnelle
- [ ] Base de données initialisée

### Qualité
- [ ] Linting et formatting configurés
- [ ] Pre-commit hooks actifs
- [ ] Type checking activé
- [ ] Monitoring basique en place

### Documentation
- [ ] Guide d'installation à jour
- [ ] Architecture documentée
- [ ] Standards de codage définis
- [ ] README projet complet

## 🎯 Critères de Passage Sprint 2

Pour passer au Sprint 2, tous ces critères doivent être validés :

1. **Infrastructure** : Environnement de développement stable
2. **Architecture** : Modules core fonctionnels et testés
3. **Qualité** : Standards de code respectés
4. **Documentation** : Guide développeur complet
5. **Tests** : Coverage > 80% et tests d'intégration OK

---

**🚀 Objectif : Fondations solides pour accélérer les sprints suivants**

*Dernière mise à jour : 07/01/2025*