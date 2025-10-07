# Sprint 4 : Interface et Notifications - Plan d'Action Détaillé

## 📋 Vue d'Ensemble

**Durée** : 2 semaines  
**Objectif** : Développer l'interface utilisateur Streamlit et le système de notifications  
**Équipe** : 1-2 développeurs  
**Priorité** : Critique  
**Prérequis** : Sprint 3 terminé (agents IA opérationnels)

## 🎯 Objectifs du Sprint

### Objectifs Principaux
1. **Interface Streamlit** : Dashboard moderne et intuitif
2. **Visualisation temps réel** : Monitoring des erreurs et diagnostics
3. **Système de notifications** : Email, Slack, Teams
4. **Gestion des alertes** : Configuration et escalade
5. **Rapports automatisés** : Génération et export

### Objectifs Secondaires
- Interface responsive et accessible
- Thème sombre/clair
- Authentification basique
- API REST pour intégrations
- Système de feedback utilisateur

## 📦 Livrables

### Interface Utilisateur
- [ ] Dashboard principal avec métriques
- [ ] Page de monitoring temps réel
- [ ] Interface de configuration
- [ ] Système de rapports
- [ ] Gestion des notifications

### Système de Notifications
- [ ] Moteur de notifications multi-canal
- [ ] Templates configurables
- [ ] Système d'escalade
- [ ] Historique des notifications
- [ ] Intégrations (Email, Slack, Teams)

### API et Intégrations
- [ ] API REST pour données
- [ ] Webhooks pour notifications
- [ ] Export de données (CSV, JSON, PDF)
- [ ] Intégration avec outils externes

## 🗓️ Planning Détaillé

### Semaine 1 : Interface Streamlit

#### Jour 1-2 : Architecture Interface
**Tâches :**
- [ ] Structure modulaire Streamlit
- [ ] Système de navigation
- [ ] Composants réutilisables
- [ ] Gestion d'état global

