# Sprint 2 : Parsing et Détection - Plan d'Action Détaillé

## 📋 Vue d'Ensemble

**Durée** : 2 semaines  
**Objectif** : Développer les capacités d'ingestion, parsing et détection d'erreurs  
**Équipe** : 1-2 développeurs  
**Priorité** : Critique  
**Prérequis** : Sprint 1 terminé (fondations)

## 🎯 Objectifs du Sprint

### Objectifs Principaux
1. **Collecteur de logs** : Ingestion temps réel de fichiers de logs
2. **Parser intelligent** : Extraction structurée des informations
3. **Détecteur d'erreurs** : Identification automatique des anomalies
4. **Normalisation** : Format uniforme pour tous les logs
5. **Pipeline de traitement** : Flux de données optimisé

### Objectifs Secondaires
- Support multi-formats de logs
- Filtrage et déduplication
- Métriques de performance
- Monitoring du pipeline

## 📦 Livrables

### Code Source
- [ ] Module collector avec watchdog
- [ ] Parser multi-formats (regex + patterns)
- [ ] Détecteur d'erreurs configurable
- [ ] Pipeline de traitement asynchrone
- [ ] Tests unitaires et d'intégration

### Documentation
- [ ] Guide configuration des parsers
- [ ] Documentation patterns d'erreurs
- [ ] Guide de troubleshooting
- [ ] Métriques et monitoring

### Configuration
- [ ] Patterns de parsing configurables
- [ ] Règles de détection d'erreurs
- [ ] Configuration des sources
- [ ] Paramètres de performance

## 🗓️ Planning Détaillé

### Semaine 1 : Ingestion et Parsing

#### Jour 1-2 : Collecteur de Logs
**Tâches :**
- [ ] Implémenter FileWatcher avec watchdog
- [ ] Créer système de rotation de logs
- [ ] Gérer les reconnexions automatiques
- [ ] Implémenter buffer circulaire

**Architecture Collecteur :**
```python
# src/collectors/file_collector.py
import asyncio
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler
from typing import AsyncGenerator, Dict, Any

class LogFileCollector(FileSystemEventHandler):
    """Collecteur de logs en temps réel"""
    
    def __init__(self, file_path: str, buffer_size: int = 1000):
        self.file_path = file_path
        self.buffer = asyncio.Queue(maxsize=buffer_size)
        self.observer = Observer()
        self.file_position = 0
        
    async def start(self) -> None:
        """Démarre la surveillance du fichier"""
        # Lire les logs existants
        await self._read_existing_logs()
        
        # Surveiller les nouveaux logs
        self.observer.schedule(self, path=os.path.dirname(self.file_path))
        self.observer.start()
        
    async def collect(self) -> AsyncGenerator[str, None]:
        """Générateur asynchrone de logs"""
        while True:
            try:
                log_line = await asyncio.wait_for(
                    self.buffer.get(), timeout=1.0
                )
                yield log_line
            except asyncio.TimeoutError:
                continue
                
    def on_modified(self, event):
        """Callback watchdog pour nouveaux logs"""
        if event.src_path == self.file_path:
            asyncio.create_task(self._read_new_logs())
```

**Critères d'acceptation :**
- Surveillance temps réel fonctionnelle
- Gestion rotation de logs
- Buffer circulaire sans perte
- Performance > 1000 logs/minute

#### Jour 3-4 : Parser Multi-Formats
**Tâches :**
- [ ] Créer parser générique avec regex
- [ ] Implémenter parsers spécialisés (Apache, Nginx, JSON)
- [ ] Système de détection automatique de format
- [ ] Validation et normalisation des données

