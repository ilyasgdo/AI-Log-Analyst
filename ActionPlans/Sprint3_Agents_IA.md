# Sprint 3 : Agents IA - Plan d'Action Détaillé

## 📋 Vue d'Ensemble

**Durée** : 2 semaines  
**Objectif** : Implémenter le système multi-agents IA avec spécialisation par domaine  
**Équipe** : 1-2 développeurs  
**Priorité** : Critique  
**Prérequis** : Sprint 2 terminé (parsing et détection)

## 🎯 Objectifs du Sprint

### Objectifs Principaux
1. **Orchestrateur multi-agents** : Système de routage intelligent
2. **Agents spécialisés** : 6 agents Fixer par domaine d'expertise
3. **Intégration Ollama** : Communication avec LLM local
4. **Système de prompts** : Templates optimisés par agent
5. **Pipeline IA** : Flux de diagnostic automatisé

### Objectifs Secondaires
- Cache des réponses IA
- Système de fallback
- Métriques de performance IA
- Validation des solutions

## 📦 Livrables

### Code Source
- [ ] Orchestrateur principal avec routage
- [ ] 6 agents Fixer spécialisés
- [ ] Intégration Ollama robuste
- [ ] Système de prompts configurables
- [ ] Pipeline IA complet

### Documentation
- [ ] Guide configuration agents
- [ ] Documentation prompts
- [ ] Guide troubleshooting IA
- [ ] Métriques et monitoring

### Configuration
- [ ] Templates de prompts par agent
- [ ] Configuration Ollama
- [ ] Paramètres de performance
- [ ] Règles de routage

## 🗓️ Planning Détaillé

### Semaine 1 : Infrastructure IA

#### Jour 1-2 : Intégration Ollama
**Tâches :**
- [ ] Créer client Ollama asynchrone
- [ ] Implémenter gestion des timeouts
- [ ] Système de retry et fallback
- [ ] Pool de connexions

**Client Ollama :**
```python
# src/ai/ollama_client.py
import asyncio
import aiohttp
import json
from typing import Dict, Any, Optional, List
from dataclasses import dataclass
import logging

@dataclass
class OllamaConfig:
    host: str = "localhost"
    port: int = 11434
    model: str = "llama3.1:7b"
    timeout: int = 120
    max_retries: int = 3
    temperature: float = 0.1

class OllamaClient:
    """Client asynchrone pour Ollama"""
    
    def __init__(self, config: OllamaConfig):
        self.config = config
        self.base_url = f"http://{config.host}:{config.port}"
        self.session = None
        
    async def __aenter__(self):
        self.session = aiohttp.ClientSession(
            timeout=aiohttp.ClientTimeout(total=self.config.timeout)
        )
        return self
        
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()
    
    async def generate(self, prompt: str, system_prompt: str = None) -> Dict[str, Any]:
        """Génère une réponse avec le modèle"""
        payload = {
            "model": self.config.model,
            "prompt": prompt,
            "stream": False,
            "options": {
                "temperature": self.config.temperature,
                "top_p": 0.9,
                "top_k": 40
            }
        }
        
        if system_prompt:
            payload["system"] = system_prompt
        
        for attempt in range(self.config.max_retries):
            try:
                async with self.session.post(
                    f"{self.base_url}/api/generate",
                    json=payload
                ) as response:
                    if response.status == 200:
                        result = await response.json()
                        return {
                            "response": result.get("response", ""),
                            "model": result.get("model", ""),
                            "created_at": result.get("created_at", ""),
                            "done": result.get("done", False),
                            "total_duration": result.get("total_duration", 0),
                            "load_duration": result.get("load_duration", 0),
                            "prompt_eval_count": result.get("prompt_eval_count", 0),
                            "eval_count": result.get("eval_count", 0)
                        }
                    else:
                        logging.error(f"Ollama error: {response.status}")
                        
            except asyncio.TimeoutError:
                logging.warning(f"Ollama timeout, attempt {attempt + 1}")
                if attempt == self.config.max_retries - 1:
                    raise
                await asyncio.sleep(2 ** attempt)  # Exponential backoff
                
            except Exception as e:
                logging.error(f"Ollama error: {e}")
                if attempt == self.config.max_retries - 1:
                    raise
                await asyncio.sleep(1)
        
        raise Exception("Max retries exceeded for Ollama")
    
    async def health_check(self) -> bool:
        """Vérifie la santé du service Ollama"""
        try:
            async with self.session.get(f"{self.base_url}/api/tags") as response:
                return response.status == 200
        except:
            return False
```

