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