**Parsers Spécialisés :**
```python
# src/parsers/log_parser.py
import re
import json
from abc import ABC, abstractmethod
from datetime import datetime
from typing import Dict, Any, Optional

class BaseLogParser(ABC):
    """Classe de base pour tous les parsers"""
    
    @abstractmethod
    def parse(self, raw_log: str) -> Optional[Dict[str, Any]]:
        pass
    
    @abstractmethod
    def can_parse(self, raw_log: str) -> bool:
        pass

class ApacheLogParser(BaseLogParser):
    """Parser pour logs Apache/Nginx"""
    
    APACHE_PATTERN = re.compile(
        r'(?P<ip>\d+\.\d+\.\d+\.\d+) - - '
        r'\[(?P<timestamp>[^\]]+)\] '
        r'"(?P<method>\w+) (?P<path>[^\s]+) HTTP/[\d\.]+" '
        r'(?P<status>\d+) (?P<size>\d+|-)'
    )
    
    def parse(self, raw_log: str) -> Optional[Dict[str, Any]]:
        match = self.APACHE_PATTERN.match(raw_log.strip())
        if not match:
            return None
            
        return {
            'timestamp': self._parse_timestamp(match.group('timestamp')),
            'level': self._determine_level(int(match.group('status'))),
            'component': 'web_server',
            'message': f"{match.group('method')} {match.group('path')}",
            'metadata': {
                'ip': match.group('ip'),
                'method': match.group('method'),
                'path': match.group('path'),
                'status': int(match.group('status')),
                'size': match.group('size')
            },
            'raw_log': raw_log
        }

class JSONLogParser(BaseLogParser):
    """Parser pour logs JSON structurés"""
    
    def parse(self, raw_log: str) -> Optional[Dict[str, Any]]:
        try:
            data = json.loads(raw_log.strip())
            return {
                'timestamp': self._parse_timestamp(data.get('timestamp')),
                'level': data.get('level', 'INFO').upper(),
                'component': data.get('component', 'unknown'),
                'message': data.get('message', ''),
                'metadata': {k: v for k, v in data.items() 
                           if k not in ['timestamp', 'level', 'component', 'message']},
                'raw_log': raw_log
            }
        except json.JSONDecodeError:
            return None

class ApplicationLogParser(BaseLogParser):
    """Parser pour logs applicatifs génériques"""
    
    APP_PATTERN = re.compile(
        r'(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}[,\.]\d{3}) '
        r'\[(?P<level>\w+)\] '
        r'(?P<component>[^\s]+) - '
        r'(?P<message>.*)'
    )
    
    def parse(self, raw_log: str) -> Optional[Dict[str, Any]]:
        match = self.APP_PATTERN.match(raw_log.strip())
        if not match:
            return None
            
        return {
            'timestamp': self._parse_timestamp(match.group('timestamp')),
            'level': match.group('level').upper(),
            'component': match.group('component'),
            'message': match.group('message'),
            'metadata': {},
            'raw_log': raw_log
        }
```

**Critères d'acceptation :**
- Support 3+ formats de logs
- Détection automatique de format
- Normalisation uniforme
- Validation des données

#### Jour 5 : Tests et Intégration
**Tâches :**
- [ ] Tests unitaires pour chaque parser
- [ ] Tests d'intégration collecteur + parser
- [ ] Tests de performance avec gros volumes
- [ ] Validation formats de sortie

### Semaine 2 : Détection et Pipeline

#### Jour 6-7 : Détecteur d'Erreurs
**Tâches :**
- [ ] Créer moteur de règles configurable
- [ ] Implémenter patterns d'erreurs par catégorie
- [ ] Système de scoring et priorités
- [ ] Déduplication d'erreurs similaires