**Structure Interface :**
```python
# src/ui/app.py
import streamlit as st
import asyncio
from datetime import datetime, timedelta
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from typing import Dict, Any, List

# Configuration de la page
st.set_page_config(
    page_title="AI Ops Copilot",
    page_icon="🤖",
    layout="wide",
    initial_sidebar_state="expanded"
)

class StreamlitApp:
    """Application Streamlit principale"""
    
    def __init__(self):
        self.init_session_state()
        self.load_custom_css()
    
    def init_session_state(self):
        """Initialise l'état de session Streamlit"""
        if 'current_page' not in st.session_state:
            st.session_state.current_page = 'dashboard'
        
        if 'auto_refresh' not in st.session_state:
            st.session_state.auto_refresh = True
        
        if 'refresh_interval' not in st.session_state:
            st.session_state.refresh_interval = 30
        
        if 'theme' not in st.session_state:
            st.session_state.theme = 'light'
    
    def load_custom_css(self):
        """Charge les styles CSS personnalisés"""
        st.markdown("""
        <style>
        .main-header {
            font-size: 2.5rem;
            font-weight: bold;
            color: #1f77b4;
            text-align: center;
            margin-bottom: 2rem;
        }
        
        .metric-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 1rem;
            border-radius: 10px;
            color: white;
            text-align: center;
            margin: 0.5rem 0;
        }
        
        .error-card {
            background: #ff6b6b;
            padding: 1rem;
            border-radius: 8px;
            color: white;
            margin: 0.5rem 0;
        }
        
        .success-card {
            background: #51cf66;
            padding: 1rem;
            border-radius: 8px;
            color: white;
            margin: 0.5rem 0;
        }
        
        .warning-card {
            background: #ffd43b;
            padding: 1rem;
            border-radius: 8px;
            color: #333;
            margin: 0.5rem 0;
        }
        
        .sidebar-logo {
            display: block;
            margin: 0 auto 2rem auto;
            width: 150px;
        }
        
        .status-indicator {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            display: inline-block;
            margin-right: 8px;
        }
        
        .status-online { background-color: #51cf66; }
        .status-warning { background-color: #ffd43b; }
        .status-offline { background-color: #ff6b6b; }
        </style>
        """, unsafe_allow_html=True)
    
    def render_sidebar(self):
        """Rendu de la barre latérale"""
        with st.sidebar:
            st.markdown('<div class="sidebar-logo">🤖 AI Ops Copilot</div>', unsafe_allow_html=True)
            
            # Navigation
            pages = {
                'dashboard': '📊 Dashboard',
                'monitoring': '🔍 Monitoring',
                'diagnostics': '🩺 Diagnostics',
                'notifications': '📧 Notifications',
                'reports': '📋 Rapports',
                'settings': '⚙️ Paramètres'
            }
            
            selected_page = st.selectbox(
                "Navigation",
                options=list(pages.keys()),
                format_func=lambda x: pages[x],
                index=list(pages.keys()).index(st.session_state.current_page)
            )
            
            if selected_page != st.session_state.current_page:
                st.session_state.current_page = selected_page
                st.rerun()
            
            st.divider()
            
            # Statut système
            st.subheader("Statut Système")
            self.render_system_status()
            
            st.divider()
            
            # Paramètres rapides
            st.subheader("Paramètres")
            st.session_state.auto_refresh = st.checkbox(
                "Actualisation auto", 
                value=st.session_state.auto_refresh
            )
            
            if st.session_state.auto_refresh:
                st.session_state.refresh_interval = st.slider(
                    "Intervalle (sec)", 
                    min_value=10, 
                    max_value=300, 
                    value=st.session_state.refresh_interval
                )
            
            # Thème
            theme_options = {'light': '☀️ Clair', 'dark': '🌙 Sombre'}
            st.session_state.theme = st.selectbox(
                "Thème",
                options=list(theme_options.keys()),
                format_func=lambda x: theme_options[x],
                index=list(theme_options.keys()).index(st.session_state.theme)
            )
    
    def render_system_status(self):
        """Rendu du statut système"""
        # Simulation des statuts (à remplacer par vraies données)
        services = {
            'Ollama': 'online',
            'Base de données': 'online',
            'Collecteur logs': 'warning',
            'Agents IA': 'online'
        }
        
        for service, status in services.items():
            status_class = f"status-{status}"
            st.markdown(
                f'<span class="status-indicator {status_class}"></span>{service}',
                unsafe_allow_html=True
            )
    
    def render_dashboard(self):
        """Rendu du dashboard principal"""
        st.markdown('<h1 class="main-header">Dashboard AI Ops Copilot</h1>', unsafe_allow_html=True)
        
        # Métriques principales
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            st.metric(
                label="Erreurs détectées (24h)",
                value="127",
                delta="-23"
            )
        
        with col2:
            st.metric(
                label="Diagnostics générés",
                value="89",
                delta="12"
            )
        
        with col3:
            st.metric(
                label="Temps résolution moyen",
                value="8.5 min",
                delta="-2.1 min"
            )
        
        with col4:
            st.metric(
                label="Taux de résolution",
                value="94.2%",
                delta="1.8%"
            )
        
        # Graphiques
        col1, col2 = st.columns(2)
        
        with col1:
            st.subheader("Évolution des erreurs")
            self.render_error_timeline()
        
        with col2:
            st.subheader("Répartition par type")
            self.render_error_distribution()
        
        # Erreurs récentes
        st.subheader("Erreurs récentes")
        self.render_recent_errors()
    
    def render_error_timeline(self):
        """Graphique timeline des erreurs"""
        # Données simulées
        dates = pd.date_range(start='2025-01-01', end='2025-01-07', freq='H')
        errors = np.random.poisson(3, len(dates))
        
        df = pd.DataFrame({
            'timestamp': dates,
            'errors': errors
        })
        
        fig = px.line(
            df, 
            x='timestamp', 
            y='errors',
            title="Erreurs par heure",
            color_discrete_sequence=['#ff6b6b']
        )
        
        fig.update_layout(
            xaxis_title="Temps",
            yaxis_title="Nombre d'erreurs",
            showlegend=False
        )
        
        st.plotly_chart(fig, use_container_width=True)
    
    def render_error_distribution(self):
        """Graphique répartition des erreurs"""
        # Données simulées
        categories = ['Database', 'Network', 'Application', 'Security', 'Infrastructure']
        values = [25, 18, 32, 8, 17]
        
        fig = px.pie(
            values=values,
            names=categories,
            title="Répartition par catégorie"
        )
        
        st.plotly_chart(fig, use_container_width=True)
    
    def render_recent_errors(self):
        """Table des erreurs récentes"""
        # Données simulées
        recent_errors = pd.DataFrame({
            'Timestamp': ['2025-01-07 14:32:15', '2025-01-07 14:28:42', '2025-01-07 14:25:18'],
            'Niveau': ['ERROR', 'WARNING', 'ERROR'],
            'Composant': ['database', 'network', 'application'],
            'Message': [
                'Connection timeout to PostgreSQL',
                'High response time detected',
                'NullPointerException in UserService'
            ],
            'Statut': ['Diagnostiqué', 'En cours', 'Nouveau']
        })
        
        # Styling conditionnel
        def style_status(val):
            if val == 'Diagnostiqué':
                return 'background-color: #51cf66; color: white'
            elif val == 'En cours':
                return 'background-color: #ffd43b; color: black'
            else:
                return 'background-color: #ff6b6b; color: white'
        
        styled_df = recent_errors.style.applymap(
            style_status, 
            subset=['Statut']
        )
        
        st.dataframe(styled_df, use_container_width=True)
    
    def run(self):
        """Point d'entrée principal de l'application"""
        self.render_sidebar()
        
        # Routage des pages
        if st.session_state.current_page == 'dashboard':
            self.render_dashboard()
        elif st.session_state.current_page == 'monitoring':
            self.render_monitoring()
        elif st.session_state.current_page == 'diagnostics':
            self.render_diagnostics()
        elif st.session_state.current_page == 'notifications':
            self.render_notifications()
        elif st.session_state.current_page == 'reports':
            self.render_reports()
        elif st.session_state.current_page == 'settings':
            self.render_settings()
        
        # Auto-refresh
        if st.session_state.auto_refresh:
            time.sleep(st.session_state.refresh_interval)
            st.rerun()

if __name__ == "__main__":
    app = StreamlitApp()
    app.run()
```

