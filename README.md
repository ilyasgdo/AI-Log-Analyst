# AI-Log-Analyst
🎯 Objectif global

Créer un système entièrement local qui :

S’abonne aux logs (app backend, serveur, proxy Cloudflared, etc.)

Détecte automatiquement les erreurs ou anomalies dans ces logs

Envoie les logs problématiques à un ensemble d’agents IA :

🧩 Agent Context → Comprend le contexte (app, endpoint, stack, requête, etc.)

🧠 Agent Diagnosis → Analyse la cause probable (bug code, config, infra, etc.)

🛠 Agent Fixer → Génère une solution / fix ou une procédure

✉️ Agent Reporter → Rédige un mail + résumé clair + ajoute au dashboard

Stocke tout (logs + analyses + solutions) dans une base vectorielle (Qdrant)

Fournit une interface Streamlit pour explorer les logs, poser des questions (“Quels problèmes sont récurrents ?”, “Quelle erreur a eu lieu à 14h30 ?”), et visualiser les suggestions..

🧱 Stack technique complète (Full Python, local-first)
Catégorie	Outils / Librairies
Langage principal	Python 3.11+
IA & Agents	LangChain / CrewAI (pour orchestrer les agents)
LLM local	Ollama (Llama 3 / Mistral / Deepseek)
Vector DB	Qdrant (local)
Ingestion logs	File watcher (watchdog) + Cloudflared log stream
Parsing logs	Regex / JSON parser / Custom parser
Dashboard UI	Streamlit
Base de données (option)	SQLite pour métadonnées (timestamps, status, etc.)
Emailing	smtplib + Gmail API (option)
Orchestration multi-agent	CrewAI / LangGraph
Monitoring / Testing	pytest + logging lib interne
🧩 Architecture logicielle (conceptuelle)
                ┌──────────────────────────┐
                │ Cloudflared / App Logs   │
                └────────────┬─────────────┘
                             │ (watcher)
                             ▼
                ┌──────────────────────────┐
                │ Log Collector & Parser   │
                │ - format JSON / raw      │
                │ - enrichit avec requête  │
                └────────────┬─────────────┘
                             │
                ┌──────────────────────────┐
                │ Qdrant (Vector DB)       │
                │ Stocke logs + embeddings │
                └────────────┬─────────────┘
                             │
       ┌────────────────────────────────────────────┐
       │             AI Agents System               │
       │────────────────────────────────────────────│
       │ 1️⃣ Agent Context → enrichit log (stack, req, app) │
       │ 2️⃣ Agent Diagnosis → trouve cause probable        │
       │ 3️⃣ Agent Fixer → propose correctif ou action      │
       │ 4️⃣ Agent Reporter → génère résumé + mail + UI     │
       └────────────────────────────────────────────┘
                             │
                             ▼
                ┌──────────────────────────┐
                │ Streamlit Dashboard       │
                │ - liste logs              │
                │ - recherche sémantique    │
                │ - chat avec IA            │
                │ - visualisation erreurs   │
                └──────────────────────────┘

🧩 MVP Roadmap (4 sprints / 4 semaines)
🏁 Sprint 1 — Core log collector & parser

Objectif : Collecter et structurer les logs Cloudflared/local.

Étapes :

 Configurer Cloudflared pour streamer les logs sur un port local (ou fichier).

 Créer un watcher Python (via watchdog) qui surveille les logs.

 Parser chaque log (timestamp, app, req, niveau, message, stacktrace).

 Stocker les logs dans Qdrant avec embedding (texte complet du log).

 Interface Streamlit simple pour afficher les logs.

🧰 Modules :

collector.py → ingestion logs

parser.py → extraction context

storage.py → push vers Qdrant

⚙️ Sprint 2 — Agents IA (analyse et diagnostic)

Objectif : Ajouter pipeline multi-agents pour analyser les erreurs.

Étapes :

 Créer l’agent Context → enrichit les logs (API, user, endpoint, app).

 Créer l’agent Diagnosis → identifie type d’erreur (backend, DB, etc.).

 Créer l’agent Fixer → propose un correctif précis.

 Créer un orchestrateur (CrewAI ou LangGraph) pour exécution séquentielle.

 Sauvegarder résultats dans Qdrant avec métadonnées (status=resolved).

🧰 Modules :

agents/context_agent.py

agents/diagnosis_agent.py

agents/fixer_agent.py

📧 Sprint 3 — Notification & Reporting

Objectif : Automatiser le reporting et la communication.

Étapes :

 Créer l’agent Reporter → rédige un résumé clair du problème + solution.

 Intégrer envoi mail (SMTP / Gmail API).

 Ajouter une vue Streamlit “Reports” (liste des incidents résolus).

 Ajouter recherche sémantique : “montre les erreurs similaires à X”.

🧰 Modules :

agents/reporter_agent.py

mail_service.py

dashboard.py

💬 Sprint 4 — Dashboard IA + Query intelligente

Objectif : Exploration et interaction IA.

Étapes :

 Intégrer un chat Streamlit relié au LLM via LangChain RAG.

 Ajouter filtres de logs (par app, niveau, temps, erreur).

 Intégrer graphe des erreurs récurrentes.

 Optimiser pipeline : logs → embedding → agents → reporting.

🧰 Modules :

ui/app.py

query_engine.py

analytics.py

🔁 Extension possible (v2)

🔧 Auto-correction : exécuter des scripts si l’erreur est récurrente.

📈 Tableau de bord temps réel (WebSocket)

🤖 RL-based agent : apprend quelles solutions fonctionnent vraiment.

🔐 Security layer (analyse anomalies réseau + alertes IA)

✅ MVP minimal viable

 Collecte des logs Cloudflared/local

 Détection auto des erreurs (niveau “error” ou “exception”)

 Agents IA context + diagnostic + solution

 Envoi mail automatique + stockage dans Qdrant

 Dashboard Streamlit (consultation + recherche IA)