**Critères d'acceptation :**
- Connexion Ollama stable
- Gestion timeouts et retry
- Pool de connexions fonctionnel
- Health check automatique

#### Jour 3-4 : Orchestrateur Multi-Agents
**Tâches :**
- [ ] Créer orchestrateur principal
- [ ] Implémenter classificateur d'erreurs
- [ ] Système de routage intelligent
- [ ] Gestion des priorités

**Orchestrateur :**
```python
# src/agents/orchestrator.py
import asyncio
from typing import Dict, Any, List, Optional
from enum import Enum
import logging

class AgentType(Enum):
    DATABASE = "database"
    NETWORK = "network"
    APPLICATION = "application"
    SECURITY = "security"
    INFRASTRUCTURE = "infrastructure"
    INTEGRATION = "integration"

class ErrorClassifier:
    """Classifie les erreurs pour routage vers le bon agent"""
    
    def __init__(self):
        self.classification_rules = {
            AgentType.DATABASE: [
                r'database.*error|db.*error|sql.*error',
                r'connection.*timeout.*database|connection.*refused.*database',
                r'deadlock|lock.*timeout|transaction.*failed',
                r'table.*not.*found|column.*not.*found',
                r'duplicate.*key|foreign.*key.*constraint'
            ],
            AgentType.NETWORK: [
                r'http.*error|api.*error|rest.*error',
                r'connection.*timeout|connection.*refused',
                r'ssl.*error|certificate.*error|tls.*error',
                r'dns.*error|network.*error',
                r'rate.*limit|gateway.*timeout'
            ],
            AgentType.SECURITY: [
                r'authentication.*failed|auth.*failed',
                r'authorization.*denied|access.*denied',
                r'token.*expired|session.*expired',
                r'csrf.*error|xss.*detected',
                r'unauthorized|forbidden'
            ],
            AgentType.INFRASTRUCTURE: [
                r'out.*of.*memory|memory.*error',
                r'disk.*full|disk.*space|storage.*error',
                r'cpu.*high|cpu.*usage|performance.*issue',
                r'container.*failed|docker.*error',
                r'service.*unavailable|server.*error'
            ],
            AgentType.APPLICATION: [
                r'null.*pointer|nullpointerexception',
                r'array.*index|indexoutofbounds',
                r'validation.*error|validation.*failed',
                r'business.*logic|application.*error',
                r'configuration.*error|config.*error'
            ],
            AgentType.INTEGRATION: [
                r'third.*party.*api|external.*api',
                r'webhook.*failed|callback.*error',
                r'message.*queue|queue.*error',
                r'sync.*failed|synchronization.*error',
                r'microservice.*error|service.*mesh'
            ]
        }
    
    def classify(self, error_data: Dict[str, Any]) -> AgentType:
        """Classifie l'erreur et retourne le type d'agent approprié"""
        message = error_data.get('message', '').lower()
        component = error_data.get('component', '').lower()
        category = error_data.get('category', '').lower()
        
        # Recherche par patterns
        scores = {}
        for agent_type, patterns in self.classification_rules.items():
            score = 0
            for pattern in patterns:
                if re.search(pattern, message, re.IGNORECASE):
                    score += 2
                if re.search(pattern, component, re.IGNORECASE):
                    score += 1
                if re.search(pattern, category, re.IGNORECASE):
                    score += 1
            
            if score > 0:
                scores[agent_type] = score
        
        # Retourne l'agent avec le meilleur score
        if scores:
            return max(scores, key=scores.get)
        
        # Fallback par défaut
        return AgentType.APPLICATION

class AgentOrchestrator:
    """Orchestrateur principal des agents IA"""
    
    def __init__(self, ollama_client):
        self.ollama_client = ollama_client
        self.classifier = ErrorClassifier()
        self.agents = {}
        self.metrics = {
            'total_requests': 0,
            'successful_diagnoses': 0,
            'failed_diagnoses': 0,
            'avg_response_time': 0.0
        }
        
    def register_agent(self, agent_type: AgentType, agent):
        """Enregistre un agent spécialisé"""
        self.agents[agent_type] = agent
        logging.info(f"Agent {agent_type.value} enregistré")
    
    async def diagnose_error(self, error_data: Dict[str, Any]) -> Dict[str, Any]:
        """Diagnostique une erreur en routant vers l'agent approprié"""
        start_time = asyncio.get_event_loop().time()
        self.metrics['total_requests'] += 1
        
        try:
            # 1. Classification de l'erreur
            agent_type = self.classifier.classify(error_data)
            logging.info(f"Erreur classifiée comme: {agent_type.value}")
            
            # 2. Sélection de l'agent
            agent = self.agents.get(agent_type)
            if not agent:
                logging.warning(f"Agent {agent_type.value} non trouvé, utilisation fallback")
                agent = self.agents.get(AgentType.APPLICATION)
            
            # 3. Génération du diagnostic
            diagnosis = await agent.diagnose(error_data, self.ollama_client)
            
            # 4. Enrichissement du résultat
            result = {
                'agent_type': agent_type.value,
                'diagnosis': diagnosis,
                'confidence': diagnosis.get('confidence', 0.5),
                'processing_time': asyncio.get_event_loop().time() - start_time,
                'timestamp': datetime.utcnow().isoformat()
            }
            
            self.metrics['successful_diagnoses'] += 1
            self._update_avg_response_time(result['processing_time'])
            
            return result
            
        except Exception as e:
            logging.error(f"Erreur lors du diagnostic: {e}")
            self.metrics['failed_diagnoses'] += 1
            
            return {
                'agent_type': 'error',
                'diagnosis': {
                    'cause': 'Erreur interne du système de diagnostic',
                    'solution': 'Vérifier les logs système et la connectivité Ollama',
                    'confidence': 0.0
                },
                'error': str(e),
                'processing_time': asyncio.get_event_loop().time() - start_time,
                'timestamp': datetime.utcnow().isoformat()
            }
    
    def _update_avg_response_time(self, response_time: float):
        """Met à jour le temps de réponse moyen"""
        current_avg = self.metrics['avg_response_time']
        total_successful = self.metrics['successful_diagnoses']
        
        self.metrics['avg_response_time'] = (
            (current_avg * (total_successful - 1) + response_time) / total_successful
        )
    
    def get_metrics(self) -> Dict[str, Any]:
        """Retourne les métriques de l'orchestrateur"""
        success_rate = 0.0
        if self.metrics['total_requests'] > 0:
            success_rate = self.metrics['successful_diagnoses'] / self.metrics['total_requests']
        
        return {
            **self.metrics,
            'success_rate': success_rate,
            'registered_agents': list(self.agents.keys())
        }
```