**Critères d'acceptation :**
- Interface moderne et responsive
- Navigation fluide entre pages
- Composants réutilisables
- Gestion d'état cohérente

#### Jour 3-4 : Dashboard et Monitoring
**Tâches :**
- [ ] Dashboard avec métriques temps réel
- [ ] Page monitoring avancée
- [ ] Graphiques interactifs
- [ ] Filtres et recherche

**Page Monitoring :**
```python
# src/ui/pages/monitoring.py
import streamlit as st
import pandas as pd
import plotly.express as px
import plotly.graph_objects as go
from datetime import datetime, timedelta

class MonitoringPage:
    """Page de monitoring temps réel"""
    
    def __init__(self, data_service):
        self.data_service = data_service
    
    def render(self):
        """Rendu de la page monitoring"""
        st.title("🔍 Monitoring Temps Réel")
        
        # Filtres
        self.render_filters()
        
        # Métriques en temps réel
        self.render_realtime_metrics()
        
        # Graphiques de monitoring
        col1, col2 = st.columns(2)
        
        with col1:
            self.render_error_heatmap()
        
        with col2:
            self.render_agent_performance()
        
        # Log stream
        self.render_log_stream()
    
    def render_filters(self):
        """Rendu des filtres"""
        col1, col2, col3, col4 = st.columns(4)
        
        with col1:
            time_range = st.selectbox(
                "Période",
                options=['1h', '6h', '24h', '7d'],
                index=2
            )
        
        with col2:
            log_level = st.multiselect(
                "Niveau",
                options=['DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL'],
                default=['ERROR', 'CRITICAL']
            )
        
        with col3:
            components = st.multiselect(
                "Composants",
                options=['database', 'network', 'application', 'security'],
                default=[]
            )
        
        with col4:
            agent_filter = st.selectbox(
                "Agent",
                options=['Tous', 'Database', 'Network', 'Application', 'Security'],
                index=0
            )
        
        return {
            'time_range': time_range,
            'log_level': log_level,
            'components': components,
            'agent_filter': agent_filter
        }
    
    def render_realtime_metrics(self):
        """Métriques temps réel"""
        col1, col2, col3, col4, col5 = st.columns(5)
        
        # Simulation de données temps réel
        current_errors = 23
        avg_response_time = 12.5
        active_agents = 6
        resolution_rate = 89.2
        queue_size = 7
        
        with col1:
            st.metric(
                "Erreurs actives",
                current_errors,
                delta=2,
                delta_color="inverse"
            )
        
        with col2:
            st.metric(
                "Temps réponse (s)",
                f"{avg_response_time:.1f}",
                delta=-1.2
            )
        
        with col3:
            st.metric(
                "Agents actifs",
                active_agents,
                delta=0
            )
        
        with col4:
            st.metric(
                "Taux résolution (%)",
                f"{resolution_rate:.1f}",
                delta=2.3
            )
        
        with col5:
            st.metric(
                "File d'attente",
                queue_size,
                delta=-3
            )
    
    def render_error_heatmap(self):
        """Heatmap des erreurs par heure/jour"""
        st.subheader("Heatmap des erreurs")
        
        # Génération de données simulées
        hours = list(range(24))
        days = ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven', 'Sam', 'Dim']
        
        # Matrice d'erreurs simulée
        import numpy as np
        error_matrix = np.random.poisson(3, (len(days), len(hours)))
        
        fig = go.Figure(data=go.Heatmap(
            z=error_matrix,
            x=hours,
            y=days,
            colorscale='Reds',
            hoverongaps=False
        ))
        
        fig.update_layout(
            title="Erreurs par heure et jour",
            xaxis_title="Heure",
            yaxis_title="Jour",
            height=400
        )
        
        st.plotly_chart(fig, use_container_width=True)
    
    def render_agent_performance(self):
        """Performance des agents"""
        st.subheader("Performance des agents")
        
        # Données simulées
        agents = ['Database', 'Network', 'Application', 'Security', 'Infrastructure', 'Integration']
        response_times = [8.2, 12.5, 15.1, 6.8, 18.3, 11.7]
        success_rates = [94.2, 87.5, 91.8, 96.1, 83.4, 89.6]
        
        fig = go.Figure()
        
        fig.add_trace(go.Scatter(
            x=response_times,
            y=success_rates,
            mode='markers+text',
            text=agents,
            textposition="top center",
            marker=dict(
                size=15,
                color=success_rates,
                colorscale='RdYlGn',
                showscale=True,
                colorbar=dict(title="Taux succès (%)")
            ),
            name="Agents"
        ))
        
        fig.update_layout(
            title="Performance agents (Temps vs Succès)",
            xaxis_title="Temps de réponse (s)",
            yaxis_title="Taux de succès (%)",
            height=400
        )
        
        st.plotly_chart(fig, use_container_width=True)
    
    def render_log_stream(self):
        """Stream des logs en temps réel"""
        st.subheader("Stream des logs")
        
        # Container pour le stream
        log_container = st.container()
        
        with log_container:
            # Simulation de logs récents
            logs = [
                {
                    'timestamp': '2025-01-07 14:35:22',
                    'level': 'ERROR',
                    'component': 'database',
                    'message': 'Connection pool exhausted',
                    'agent': 'DatabaseAgent'
                },
                {
                    'timestamp': '2025-01-07 14:34:18',
                    'level': 'WARNING',
                    'component': 'network',
                    'message': 'High latency detected: 2.5s',
                    'agent': 'NetworkAgent'
                },
                {
                    'timestamp': '2025-01-07 14:33:45',
                    'level': 'ERROR',
                    'component': 'application',
                    'message': 'NullPointerException in UserController',
                    'agent': 'ApplicationAgent'
                }
            ]
            
            for log in logs:
                level_color = {
                    'ERROR': '#ff6b6b',
                    'WARNING': '#ffd43b',
                    'INFO': '#51cf66',
                    'DEBUG': '#74c0fc'
                }.get(log['level'], '#868e96')
                
                st.markdown(f"""
                <div style="
                    border-left: 4px solid {level_color};
                    padding: 10px;
                    margin: 5px 0;
                    background-color: rgba(255,255,255,0.05);
                    border-radius: 4px;
                ">
                    <strong>{log['timestamp']}</strong> 
                    <span style="color: {level_color}; font-weight: bold;">[{log['level']}]</span>
                    <span style="color: #666;">{log['component']}</span><br>
                    {log['message']}<br>
                    <small>Agent: {log['agent']}</small>
                </div>
                """, unsafe_allow_html=True)
```