**Détecteur d'Erreurs :**
```python
# src/detectors/error_detector.py
import re
from typing import Dict, List, Any, Optional
from dataclasses import dataclass
from enum import Enum

class ErrorSeverity(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4

@dataclass
class ErrorPattern:
    name: str
    pattern: str
    severity: ErrorSeverity
    category: str
    description: str

class ErrorDetector:
    """Détecteur d'erreurs configurable"""
    
    def __init__(self):
        self.patterns = self._load_error_patterns()
        self.recent_errors = {}  # Pour déduplication
        
    def _load_error_patterns(self) -> List[ErrorPattern]:
        """Charge les patterns d'erreurs depuis la configuration"""
        return [
            # Erreurs de base de données
            ErrorPattern(
                name="db_connection_timeout",
                pattern=r"connection.*timeout|database.*timeout",
                severity=ErrorSeverity.HIGH,
                category="database",
                description="Timeout de connexion base de données"
            ),
            ErrorPattern(
                name="db_connection_refused",
                pattern=r"connection.*refused|connection.*failed",
                severity=ErrorSeverity.CRITICAL,
                category="database",
                description="Connexion base de données refusée"
            ),
            
            # Erreurs réseau
            ErrorPattern(
                name="http_5xx_error",
                pattern=r"HTTP/[\d\.]+ 5\d{2}|status.*5\d{2}",
                severity=ErrorSeverity.HIGH,
                category="network",
                description="Erreur serveur HTTP 5xx"
            ),
            ErrorPattern(
                name="api_timeout",
                pattern=r"api.*timeout|request.*timeout",
                severity=ErrorSeverity.MEDIUM,
                category="network",
                description="Timeout d'API externe"
            ),
            
            # Erreurs applicatives
            ErrorPattern(
                name="null_pointer",
                pattern=r"null.*pointer|nullpointerexception",
                severity=ErrorSeverity.MEDIUM,
                category="application",
                description="Erreur de pointeur null"
            ),
            ErrorPattern(
                name="out_of_memory",
                pattern=r"out.*of.*memory|outofmemoryerror",
                severity=ErrorSeverity.CRITICAL,
                category="infrastructure",
                description="Mémoire insuffisante"
            ),
            
            # Erreurs de sécurité
            ErrorPattern(
                name="auth_failed",
                pattern=r"authentication.*failed|unauthorized",
                severity=ErrorSeverity.HIGH,
                category="security",
                description="Échec d'authentification"
            ),
            ErrorPattern(
                name="access_denied",
                pattern=r"access.*denied|forbidden",
                severity=ErrorSeverity.MEDIUM,
                category="security",
                description="Accès refusé"
            )
        ]
    
    def detect_errors(self, parsed_log: Dict[str, Any]) -> List[Dict[str, Any]]:
        """Détecte les erreurs dans un log parsé"""
        detected_errors = []
        message = parsed_log.get('message', '').lower()
        level = parsed_log.get('level', 'INFO')
        
        # Vérification niveau de log
        if level in ['ERROR', 'FATAL', 'CRITICAL']:
            # Recherche de patterns spécifiques
            for pattern in self.patterns:
                if re.search(pattern.pattern, message, re.IGNORECASE):
                    error_info = {
                        'pattern_name': pattern.name,
                        'category': pattern.category,
                        'severity': pattern.severity.value,
                        'description': pattern.description,
                        'matched_text': message,
                        'log_data': parsed_log,
                        'detected_at': datetime.utcnow().isoformat()
                    }
                    
                    # Déduplication
                    if not self._is_duplicate(error_info):
                        detected_errors.append(error_info)
                        self._record_error(error_info)
        
        return detected_errors
    
    def _is_duplicate(self, error_info: Dict[str, Any]) -> bool:
        """Vérifie si l'erreur est un doublon récent"""
        key = f"{error_info['pattern_name']}_{error_info['category']}"
        last_seen = self.recent_errors.get(key)
        
        if last_seen:
            time_diff = datetime.utcnow() - last_seen
            return time_diff.total_seconds() < 300  # 5 minutes
        
        return False
    
    def _record_error(self, error_info: Dict[str, Any]) -> None:
        """Enregistre l'erreur pour déduplication"""
        key = f"{error_info['pattern_name']}_{error_info['category']}"
        self.recent_errors[key] = datetime.utcnow()
```

**Critères d'acceptation :**
- Détection > 90% des erreurs évidentes
- Déduplication fonctionnelle
- Scoring par sévérité
- Configuration flexible

