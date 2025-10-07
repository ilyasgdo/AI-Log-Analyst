# Sprint 5 : Qdrant et Recherche Sémantique - Plan d'Action Détaillé

## 📋 Vue d'Ensemble

**Durée** : 2 semaines  
**Objectif** : Implémenter la base de données vectorielle Qdrant et la recherche sémantique avancée  
**Équipe** : 1-2 développeurs  
**Priorité** : Élevée  
**Prérequis** : Sprint 4 terminé (interface et notifications opérationnelles)

## 🎯 Objectifs du Sprint

### Objectifs Principaux
1. **Base vectorielle Qdrant** : Installation et configuration
2. **Embeddings des logs** : Génération et stockage des vecteurs
3. **Recherche sémantique** : Similarité et clustering des erreurs
4. **Détection d'anomalies** : Patterns inhabituels via ML
5. **Recommandations IA** : Suggestions basées sur l'historique

### Objectifs Secondaires
- Optimisation des performances vectorielles
- Interface de recherche avancée
- Visualisation des clusters d'erreurs
- Système de feedback pour améliorer les embeddings
- Intégration avec les agents existants

## 📦 Livrables

### Infrastructure Qdrant
- [ ] Installation et configuration Qdrant
- [ ] Collections optimisées pour les logs
- [ ] Système de backup et restauration
- [ ] Monitoring des performances vectorielles

### Système d'Embeddings
- [ ] Pipeline de génération d'embeddings
- [ ] Modèles de vectorisation optimisés
- [ ] Mise à jour incrémentale des vecteurs
- [ ] Cache intelligent des embeddings

### Recherche Sémantique
- [ ] API de recherche par similarité
- [ ] Clustering automatique des erreurs
- [ ] Détection d'anomalies ML
- [ ] Système de recommandations

### Interface Avancée
- [ ] Interface de recherche sémantique
- [ ] Visualisation des clusters
- [ ] Exploration interactive des patterns
- [ ] Dashboard d'anomalies

## 🗓️ Planning Détaillé

### Semaine 1 : Infrastructure Qdrant et Embeddings

#### Jour 1-2 : Installation et Configuration Qdrant
**Tâches :**
- [ ] Installation Qdrant (Docker/local)
- [ ] Configuration des collections
- [ ] Optimisation des performances
- [ ] Tests de connectivité