**Critères d'acceptation :**
- Dashboard temps réel fonctionnel
- Métriques précises et à jour
- Graphiques interactifs
- Filtres et recherche opérationnels

#### Jour 5 : Tests Interface
**Tâches :**
- [ ] Tests composants Streamlit
- [ ] Tests navigation et état
- [ ] Tests responsive design
- [ ] Optimisation performance

### Semaine 2 : Notifications et Intégrations

#### Jour 6-7 : Système de Notifications
**Tâches :**
- [ ] Moteur de notifications multi-canal
- [ ] Templates configurables
- [ ] Système d'escalade
- [ ] Intégrations (Email, Slack, Teams)

**Moteur de Notifications :**
```python
# src/notifications/notification_engine.py
import asyncio
import smtplib
import json
from typing import Dict, Any, List, Optional
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from dataclasses import dataclass
from enum import Enum
import aiohttp
import logging

class NotificationChannel(Enum):
    EMAIL = "email"
    SLACK = "slack"
    TEAMS = "teams"
    WEBHOOK = "webhook"

class NotificationPriority(Enum):
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CRITICAL = 4

@dataclass
class NotificationConfig:
    """Configuration des notifications"""
    
    # Email
    smtp_host: str = "localhost"
    smtp_port: int = 587
    smtp_user: str = ""
    smtp_password: str = ""
    from_email: str = "aiops@company.com"
    
    # Slack
    slack_webhook_url: str = ""
    slack_channel: str = "#alerts"
    
    # Teams
    teams_webhook_url: str = ""
    
    # Général
    enabled_channels: List[NotificationChannel] = None
    escalation_delay: int = 300  # 5 minutes
    max_retries: int = 3

class NotificationTemplate:
    """Templates de notifications"""
    
    EMAIL_TEMPLATES = {
        'error_detected': {
            'subject': '🚨 Erreur détectée - {component}',
            'body': '''
            <h2>Erreur détectée dans le système</h2>
            
            <p><strong>Composant :</strong> {component}</p>
            <p><strong>Niveau :</strong> {level}</p>
            <p><strong>Timestamp :</strong> {timestamp}</p>
            
            <h3>Message d'erreur :</h3>
            <pre>{message}</pre>
            
            <h3>Diagnostic IA :</h3>
            <p><strong>Cause probable :</strong> {diagnosis_cause}</p>
            <p><strong>Solution recommandée :</strong></p>
            <ul>
            {solution_steps}
            </ul>
            
            <p><strong>Confiance :</strong> {confidence}%</p>
            
            <hr>
            <p><small>AI Ops Copilot - {timestamp}</small></p>
            '''
        },
        'error_resolved': {
            'subject': '✅ Erreur résolue - {component}',
            'body': '''
            <h2>Erreur résolue</h2>
            
            <p><strong>Composant :</strong> {component}</p>
            <p><strong>Durée :</strong> {duration}</p>
            <p><strong>Solution appliquée :</strong> {solution}</p>
            
            <hr>
            <p><small>AI Ops Copilot - {timestamp}</small></p>
            '''
        }
    }
    
    SLACK_TEMPLATES = {
        'error_detected': {
            "text": "🚨 Erreur détectée",
            "blocks": [
                {
                    "type": "header",
                    "text": {
                        "type": "plain_text",
                        "text": "🚨 Erreur détectée"
                    }
                },
                {
                    "type": "section",
                    "fields": [
                        {
                            "type": "mrkdwn",
                            "text": "*Composant:*\n{component}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": "*Niveau:*\n{level}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": "*Timestamp:*\n{timestamp}"
                        },
                        {
                            "type": "mrkdwn",
                            "text": "*Confiance:*\n{confidence}%"
                        }
                    ]
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": "*Message:*\n```{message}```"
                    }
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": "*Diagnostic IA:*\n{diagnosis_cause}"
                    }
                },
                {
                    "type": "section",
                    "text": {
                        "type": "mrkdwn",
                        "text": "*Solution recommandée:*\n{solution_summary}"
                    }
                }
            ]
        }
    }

class NotificationEngine:
    """Moteur de notifications multi-canal"""
    
    def __init__(self, config: NotificationConfig):
        self.config = config
        self.templates = NotificationTemplate()
        self.notification_queue = asyncio.Queue()
        self.escalation_tasks = {}
        
    async def send_notification(
        self, 
        notification_type: str,
        data: Dict[str, Any],
        priority: NotificationPriority = NotificationPriority.MEDIUM,
        channels: Optional[List[NotificationChannel]] = None
    ) -> Dict[str, bool]:
        """Envoie une notification sur les canaux spécifiés"""
        
        if channels is None:
            channels = self.config.enabled_channels or [NotificationChannel.EMAIL]
        
        results = {}
        
        # Envoi sur chaque canal
        for channel in channels:
            try:
                if channel == NotificationChannel.EMAIL:
                    success = await self._send_email(notification_type, data)
                elif channel == NotificationChannel.SLACK:
                    success = await self._send_slack(notification_type, data)
                elif channel == NotificationChannel.TEAMS:
                    success = await self._send_teams(notification_type, data)
                elif channel == NotificationChannel.WEBHOOK:
                    success = await self._send_webhook(notification_type, data)
                else:
                    success = False
                
                results[channel.value] = success
                
                # Escalade si critique et échec
                if not success and priority == NotificationPriority.CRITICAL:
                    await self._schedule_escalation(notification_type, data, channel)
                    
            except Exception as e:
                logging.error(f"Erreur envoi notification {channel.value}: {e}")
                results[channel.value] = False
        
        return results
    
    async def _send_email(self, notification_type: str, data: Dict[str, Any]) -> bool:
        """Envoi de notification par email"""
        try:
            template = self.templates.EMAIL_TEMPLATES.get(notification_type)
            if not template:
                logging.error(f"Template email non trouvé: {notification_type}")
                return False
            
            # Préparation du contenu
            subject = template['subject'].format(**data)
            body = template['body'].format(**data)
            
            # Création du message
            msg = MIMEMultipart('alternative')
            msg['Subject'] = subject
            msg['From'] = self.config.from_email
            msg['To'] = data.get('recipient_email', 'admin@company.com')
            
            # Ajout du contenu HTML
            html_part = MIMEText(body, 'html')
            msg.attach(html_part)
            
            # Envoi
            with smtplib.SMTP(self.config.smtp_host, self.config.smtp_port) as server:
                if self.config.smtp_user:
                    server.starttls()
                    server.login(self.config.smtp_user, self.config.smtp_password)
                
                server.send_message(msg)
            
            logging.info(f"Email envoyé: {subject}")
            return True
            
        except Exception as e:
            logging.error(f"Erreur envoi email: {e}")
            return False
    
    async def _send_slack(self, notification_type: str, data: Dict[str, Any]) -> bool:
        """Envoi de notification Slack"""
        try:
            if not self.config.slack_webhook_url:
                logging.warning("URL webhook Slack non configurée")
                return False
            
            template = self.templates.SLACK_TEMPLATES.get(notification_type)
            if not template:
                logging.error(f"Template Slack non trouvé: {notification_type}")
                return False
            
            # Formatage du message
            payload = json.loads(json.dumps(template).format(**data))
            
            # Envoi via webhook
            async with aiohttp.ClientSession() as session:
                async with session.post(
                    self.config.slack_webhook_url,
                    json=payload
                ) as response:
                    success = response.status == 200
                    
                    if success:
                        logging.info("Notification Slack envoyée")
                    else:
                        logging.error(f"Erreur Slack: {response.status}")
                    
                    return success
                    
        except Exception as e:
            logging.error(f"Erreur envoi Slack: {e}")
            return False
    
    async def _schedule_escalation(
        self, 
        notification_type: str, 
        data: Dict[str, Any], 
        failed_channel: NotificationChannel
    ):
        """Programme une escalade en cas d'échec critique"""
        escalation_id = f"{notification_type}_{failed_channel.value}_{data.get('timestamp', '')}"
        
        if escalation_id not in self.escalation_tasks:
            task = asyncio.create_task(
                self._escalate_notification(notification_type, data, failed_channel)
            )
            self.escalation_tasks[escalation_id] = task
    
    async def _escalate_notification(
        self, 
        notification_type: str, 
        data: Dict[str, Any], 
        failed_channel: NotificationChannel
    ):
        """Escalade une notification critique"""
        await asyncio.sleep(self.config.escalation_delay)
        
        # Tentative sur tous les autres canaux
        remaining_channels = [
            ch for ch in self.config.enabled_channels 
            if ch != failed_channel
        ]
        
        escalated_data = {
            **data,
            'escalated': True,
            'original_channel': failed_channel.value
        }
        
        await self.send_notification(
            f"{notification_type}_escalated",
            escalated_data,
            priority=NotificationPriority.CRITICAL,
            channels=remaining_channels
        )
    
    async def start_worker(self):
        """Démarre le worker de traitement des notifications"""
        while True:
            try:
                notification = await self.notification_queue.get()
                await self.send_notification(**notification)
                self.notification_queue.task_done()
            except Exception as e:
                logging.error(f"Erreur worker notifications: {e}")
                await asyncio.sleep(1)
    
    def queue_notification(self, **kwargs):
        """Ajoute une notification à la queue"""
        self.notification_queue.put_nowait(kwargs)
```