**Critères d'acceptation :**
- Classification automatique > 85%
- Routage vers bon agent
- Gestion des fallbacks
- Métriques de performance

#### Jour 5 : Tests Infrastructure
**Tâches :**
- [ ] Tests unitaires Ollama client
- [ ] Tests classificateur d'erreurs
- [ ] Tests orchestrateur
- [ ] Tests d'intégration

### Semaine 2 : Agents Spécialisés

#### Jour 6-7 : Agents Critiques
**Tâches :**
- [ ] DatabaseFixerAgent
- [ ] SecurityFixerAgent
- [ ] NetworkFixerAgent
- [ ] InfrastructureFixerAgent

**Agent Database :**
```python
# src/agents/database_fixer_agent.py
import re
import json
from typing import Dict, Any
from datetime import datetime

class DatabaseFixerAgent:
    """Agent spécialisé pour les erreurs de base de données"""
    
    def __init__(self):
        self.agent_type = "database"
        self.expertise_areas = [
            "SQL", "NoSQL", "ORM", "Transactions", 
            "Performance", "Connections", "Migrations"
        ]
        
        self.solution_templates = {
            'connection_timeout': {
                'immediate': [
                    'Vérifier la connectivité réseau vers la base de données',
                    'Augmenter le timeout de connexion dans la configuration',
                    'Redémarrer le service de base de données si nécessaire'
                ],
                'preventive': [
                    'Implémenter un pool de connexions',
                    'Ajouter un système de retry avec backoff exponentiel',
                    'Monitorer les métriques de performance de la DB'
                ]
            },
            'deadlock': {
                'immediate': [
                    'Identifier les transactions en conflit',
                    'Implémenter un mécanisme de retry pour les deadlocks',
                    'Optimiser l\'ordre des opérations dans les transactions'
                ],
                'preventive': [
                    'Revoir les niveaux d\'isolation des transactions',
                    'Minimiser la durée des transactions',
                    'Implémenter un ordre cohérent d\'accès aux ressources'
                ]
            }
        }
    
    async def diagnose(self, error_data: Dict[str, Any], ollama_client) -> Dict[str, Any]:
        """Diagnostique une erreur de base de données"""
        
        # Préparation du contexte
        context = self._prepare_context(error_data)
        
        # Génération du prompt spécialisé
        prompt = self._generate_prompt(error_data, context)
        
        # Appel à Ollama
        try:
            response = await ollama_client.generate(
                prompt=prompt,
                system_prompt=self._get_system_prompt()
            )
            
            # Parsing de la réponse
            diagnosis = self._parse_response(response['response'])
            
            # Enrichissement avec templates
            diagnosis = self._enrich_with_templates(diagnosis, error_data)
            
            return diagnosis
            
        except Exception as e:
            return self._fallback_diagnosis(error_data, str(e))
    
    def _get_system_prompt(self) -> str:
        """Retourne le prompt système pour l'agent database"""
        return """Tu es un expert DBA (Database Administrator) avec 15+ ans d'expérience.
        Tu maîtrises parfaitement :
        - SQL et NoSQL (PostgreSQL, MySQL, MongoDB, Redis)
        - Optimisation de performances et requêtes
        - Gestion des transactions et des deadlocks
        - Architecture haute disponibilité
        - Monitoring et troubleshooting
        
        Analyse les erreurs de base de données avec précision technique.
        Fournis des solutions pratiques et immédiatement applicables.
        Utilise un langage technique précis mais accessible."""
    
    def _generate_prompt(self, error_data: Dict[str, Any], context: Dict[str, Any]) -> str:
        """Génère le prompt spécialisé pour l'erreur database"""
        
        error_message = error_data.get('message', '')
        component = error_data.get('component', '')
        timestamp = error_data.get('timestamp', '')
        
        prompt = f"""
ANALYSE D'ERREUR BASE DE DONNÉES

Erreur détectée : {error_message}
Composant : {component}
Timestamp : {timestamp}

Contexte technique :
{json.dumps(context, indent=2)}

Fournis une analyse structurée en JSON avec :
{{
    "cause_racine": "Cause technique précise de l'erreur",
    "type_erreur": "Catégorie (connection, query, transaction, performance, etc.)",
    "severite": "1-5 (1=info, 5=critique)",
    "solution_immediate": [
        "Action 1 à effectuer immédiatement",
        "Action 2 pour résoudre rapidement"
    ],
    "solution_preventive": [
        "Mesure 1 pour éviter la récurrence",
        "Mesure 2 d'amélioration long terme"
    ],
    "requetes_diagnostic": [
        "SELECT query pour diagnostiquer",
        "SHOW command pour vérifier l'état"
    ],
    "impact_metier": "Description de l'impact sur les utilisateurs",
    "confidence": 0.85
}}

Sois précis, technique et actionnable.
"""
        return prompt
    
    def _prepare_context(self, error_data: Dict[str, Any]) -> Dict[str, Any]:
        """Prépare le contexte technique pour l'analyse"""
        return {
            'error_level': error_data.get('level', 'UNKNOWN'),
            'error_category': error_data.get('category', 'database'),
            'metadata': error_data.get('metadata', {}),
            'recent_errors': self._get_recent_similar_errors(error_data),
            'system_info': {
                'timestamp': datetime.utcnow().isoformat(),
                'agent': self.agent_type
            }
        }
    
    def _parse_response(self, response: str) -> Dict[str, Any]:
        """Parse la réponse JSON d'Ollama"""
        try:
            # Extraction du JSON de la réponse
            json_start = response.find('{')
            json_end = response.rfind('}') + 1
            
            if json_start != -1 and json_end != -1:
                json_str = response[json_start:json_end]
                return json.loads(json_str)
            else:
                # Fallback si pas de JSON trouvé
                return self._extract_structured_info(response)
                
        except json.JSONDecodeError:
            return self._extract_structured_info(response)
    
    def _enrich_with_templates(self, diagnosis: Dict[str, Any], error_data: Dict[str, Any]) -> Dict[str, Any]:
        """Enrichit le diagnostic avec les templates prédéfinis"""
        error_type = diagnosis.get('type_erreur', '').lower()
        
        # Recherche de template correspondant
        for template_key, template_data in self.solution_templates.items():
            if template_key in error_type or any(
                keyword in error_data.get('message', '').lower() 
                for keyword in template_key.split('_')
            ):
                # Fusion avec le template
                if 'solution_immediate' not in diagnosis:
                    diagnosis['solution_immediate'] = template_data['immediate']
                if 'solution_preventive' not in diagnosis:
                    diagnosis['solution_preventive'] = template_data['preventive']
                break
        
        return diagnosis
    
    def _fallback_diagnosis(self, error_data: Dict[str, Any], error: str) -> Dict[str, Any]:
        """Diagnostic de fallback en cas d'erreur"""
        return {
            'cause_racine': 'Erreur de base de données détectée - analyse automatique échouée',
            'type_erreur': 'database_generic',
            'severite': 3,
            'solution_immediate': [
                'Vérifier les logs de la base de données',
                'Contrôler la connectivité et les ressources',
                'Redémarrer les services si nécessaire'
            ],
            'solution_preventive': [
                'Implémenter un monitoring proactif',
                'Configurer des alertes sur les métriques critiques'
            ],
            'impact_metier': 'Impact potentiel sur les fonctionnalités utilisant la base de données',
            'confidence': 0.3,
            'fallback_reason': error
        }
```