**Configuration Qdrant :**
```python
# src/vector_db/qdrant_client.py
import asyncio
from qdrant_client import QdrantClient
from qdrant_client.http import models
from qdrant_client.http.models import Distance, VectorParams, PointStruct
from typing import List, Dict, Any, Optional
import numpy as np
import logging
from dataclasses import dataclass
import json

@dataclass
class QdrantConfig:
    """Configuration Qdrant"""
    host: str = "localhost"
    port: int = 6333
    api_key: Optional[str] = None
    timeout: int = 60
    
    # Collections
    logs_collection: str = "error_logs"
    diagnostics_collection: str = "diagnostics"
    patterns_collection: str = "error_patterns"
    
    # Vecteurs
    vector_size: int = 384  # all-MiniLM-L6-v2
    distance_metric: Distance = Distance.COSINE
    
    # Performance
    hnsw_config: Dict[str, Any] = None
    quantization_config: Dict[str, Any] = None

class QdrantManager:
    """Gestionnaire de la base vectorielle Qdrant"""
    
    def __init__(self, config: QdrantConfig):
        self.config = config
        self.client = None
        self._initialize_client()
    
    def _initialize_client(self):
        """Initialise le client Qdrant"""
        try:
            self.client = QdrantClient(
                host=self.config.host,
                port=self.config.port,
                api_key=self.config.api_key,
                timeout=self.config.timeout
            )
            logging.info("Client Qdrant initialisé")
        except Exception as e:
            logging.error(f"Erreur initialisation Qdrant: {e}")
            raise
    
    async def setup_collections(self):
        """Configure les collections Qdrant"""
        collections_config = {
            self.config.logs_collection: {
                "description": "Logs d'erreurs avec embeddings",
                "vector_config": VectorParams(
                    size=self.config.vector_size,
                    distance=self.config.distance_metric
                ),
                "hnsw_config": models.HnswConfigDiff(
                    m=16,
                    ef_construct=100,
                    full_scan_threshold=10000
                ),
                "quantization_config": models.ScalarQuantization(
                    scalar=models.ScalarQuantizationConfig(
                        type=models.ScalarType.INT8,
                        quantile=0.99,
                        always_ram=True
                    )
                )
            },
            self.config.diagnostics_collection: {
                "description": "Diagnostics IA avec embeddings",
                "vector_config": VectorParams(
                    size=self.config.vector_size,
                    distance=self.config.distance_metric
                )
            },
            self.config.patterns_collection: {
                "description": "Patterns d'erreurs identifiés",
                "vector_config": VectorParams(
                    size=self.config.vector_size,
                    distance=self.config.distance_metric
                )
            }
        }
        
        for collection_name, config in collections_config.items():
            try:
                # Vérifier si la collection existe
                collections = self.client.get_collections()
                existing_names = [col.name for col in collections.collections]
                
                if collection_name not in existing_names:
                    # Créer la collection
                    self.client.create_collection(
                        collection_name=collection_name,
                        vectors_config=config["vector_config"],
                        hnsw_config=config.get("hnsw_config"),
                        quantization_config=config.get("quantization_config")
                    )
                    logging.info(f"Collection {collection_name} créée")
                else:
                    logging.info(f"Collection {collection_name} existe déjà")
                    
            except Exception as e:
                logging.error(f"Erreur création collection {collection_name}: {e}")
                raise
    
    async def insert_log_embedding(
        self, 
        log_id: str, 
        embedding: List[float], 
        metadata: Dict[str, Any]
    ) -> bool:
        """Insère un embedding de log"""
        try:
            point = PointStruct(
                id=log_id,
                vector=embedding,
                payload={
                    **metadata,
                    "timestamp": metadata.get("timestamp"),
                    "component": metadata.get("component", "unknown"),
                    "level": metadata.get("level", "INFO"),
                    "message": metadata.get("message", ""),
                    "indexed_at": datetime.utcnow().isoformat()
                }
            )
            
            self.client.upsert(
                collection_name=self.config.logs_collection,
                points=[point]
            )
            
            return True
            
        except Exception as e:
            logging.error(f"Erreur insertion embedding {log_id}: {e}")
            return False
    
    async def search_similar_logs(
        self, 
        query_embedding: List[float], 
        limit: int = 10,
        score_threshold: float = 0.7,
        filters: Optional[Dict[str, Any]] = None
    ) -> List[Dict[str, Any]]:
        """Recherche des logs similaires"""
        try:
            # Construction des filtres Qdrant
            qdrant_filter = None
            if filters:
                conditions = []
                
                if "component" in filters:
                    conditions.append(
                        models.FieldCondition(
                            key="component",
                            match=models.MatchValue(value=filters["component"])
                        )
                    )
                
                if "level" in filters:
                    conditions.append(
                        models.FieldCondition(
                            key="level",
                            match=models.MatchAny(any=filters["level"])
                        )
                    )
                
                if "time_range" in filters:
                    conditions.append(
                        models.FieldCondition(
                            key="timestamp",
                            range=models.Range(
                                gte=filters["time_range"]["start"],
                                lte=filters["time_range"]["end"]
                            )
                        )
                    )
                
                if conditions:
                    qdrant_filter = models.Filter(must=conditions)
            
            # Recherche
            results = self.client.search(
                collection_name=self.config.logs_collection,
                query_vector=query_embedding,
                query_filter=qdrant_filter,
                limit=limit,
                score_threshold=score_threshold,
                with_payload=True,
                with_vectors=False
            )
            
            # Formatage des résultats
            similar_logs = []
            for result in results:
                similar_logs.append({
                    "id": result.id,
                    "score": result.score,
                    "metadata": result.payload,
                    "message": result.payload.get("message", ""),
                    "component": result.payload.get("component", ""),
                    "timestamp": result.payload.get("timestamp", "")
                })
            
            return similar_logs
            
        except Exception as e:
            logging.error(f"Erreur recherche similarité: {e}")
            return []
    
    async def get_error_clusters(
        self, 
        min_cluster_size: int = 5,
        time_window_hours: int = 24
    ) -> List[Dict[str, Any]]:
        """Identifie les clusters d'erreurs"""
        try:
            # Récupération des logs récents
            time_filter = models.Filter(
                must=[
                    models.FieldCondition(
                        key="timestamp",
                        range=models.Range(
                            gte=(datetime.utcnow() - timedelta(hours=time_window_hours)).isoformat()
                        )
                    )
                ]
            )
            
            # Scroll pour récupérer tous les points
            points = []
            offset = None
            
            while True:
                result = self.client.scroll(
                    collection_name=self.config.logs_collection,
                    scroll_filter=time_filter,
                    limit=1000,
                    offset=offset,
                    with_payload=True,
                    with_vectors=True
                )
                
                points.extend(result[0])
                
                if result[1] is None:  # Plus de points
                    break
                offset = result[1]
            
            if len(points) < min_cluster_size:
                return []
            
            # Clustering avec sklearn
            from sklearn.cluster import DBSCAN
            import numpy as np
            
            vectors = np.array([point.vector for point in points])
            
            # DBSCAN clustering
            clustering = DBSCAN(
                eps=0.3,  # Distance maximale entre points
                min_samples=min_cluster_size,
                metric='cosine'
            ).fit(vectors)
            
            # Analyse des clusters
            clusters = {}
            for i, (point, label) in enumerate(zip(points, clustering.labels_)):
                if label == -1:  # Bruit
                    continue
                
                if label not in clusters:
                    clusters[label] = {
                        'points': [],
                        'components': set(),
                        'levels': set(),
                        'messages': []
                    }
                
                clusters[label]['points'].append({
                    'id': point.id,
                    'payload': point.payload
                })
                clusters[label]['components'].add(point.payload.get('component', 'unknown'))
                clusters[label]['levels'].add(point.payload.get('level', 'INFO'))
                clusters[label]['messages'].append(point.payload.get('message', ''))
            
            # Formatage des résultats
            cluster_results = []
            for cluster_id, cluster_data in clusters.items():
                if len(cluster_data['points']) >= min_cluster_size:
                    cluster_results.append({
                        'cluster_id': cluster_id,
                        'size': len(cluster_data['points']),
                        'components': list(cluster_data['components']),
                        'levels': list(cluster_data['levels']),
                        'sample_messages': cluster_data['messages'][:5],
                        'points': cluster_data['points']
                    })
            
            return sorted(cluster_results, key=lambda x: x['size'], reverse=True)
            
        except Exception as e:
            logging.error(f"Erreur clustering: {e}")
            return []
    
    async def detect_anomalies(
        self, 
        time_window_hours: int = 24,
        anomaly_threshold: float = 0.1
    ) -> List[Dict[str, Any]]:
        """Détecte les anomalies dans les patterns d'erreurs"""
        try:
            # Récupération des logs récents
            recent_logs = await self._get_recent_logs(time_window_hours)
            
            if len(recent_logs) < 10:
                return []
            
            # Analyse des patterns temporels
            from collections import defaultdict
            import pandas as pd
            
            # Groupement par heure
            hourly_counts = defaultdict(int)
            component_counts = defaultdict(int)
            level_counts = defaultdict(int)
            
            for log in recent_logs:
                timestamp = pd.to_datetime(log.payload.get('timestamp'))
                hour_key = timestamp.strftime('%Y-%m-%d %H:00:00')
                
                hourly_counts[hour_key] += 1
                component_counts[log.payload.get('component', 'unknown')] += 1
                level_counts[log.payload.get('level', 'INFO')] += 1
            
            # Détection d'anomalies temporelles
            anomalies = []
            
            # Anomalie: pic d'erreurs inhabituel
            counts = list(hourly_counts.values())
            if counts:
                mean_count = np.mean(counts)
                std_count = np.std(counts)
                
                for hour, count in hourly_counts.items():
                    if count > mean_count + 2 * std_count:
                        anomalies.append({
                            'type': 'temporal_spike',
                            'timestamp': hour,
                            'value': count,
                            'expected': mean_count,
                            'severity': 'high' if count > mean_count + 3 * std_count else 'medium'
                        })
            
            # Anomalie: nouveau composant avec beaucoup d'erreurs
            total_logs = len(recent_logs)
            for component, count in component_counts.items():
                if count / total_logs > anomaly_threshold:
                    # Vérifier si c'est un nouveau pattern
                    historical_count = await self._get_historical_component_count(
                        component, 
                        time_window_hours * 7  # 7x la fenêtre
                    )
                    
                    if historical_count == 0 or count > historical_count * 3:
                        anomalies.append({
                            'type': 'new_component_pattern',
                            'component': component,
                            'current_count': count,
                            'historical_count': historical_count,
                            'severity': 'high' if count > historical_count * 5 else 'medium'
                        })
            
            return anomalies
            
        except Exception as e:
            logging.error(f"Erreur détection anomalies: {e}")
            return []
    
    async def get_recommendations(
        self, 
        error_embedding: List[float],
        context: Dict[str, Any]
    ) -> List[Dict[str, Any]]:
        """Génère des recommandations basées sur l'historique"""
        try:
            # Recherche d'erreurs similaires résolues
            similar_logs = await self.search_similar_logs(
                query_embedding=error_embedding,
                limit=20,
                score_threshold=0.8,
                filters={
                    "component": context.get("component"),
                    "level": ["ERROR", "CRITICAL"]
                }
            )
            
            # Recherche de diagnostics associés
            recommendations = []
            
            for similar_log in similar_logs:
                # Rechercher les diagnostics pour ce log
                diagnostic_results = self.client.search(
                    collection_name=self.config.diagnostics_collection,
                    query_vector=error_embedding,
                    query_filter=models.Filter(
                        must=[
                            models.FieldCondition(
                                key="original_log_id",
                                match=models.MatchValue(value=similar_log["id"])
                            )
                        ]
                    ),
                    limit=5,
                    with_payload=True
                )
                
                for diagnostic in diagnostic_results:
                    if diagnostic.score > 0.7:
                        recommendations.append({
                            'type': 'historical_solution',
                            'similarity_score': diagnostic.score,
                            'original_error': similar_log["message"],
                            'solution': diagnostic.payload.get("solution", ""),
                            'success_rate': diagnostic.payload.get("success_rate", 0.0),
                            'last_used': diagnostic.payload.get("last_used", ""),
                            'confidence': diagnostic.payload.get("confidence", 0.5)
                        })
            
            # Tri par pertinence
            recommendations.sort(
                key=lambda x: x['similarity_score'] * x['confidence'], 
                reverse=True
            )
            
            return recommendations[:5]  # Top 5
            
        except Exception as e:
            logging.error(f"Erreur génération recommandations: {e}")
            return []
    
    async def health_check(self) -> Dict[str, Any]:
        """Vérification de santé Qdrant"""
        try:
            # Test de connectivité
            collections = self.client.get_collections()
            
            # Statistiques des collections
            stats = {}
            for collection in collections.collections:
                collection_info = self.client.get_collection(collection.name)
                stats[collection.name] = {
                    'points_count': collection_info.points_count,
                    'segments_count': collection_info.segments_count,
                    'disk_data_size': collection_info.disk_data_size,
                    'ram_data_size': collection_info.ram_data_size
                }
            
            return {
                'status': 'healthy',
                'collections': len(collections.collections),
                'collection_stats': stats,
                'timestamp': datetime.utcnow().isoformat()
            }
            
        except Exception as e:
            return {
                'status': 'unhealthy',
                'error': str(e),
                'timestamp': datetime.utcnow().isoformat()
            }
```