#### Jour 8-9 : Pipeline de Traitement
**Tâches :**
- [ ] Créer pipeline asynchrone complet
- [ ] Implémenter système de queues
- [ ] Gestion des erreurs et retry
- [ ] Monitoring et métriques

**Pipeline Asynchrone :**
```python
# src/pipeline/log_pipeline.py
import asyncio
from typing import AsyncGenerator, Dict, Any, List
from dataclasses import dataclass
import logging

@dataclass
class PipelineMetrics:
    logs_processed: int = 0
    errors_detected: int = 0
    processing_time: float = 0.0
    failed_logs: int = 0

class LogProcessingPipeline:
    """Pipeline de traitement des logs"""
    
    def __init__(self, collector, parser, detector, storage):
        self.collector = collector
        self.parser = parser
        self.detector = detector
        self.storage = storage
        self.metrics = PipelineMetrics()
        self.running = False
        
    async def start(self) -> None:
        """Démarre le pipeline de traitement"""
        self.running = True
        logging.info("Démarrage du pipeline de traitement des logs")
        
        # Démarrer le collecteur
        await self.collector.start()
        
        # Traitement en continu
        async for raw_log in self.collector.collect():
            if not self.running:
                break
                
            await self._process_log(raw_log)
    
    async def _process_log(self, raw_log: str) -> None:
        """Traite un log individuel"""
        start_time = asyncio.get_event_loop().time()
        
        try:
            # 1. Parsing
            parsed_log = await self._parse_log(raw_log)
            if not parsed_log:
                self.metrics.failed_logs += 1
                return
            
            # 2. Stockage du log
            await self.storage.store_log(parsed_log)
            
            # 3. Détection d'erreurs
            errors = self.detector.detect_errors(parsed_log)
            
            # 4. Traitement des erreurs détectées
            for error in errors:
                await self._handle_detected_error(error)
                self.metrics.errors_detected += 1
            
            # 5. Mise à jour métriques
            self.metrics.logs_processed += 1
            processing_time = asyncio.get_event_loop().time() - start_time
            self.metrics.processing_time += processing_time
            
        except Exception as e:
            logging.error(f"Erreur traitement log: {e}")
            self.metrics.failed_logs += 1
    
    async def _parse_log(self, raw_log: str) -> Dict[str, Any]:
        """Parse un log brut"""
        return await asyncio.get_event_loop().run_in_executor(
            None, self.parser.parse, raw_log
        )
    
    async def _handle_detected_error(self, error: Dict[str, Any]) -> None:
        """Traite une erreur détectée"""
        # Stockage de l'erreur
        await self.storage.store_error(error)
        
        # Notification si critique
        if error['severity'] >= 3:
            await self._send_alert(error)
    
    async def _send_alert(self, error: Dict[str, Any]) -> None:
        """Envoie une alerte pour erreur critique"""
        # TODO: Implémenter système de notification
        logging.warning(f"Erreur critique détectée: {error['description']}")
    
    async def stop(self) -> None:
        """Arrête le pipeline"""
        self.running = False
        await self.collector.stop()
        logging.info("Pipeline arrêté")
    
    def get_metrics(self) -> Dict[str, Any]:
        """Retourne les métriques du pipeline"""
        avg_processing_time = (
            self.metrics.processing_time / max(self.metrics.logs_processed, 1)
        )
        
        return {
            'logs_processed': self.metrics.logs_processed,
            'errors_detected': self.metrics.errors_detected,
            'failed_logs': self.metrics.failed_logs,
            'avg_processing_time': avg_processing_time,
            'error_rate': self.metrics.errors_detected / max(self.metrics.logs_processed, 1)
        }
```

**Critères d'acceptation :**
- Pipeline asynchrone fonctionnel
- Gestion d'erreurs robuste
- Métriques en temps réel
- Performance > 500 logs/minute

#### Jour 10 : Tests et Optimisation
**Tâches :**
- [ ] Tests de charge avec gros volumes
- [ ] Optimisation des performances
- [ ] Tests d'intégration complète
- [ ] Documentation technique