**Critères d'acceptation :**
- 4 agents spécialisés fonctionnels
- Prompts optimisés par domaine
- Solutions structurées et actionables
- Système de fallback robuste

#### Jour 8-9 : Agents Complémentaires
**Tâches :**
- [ ] ApplicationFixerAgent
- [ ] IntegrationFixerAgent
- [ ] Système de cache des réponses
- [ ] Validation des solutions

#### Jour 10 : Tests et Optimisation
**Tâches :**
- [ ] Tests end-to-end du système multi-agents
- [ ] Optimisation des prompts
- [ ] Tests de performance avec Ollama
- [ ] Documentation complète

## 🔧 Configuration Technique

### Configuration Ollama
```yaml
# config/ollama.yaml
ollama:
  host: "localhost"
  port: 11434
  model: "llama3.1:7b"
  timeout: 120
  max_retries: 3
  temperature: 0.1
  
  # Pool de connexions
  pool_size: 5
  max_concurrent_requests: 10
  
  # Cache
  cache_enabled: true
  cache_ttl: 3600  # 1 heure
  
  # Fallback
  fallback_enabled: true
  fallback_timeout: 30
```

### Templates de Prompts
```yaml
# config/agent_prompts.yaml
agents:
  database:
    system_prompt: |
      Tu es un expert DBA avec 15+ ans d'expérience.
      Analyse les erreurs de base de données avec précision technique.
      
    user_prompt_template: |
      ANALYSE D'ERREUR BASE DE DONNÉES
      
      Erreur: {error_message}
      Composant: {component}
      Contexte: {context}
      
      Fournis une analyse JSON structurée...
      
  security:
    system_prompt: |
      Tu es un expert en cybersécurité et compliance.
      Analyse les incidents de sécurité avec rigueur.
      
    user_prompt_template: |
      ANALYSE D'INCIDENT SÉCURITÉ
      
      Alerte: {error_message}
      Type: {security_type}
      Contexte: {context}
      
      Fournis une analyse de sécurité...
```