**Critères d'acceptation :**
- Qdrant installé et configuré
- Collections optimisées créées
- Tests de connectivité réussis
- Monitoring des performances

#### Jour 3-4 : Système d'Embeddings
**Tâches :**
- [ ] Pipeline de génération d'embeddings
- [ ] Modèles de vectorisation
- [ ] Cache intelligent
- [ ] Mise à jour incrémentale

**Pipeline d'Embeddings :**
```python
# src/embeddings/embedding_pipeline.py
import asyncio
from typing import List, Dict, Any, Optional
from sentence_transformers import SentenceTransformer
import numpy as np
import hashlib
import json
import logging
from dataclasses import dataclass
from datetime import datetime
import redis
import pickle

@dataclass
class EmbeddingConfig:
    """Configuration des embeddings"""
    model_name: str = "all-MiniLM-L6-v2"
    max_sequence_length: int = 512
    batch_size: int = 32
    
    # Cache Redis
    redis_host: str = "localhost"
    redis_port: int = 6379
    redis_db: int = 0
    cache_ttl: int = 86400  # 24h
    
    # Optimisations
    use_gpu: bool = False
    normalize_embeddings: bool = True
    
class EmbeddingPipeline:
    """Pipeline de génération d'embeddings pour les logs"""
    
    def __init__(self, config: EmbeddingConfig):
        self.config = config
        self.model = None
        self.redis_client = None
        self._initialize_model()
        self._initialize_cache()
    
    def _initialize_model(self):
        """Initialise le modèle d'embeddings"""
        try:
            device = 'cuda' if self.config.use_gpu else 'cpu'
            self.model = SentenceTransformer(
                self.config.model_name,
                device=device
            )
            
            # Configuration du modèle
            self.model.max_seq_length = self.config.max_sequence_length
            
            logging.info(f"Modèle d'embeddings {self.config.model_name} initialisé sur {device}")
            
        except Exception as e:
            logging.error(f"Erreur initialisation modèle embeddings: {e}")
            raise
    
    def _initialize_cache(self):
        """Initialise le cache Redis"""
        try:
            self.redis_client = redis.Redis(
                host=self.config.redis_host,
                port=self.config.redis_port,
                db=self.config.redis_db,
                decode_responses=False  # Pour les données binaires
            )
            
            # Test de connectivité
            self.redis_client.ping()
            logging.info("Cache Redis initialisé")
            
        except Exception as e:
            logging.warning(f"Cache Redis non disponible: {e}")
            self.redis_client = None
    
    def _generate_cache_key(self, text: str) -> str:
        """Génère une clé de cache pour le texte"""
        text_hash = hashlib.md5(text.encode('utf-8')).hexdigest()
        return f"embedding:{self.config.model_name}:{text_hash}"
    
    def _get_cached_embedding(self, text: str) -> Optional[np.ndarray]:
        """Récupère un embedding du cache"""
        if not self.redis_client:
            return None
        
        try:
            cache_key = self._generate_cache_key(text)
            cached_data = self.redis_client.get(cache_key)
            
            if cached_data:
                embedding = pickle.loads(cached_data)
                return np.array(embedding)
            
        except Exception as e:
            logging.warning(f"Erreur lecture cache: {e}")
        
        return None
    
    def _cache_embedding(self, text: str, embedding: np.ndarray):
        """Met en cache un embedding"""
        if not self.redis_client:
            return
        
        try:
            cache_key = self._generate_cache_key(text)
            cached_data = pickle.dumps(embedding.tolist())
            
            self.redis_client.setex(
                cache_key,
                self.config.cache_ttl,
                cached_data
            )
            
        except Exception as e:
            logging.warning(f"Erreur écriture cache: {e}")
    
    def preprocess_log_text(self, log_data: Dict[str, Any]) -> str:
        """Préprocesse un log pour la vectorisation"""
        # Extraction des champs pertinents
        message = log_data.get('message', '')
        component = log_data.get('component', '')
        level = log_data.get('level', '')
        
        # Construction du texte enrichi
        enriched_text = f"[{level}] {component}: {message}"
        
        # Ajout du contexte si disponible
        if 'stack_trace' in log_data:
            stack_trace = log_data['stack_trace'][:500]  # Limiter la taille
            enriched_text += f" | Stack: {stack_trace}"
        
        if 'metadata' in log_data:
            metadata = log_data['metadata']
            if isinstance(metadata, dict):
                for key, value in metadata.items():
                    if key in ['user_id', 'session_id', 'request_id']:
                        enriched_text += f" | {key}: {value}"
        
        # Nettoyage et normalisation
        enriched_text = enriched_text.replace('\n', ' ').replace('\t', ' ')
        enriched_text = ' '.join(enriched_text.split())  # Normaliser les espaces
        
        # Troncature si nécessaire
        if len(enriched_text) > self.config.max_sequence_length * 4:  # Approximation
            enriched_text = enriched_text[:self.config.max_sequence_length * 4]
        
        return enriched_text
    
    async def generate_embedding(self, text: str) -> np.ndarray:
        """Génère un embedding pour un texte"""
        # Vérification du cache
        cached_embedding = self._get_cached_embedding(text)
        if cached_embedding is not None:
            return cached_embedding
        
        try:
            # Génération de l'embedding
            embedding = self.model.encode(
                text,
                convert_to_numpy=True,
                normalize_embeddings=self.config.normalize_embeddings
            )
            
            # Mise en cache
            self._cache_embedding(text, embedding)
            
            return embedding
            
        except Exception as e:
            logging.error(f"Erreur génération embedding: {e}")
            # Retourner un vecteur zéro en cas d'erreur
            return np.zeros(self.model.get_sentence_embedding_dimension())
    
    async def generate_batch_embeddings(self, texts: List[str]) -> List[np.ndarray]:
        """Génère des embeddings par batch"""
        embeddings = []
        uncached_texts = []
        uncached_indices = []
        
        # Vérification du cache pour chaque texte
        for i, text in enumerate(texts):
            cached_embedding = self._get_cached_embedding(text)
            if cached_embedding is not None:
                embeddings.append(cached_embedding)
            else:
                embeddings.append(None)  # Placeholder
                uncached_texts.append(text)
                uncached_indices.append(i)
        
        # Génération des embeddings non cachés
        if uncached_texts:
            try:
                batch_embeddings = self.model.encode(
                    uncached_texts,
                    batch_size=self.config.batch_size,
                    convert_to_numpy=True,
                    normalize_embeddings=self.config.normalize_embeddings,
                    show_progress_bar=len(uncached_texts) > 10
                )
                
                # Mise à jour des résultats et du cache
                for i, (text, embedding) in enumerate(zip(uncached_texts, batch_embeddings)):
                    original_index = uncached_indices[i]
                    embeddings[original_index] = embedding
                    
                    # Mise en cache
                    self._cache_embedding(text, embedding)
                    
            except Exception as e:
                logging.error(f"Erreur génération batch embeddings: {e}")
                # Remplacer les None par des vecteurs zéro
                zero_vector = np.zeros(self.model.get_sentence_embedding_dimension())
                for i in uncached_indices:
                    embeddings[i] = zero_vector
        
        return embeddings
    
    async def process_log_batch(self, logs: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Traite un batch de logs pour générer leurs embeddings"""
        # Préprocessing
        processed_texts = [self.preprocess_log_text(log) for log in logs]
        
        # Génération des embeddings
        embeddings = await self.generate_batch_embeddings(processed_texts)
        
        # Enrichissement des logs
        enriched_logs = []
        for log, embedding, processed_text in zip(logs, embeddings, processed_texts):
            enriched_log = {
                **log,
                'embedding': embedding.tolist(),
                'processed_text': processed_text,
                'embedding_model': self.config.model_name,
                'embedding_timestamp': datetime.utcnow().isoformat()
            }
            enriched_logs.append(enriched_log)
        
        return enriched_logs
    
    def calculate_similarity(
        self, 
        embedding1: np.ndarray, 
        embedding2: np.ndarray
    ) -> float:
        """Calcule la similarité cosinus entre deux embeddings"""
        try:
            # Normalisation si nécessaire
            if not self.config.normalize_embeddings:
                embedding1 = embedding1 / np.linalg.norm(embedding1)
                embedding2 = embedding2 / np.linalg.norm(embedding2)
            
            # Similarité cosinus
            similarity = np.dot(embedding1, embedding2)
            return float(similarity)
            
        except Exception as e:
            logging.error(f"Erreur calcul similarité: {e}")
            return 0.0
    
    def get_model_info(self) -> Dict[str, Any]:
        """Retourne les informations du modèle"""
        return {
            'model_name': self.config.model_name,
            'embedding_dimension': self.model.get_sentence_embedding_dimension(),
            'max_sequence_length': self.config.max_sequence_length,
            'device': str(self.model.device),
            'cache_enabled': self.redis_client is not None
        }
    
    async def cleanup_cache(self, older_than_hours: int = 168):  # 7 jours
        """Nettoie le cache des anciens embeddings"""
        if not self.redis_client:
            return
        
        try:
            # Pattern pour les clés d'embeddings
            pattern = f"embedding:{self.config.model_name}:*"
            
            # Scan et suppression des anciennes clés
            cursor = 0
            deleted_count = 0
            
            while True:
                cursor, keys = self.redis_client.scan(
                    cursor=cursor,
                    match=pattern,
                    count=1000
                )
                
                if keys:
                    # Vérification de l'âge des clés (approximatif via TTL)
                    pipeline = self.redis_client.pipeline()
                    for key in keys:
                        ttl = self.redis_client.ttl(key)
                        if ttl < (self.config.cache_ttl - older_than_hours * 3600):
                            pipeline.delete(key)
                            deleted_count += 1
                    
                    pipeline.execute()
                
                if cursor == 0:
                    break
            
            logging.info(f"Cache nettoyé: {deleted_count} entrées supprimées")
            
        except Exception as e:
            logging.error(f"Erreur nettoyage cache: {e}")
```