**Critères d'acceptation :**
- Notifications multi-canal fonctionnelles
- Templates configurables
- Système d'escalade opérationnel
- Intégrations Slack/Teams/Email

#### Jour 8-9 : API et Intégrations
**Tâches :**
- [ ] API REST pour données
- [ ] Webhooks pour notifications
- [ ] Export de données
- [ ] Documentation API

#### Jour 10 : Tests et Finalisation
**Tâches :**
- [ ] Tests end-to-end interface
- [ ] Tests notifications
- [ ] Tests API
- [ ] Documentation utilisateur

## 🔧 Configuration Technique

### Configuration Streamlit
```yaml
# config/streamlit.yaml
streamlit:
  server:
    port: 8501
    address: "0.0.0.0"
    enableCORS: false
    enableXsrfProtection: true
  
  theme:
    primaryColor: "#1f77b4"
    backgroundColor: "#ffffff"
    secondaryBackgroundColor: "#f0f2f6"
    textColor: "#262730"
  
  ui:
    auto_refresh: true
    refresh_interval: 30
    max_log_entries: 1000
    chart_theme: "streamlit"
```

### Configuration Notifications
```yaml
# config/notifications.yaml
notifications:
  enabled_channels:
    - email
    - slack
  
  email:
    smtp_host: "smtp.gmail.com"
    smtp_port: 587
    smtp_user: "${SMTP_USER}"
    smtp_password: "${SMTP_PASSWORD}"
    from_email: "aiops@company.com"
    recipients:
      - "admin@company.com"
      - "devops@company.com"
  
  slack:
    webhook_url: "${SLACK_WEBHOOK_URL}"
    channel: "#alerts"
    username: "AI Ops Copilot"
    icon_emoji: ":robot_face:"
  
  teams:
    webhook_url: "${TEAMS_WEBHOOK_URL}"
  
  escalation:
    enabled: true
    delay_seconds: 300
    max_retries: 3
    critical_channels:
      - email
      - slack
      - teams
```

