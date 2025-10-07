# AI Ops Copilot - Documentation

## 📋 Plan d'Action - Structure du Projet

Ce projet est organisé selon une structure modulaire pour faciliter le développement et la maintenance.

## 📁 Structure des Fichiers

```
AI-Log-Analyst/
├── ActionPlans/           # Plans détaillés par sprint
│   ├── Sprint1_Fondations.md
│   ├── Sprint2_Parsing.md
│   ├── Sprint3_Agents.md
│   ├── Sprint4_Interface.md
│   ├── Sprint5_Qdrant.md
│   └── Sprint6_Production.md
├── docs/                  # Documentation technique
│   ├── Architecture.md
│   ├── TechnicalStack.md
│   ├── MVP.md
│   └── Requirements.md
├── src/                   # Code source
│   ├── collector/
│   ├── parser/
│   ├── agents/
│   ├── storage/
│   └── ui/
├── config/               # Fichiers de configuration
│   ├── requirements.txt
│   └── config.yaml
├── tests/               # Tests automatisés
├── CONTEXT/            # Contexte et prompts
├── ProgressLog.md      # Journal de progression
└── README.md          # Ce fichier
```

## 🎯 Objectif

**AI Ops Copilot** est un système multi-agents local-first qui analyse les logs d'applications backend pour détecter les erreurs et proposer des solutions automatisées.

## 🚀 Démarrage Rapide

1. **Consulter le PDR complet** : `PDR_AI_Ops_Copilot.md`
2. **Suivre la roadmap** : Dossier `ActionPlans/`
3. **Vérifier l'architecture** : `docs/Architecture.md`
4. **Installer les dépendances** : `config/requirements.txt`

## 📊 Suivi de Progression

Le fichier `ProgressLog.md` contient le journal détaillé de l'avancement du projet avec :
- État actuel de chaque sprint
- Tâches complétées et en cours
- Blocages et résolutions
- Métriques de performance

## 🔗 Liens Utiles

- [Architecture Détaillée](docs/Architecture.md)
- [Stack Technique](docs/TechnicalStack.md)
- [MVP](docs/MVP.md)
- [Sprint 1 - Fondations](ActionPlans/Sprint1_Fondations.md)

---
*Dernière mise à jour : 07/01/2025*