**Critères d'acceptation :**
- Pipeline d'embeddings fonctionnel
- Cache Redis opérationnel
- Traitement par batch efficace
- Modèles optimisés chargés

#### Jour 5 : Tests Infrastructure
**Tâches :**
- [ ] Tests Qdrant et collections
- [ ] Tests pipeline embeddings
- [ ] Tests de performance
- [ ] Benchmarks de similarité

### Semaine 2 : Recherche Sémantique et Interface

#### Jour 6-7 : Recherche Sémantique Avancée
**Tâches :**
- [ ] API de recherche par similarité
- [ ] Clustering automatique
- [ ] Détection d'anomalies ML
- [ ] Système de recommandations

#### Jour 8-9 : Interface de Recherche
**Tâches :**
- [ ] Interface de recherche sémantique
- [ ] Visualisation des clusters
- [ ] Dashboard d'anomalies
- [ ] Exploration interactive

#### Jour 10 : Tests et Optimisation
**Tâches :**
- [ ] Tests end-to-end recherche sémantique
- [ ] Optimisation des performances
- [ ] Tests de charge Qdrant
- [ ] Documentation complète

## 🔧 Configuration Technique

### Configuration Qdrant
```yaml
# config/qdrant.yaml
qdrant:
  host: "localhost"
  port: 6333
  api_key: null
  timeout: 60
  
  collections:
    error_logs:
      vector_size: 384
      distance: "Cosine"
      hnsw_config:
        m: 16
        ef_construct: 100
        full_scan_threshold: 10000
      quantization:
        type: "int8"
        quantile: 0.99
        always_ram: true
    
    diagnostics:
      vector_size: 384
      distance: "Cosine"
    
    patterns:
      vector_size: 384
      distance: "Cosine"
  
  performance:
    max_concurrent_requests: 100
    indexing_threshold: 20000
    payload_storage_type: "on_disk"
```