## 📊 Métriques de Succès

### Métriques Techniques
- **Temps de réponse IA** : < 30 secondes
- **Précision diagnostic** : > 85%
- **Disponibilité Ollama** : > 95%
- **Classification correcte** : > 90%

### Métriques Qualité
- **Solutions actionables** : > 80%
- **Confidence moyenne** : > 0.7
- **Taux de fallback** : < 10%
- **Satisfaction utilisateur** : > 4/5

### Métriques Performance
- **Débit diagnostic** : > 50/heure
- **Utilisation mémoire** : < 500MB
- **Utilisation CPU** : < 30%
- **Cache hit rate** : > 60%

## 🚨 Risques et Mitigation

### Risques Techniques
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Ollama indisponible | Moyenne | Critique | Système de fallback + monitoring |
| Réponses IA incohérentes | Élevée | Moyen | Validation + templates de secours |
| Performance dégradée | Moyenne | Élevé | Cache + optimisation prompts |

### Risques Fonctionnels
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Classification incorrecte | Moyenne | Moyen | Fine-tuning règles + feedback |
| Solutions non applicables | Élevée | Élevé | Validation humaine + templates |

## 📝 Checklist de Fin de Sprint

### Infrastructure IA
- [ ] Client Ollama robuste et performant
- [ ] Orchestrateur multi-agents fonctionnel
- [ ] Classificateur d'erreurs précis
- [ ] Système de fallback opérationnel

### Agents Spécialisés
- [ ] 6 agents Fixer implémentés
- [ ] Prompts optimisés par domaine
- [ ] Solutions structurées et actionables
- [ ] Cache et performance optimisés

### Tests et Qualité
- [ ] Tests unitaires > 85% coverage
- [ ] Tests d'intégration avec Ollama
- [ ] Tests de performance validés
- [ ] Documentation complète

### Configuration
- [ ] Templates de prompts configurables
- [ ] Paramètres Ollama optimisés
- [ ] Règles de classification ajustables
- [ ] Monitoring et métriques

## 🎯 Critères de Passage Sprint 4

Pour passer au Sprint 4, tous ces critères doivent être validés :

1. **IA fonctionnelle** : Ollama intégré et stable
2. **Multi-agents** : 6 agents spécialisés opérationnels
3. **Diagnostic** : Solutions structurées et actionables
4. **Performance** : Temps de réponse < 30 secondes
5. **Qualité** : Tests et documentation complets

---

**🚀 Objectif : Système multi-agents IA intelligent et spécialisé**

*Dernière mise à jour : 07/01/2025*