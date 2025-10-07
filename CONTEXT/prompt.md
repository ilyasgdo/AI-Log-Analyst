Tu es un expert en ingénierie logicielle et intelligence artificielle. Ta tâche est de générer un **Plan de Réalisation (PDR)** complet pour un projet IA appelé **AI Ops Copilot**. Ce projet est un système multi-agents local-first qui analyse les logs d’applications backend pour détecter les erreurs et proposer des solutions automatisées.

### Objectifs du projet :
1. Surveiller les logs des apps backend et serveurs (Cloudflared ou fichiers locaux).
2. Détecter automatiquement les erreurs ou anomalies.
3. Envoyer les logs problématiques à des agents IA qui vont :
   - Comprendre le contexte (app, endpoint, payload, stacktrace).
   - Diagnostiquer la cause probable.
   - Proposer un correctif ou une solution.
   - Rédiger un résumé et envoyer un mail ou mettre à jour un dashboard.
4. Stocker tous les logs, solutions et résumés dans une base vectorielle (Qdrant) pour recherche et mémoire sémantique.
5. Fournir une interface Streamlit pour explorer les logs, poser des questions et visualiser les erreurs et solutions.
6. Permettre la recherche sémantique pour retrouver des erreurs similaires.

### Stack technique (full Python, local-first) :
- Python 3.11+
- LLM local : Ollama (Llama 3 / Mistral)
- Orchestration multi-agents : LangChain + CrewAI
- Vector DB : Qdrant local
- Dashboard UI : Streamlit
- Parsing logs : regex / JSON / custom
- Email : smtplib ou Gmail API
- Base historique : SQLite (optionnelle)

### Modules / fonctionnalités à implémenter :
1. **Collector** : ingestion temps réel des logs Cloudflared ou fichiers locaux.
2. **Parser** : extraction des champs (timestamp, niveau, message, endpoint, requête, payload, stacktrace).
3. **Storage** : stockage dans Qdrant et SQLite avec embeddings vectoriels.
4. **Agents IA** :
   - Context Agent → enrichit le log avec le contexte.
   - Diagnosis Agent → identifie la cause probable.
   - Fixer Agent → propose un correctif ou action concrète.
   - Reporter Agent → rédige résumé et envoie mail ou dashboard.
5. **Dashboard Streamlit** : exploration logs, rapports, chat IA interactif.
6. **Recherche sémantique** : retrouver erreurs similaires via vector DB.
7. **Notification automatique** : envoi mail en cas d’erreur.

### Consignes pour générer le PDR :
- Rédige un **PDR complet et structuré**, contenant :
  1. Résumé du projet
  2. Objectifs détaillés
  3. Stack technique complète
  4. Architecture logicielle et modules
  5. Pipeline multi-agents
  6. Data flow complet (diagramme ASCII inclus)
  7. MVP (fonctionnalités minimales)
  8. Roadmap / phases de réalisation (sprints, tâches, durée)
  9. Dépendances principales (`requirements.txt`)
 10. Livrables attendus
- Fournis un **diagramme ASCII** clair représentant l’architecture et le flux de données.
- Décris **chaque module et agent avec ses responsabilités et interactions**.
- Donne une **roadmap détaillée par sprint** (1 à 4 semaines), avec tâches et priorités.
- Sois précis, structuré et exploitable par un développeur pour commencer le projet immédiatement.
- Conserve la contrainte **full Python + local-first + multi-agents IA + stockage vectoriel + dashboard interactif**.