## 📊 Métriques de Succès

### Métriques Interface
- **Temps de chargement** : < 3 secondes
- **Responsive design** : Support mobile/tablet
- **Accessibilité** : Score WCAG AA
- **Performance** : > 90 Lighthouse

### Métriques Notifications
- **Taux de livraison** : > 95%
- **Temps d'envoi** : < 30 secondes
- **Escalade fonctionnelle** : 100%
- **Templates configurables** : 100%

### Métriques API
- **Temps de réponse** : < 500ms
- **Disponibilité** : > 99%
- **Documentation** : Complète
- **Sécurité** : Authentification + HTTPS

## 🚨 Risques et Mitigation

### Risques Techniques
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| Performance Streamlit | Moyenne | Moyen | Cache + optimisation |
| Échec notifications | Élevée | Critique | Système d'escalade |
| Surcharge API | Moyenne | Élevé | Rate limiting + cache |

### Risques Fonctionnels
| Risque | Probabilité | Impact | Mitigation |
|--------|-------------|---------|------------|
| UX complexe | Élevée | Moyen | Tests utilisateur |
| Notifications spam | Moyenne | Élevé | Filtres intelligents |

## 📝 Checklist de Fin de Sprint

### Interface Utilisateur
- [ ] Dashboard moderne et fonctionnel
- [ ] Navigation intuitive
- [ ] Monitoring temps réel
- [ ] Responsive design
- [ ] Thèmes clair/sombre

### Système de Notifications
- [ ] Multi-canal (Email, Slack, Teams)
- [ ] Templates configurables
- [ ] Système d'escalade
- [ ] Historique des notifications
- [ ] Configuration flexible

### API et Intégrations
- [ ] API REST documentée
- [ ] Webhooks fonctionnels
- [ ] Export de données
- [ ] Authentification sécurisée
- [ ] Rate limiting

### Tests et Qualité
- [ ] Tests interface > 80% coverage
- [ ] Tests notifications complets
- [ ] Tests API automatisés
- [ ] Documentation utilisateur
- [ ] Guide d'installation

## 🎯 Critères de Passage Sprint 5

Pour passer au Sprint 5, tous ces critères doivent être validés :

1. **Interface complète** : Dashboard et monitoring opérationnels
2. **Notifications** : Multi-canal avec escalade
3. **API fonctionnelle** : Endpoints documentés et sécurisés
4. **Tests validés** : Coverage > 80%
5. **Documentation** : Guide utilisateur complet

---

**🚀 Objectif : Interface utilisateur moderne et système de notifications robuste**

*Dernière mise à jour : 07/01/2025*