## 🔧 Configuration Technique

### Configuration des Sources
```yaml
# config/sources.yaml
log_sources:
  - name: "application_logs"
    type: "file"
    path: "/var/log/app/application.log"
    format: "application"
    enabled: true
    
  - name: "nginx_access"
    type: "file"
    path: "/var/log/nginx/access.log"
    format: "apache"
    enabled: true
    
  - name: "json_logs"
    type: "file"
    path: "/var/log/app/structured.log"
    format: "json"
    enabled: true

parsing:
  buffer_size: 1000
  batch_size: 50
  timeout: 30
  retry_attempts: 3
```

### Patterns d'Erreurs
```yaml
# config/error_patterns.yaml
error_patterns:
  database:
    - name: "connection_timeout"
      pattern: "connection.*timeout|database.*timeout"
      severity: "high"
      
    - name: "deadlock"
      pattern: "deadlock.*detected|lock.*timeout"
      severity: "medium"
      
  network:
    - name: "http_5xx"
      pattern: "HTTP/[\d\.]+ 5\d{2}|status.*5\d{2}"
      severity: "high"
      
    - name: "api_timeout"
      pattern: "api.*timeout|request.*timeout"
      severity: "medium"

detection:
  deduplication_window: 300  # 5 minutes
  min_severity: 1
  max_errors_per_minute: 100
```

## 📊 Métriques de Succès

### Métriques Techniques
- **Débit** : > 500 logs/minute traités
- **Latence** : < 100ms par log
- **Précision détection** : > 90%
- **Faux positifs** : < 5%

### Métriques Qualité
- **Coverage tests** : > 85%
- **Performance** : Stable sur 8h
- **Mémoire** : < 200MB utilisation
- **CPU** : < 20% utilisation moyenne

### Métriques Fonctionnelles
- **Formats supportés** : 3+ types de logs
- **Patterns d'erreurs** : 20+ patterns configurés
- **Déduplication** : Fonctionnelle sur 5 minutes
- **Pipeline** : Traitement temps réel stable

## 🚨 Risques et Mitigation

### Risques Techniques
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Performance parsing | Moyenne | Élevé | Optimisation regex + tests charge |
| Perte de logs | Faible | Critique | Buffer persistant + monitoring |
| Faux positifs | Élevée | Moyen | Fine-tuning patterns + validation |

### Risques Fonctionnels
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Formats non supportés | Moyenne | Moyen | Parser générique + extensibilité |
| Volume trop élevé | Faible | Élevé | Système de queues + scaling |

## 📝 Checklist de Fin de Sprint

### Code
- [ ] Collecteur temps réel fonctionnel
- [ ] Parsers multi-formats implémentés
- [ ] Détecteur d'erreurs configurable
- [ ] Pipeline asynchrone opérationnel
- [ ] Tests unitaires et d'intégration

### Performance
- [ ] Débit > 500 logs/minute
- [ ] Latence < 100ms par log
- [ ] Stabilité sur 8h continues
- [ ] Utilisation ressources optimisée

### Configuration
- [ ] Patterns d'erreurs configurables
- [ ] Sources multiples supportées
- [ ] Paramètres de performance ajustables
- [ ] Monitoring et métriques

### Documentation
- [ ] Guide configuration parsers
- [ ] Documentation patterns d'erreurs
- [ ] Guide troubleshooting
- [ ] Métriques et alertes

## 🎯 Critères de Passage Sprint 3

Pour passer au Sprint 3, tous ces critères doivent être validés :

1. **Ingestion** : Collecteur temps réel stable et performant
2. **Parsing** : Support multi-formats avec normalisation
3. **Détection** : Identification automatique d'erreurs > 90%
4. **Pipeline** : Traitement asynchrone fonctionnel
5. **Tests** : Coverage > 85% et tests de charge OK

---

**🚀 Objectif : Pipeline de traitement des logs robuste et performant**

*Dernière mise à jour : 07/01/2025*