### Configuration Embeddings
```yaml
# config/embeddings.yaml
embeddings:
  model_name: "all-MiniLM-L6-v2"
  max_sequence_length: 512
  batch_size: 32
  use_gpu: false
  normalize_embeddings: true
  
  cache:
    redis_host: "localhost"
    redis_port: 6379
    redis_db: 0
    ttl_seconds: 86400
  
  preprocessing:
    include_stack_trace: true
    max_stack_trace_length: 500
    include_metadata_fields:
      - "user_id"
      - "session_id"
      - "request_id"
  
  similarity:
    default_threshold: 0.7
    clustering_threshold: 0.8
    anomaly_threshold: 0.9
```

## 📊 Métriques de Succès

### Métriques Techniques
- **Temps génération embedding** : < 100ms
- **Débit Qdrant** : > 1000 req/sec
- **Précision recherche** : > 85%
- **Temps réponse recherche** : < 500ms

### Métriques Qualité
- **Pertinence clusters** : > 80%
- **Détection anomalies** : > 90%
- **Recommandations utiles** : > 75%
- **Cache hit rate** : > 60%

### Métriques Performance
- **Utilisation mémoire Qdrant** : < 2GB
- **Utilisation CPU embeddings** : < 50%
- **Stockage vectoriel** : Optimisé
- **Latence réseau** : < 50ms

## 🚨 Risques et Mitigation

### Risques Techniques
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Performance Qdrant | Moyenne | Élevé | Optimisation + monitoring |
| Qualité embeddings | Élevée | Moyen | Fine-tuning + validation |
| Consommation mémoire | Élevée | Élevé | Quantization + cache |

### Risques Fonctionnels
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Faux positifs recherche | Élevée | Moyen | Seuils adaptatifs |
| Clusters non pertinents | Moyenne | Moyen | Validation humaine |

## 📝 Checklist de Fin de Sprint

### Infrastructure Qdrant
- [ ] Installation et configuration réussies
- [ ] Collections optimisées créées
- [ ] Monitoring des performances
- [ ] Système de backup configuré

### Système d'Embeddings
- [ ] Pipeline de génération fonctionnel
- [ ] Cache Redis opérationnel
- [ ] Modèles optimisés chargés
- [ ] Traitement par batch efficace

### Recherche Sémantique
- [ ] API de recherche par similarité
- [ ] Clustering automatique des erreurs
- [ ] Détection d'anomalies ML
- [ ] Système de recommandations

### Interface Avancée
- [ ] Interface de recherche sémantique
- [ ] Visualisation des clusters
- [ ] Dashboard d'anomalies
- [ ] Exploration interactive des patterns

### Tests et Qualité
- [ ] Tests unitaires > 85% coverage
- [ ] Tests de performance validés
- [ ] Benchmarks de similarité
- [ ] Documentation complète

## 🎯 Critères de Passage Sprint 6

Pour passer au Sprint 6, tous ces critères doivent être validés :

1. **Qdrant opérationnel** : Base vectorielle performante
2. **Embeddings fonctionnels** : Pipeline de vectorisation efficace
3. **Recherche sémantique** : API et interface opérationnelles
4. **Détection d'anomalies** : ML et clustering fonctionnels
5. **Performance validée** : Métriques cibles atteintes

---

**🚀 Objectif : Recherche sémantique intelligente et détection d'anomalies ML**

*Dernière mise à jour : 07/01/2025*