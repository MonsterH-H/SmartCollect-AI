# Documentation Technique : SmartCollect AI

Ce document détaille le fonctionnement, l'architecture et les spécifications techniques de la plateforme **SmartCollect AI**.

---

## 1. Vue d'Ensemble
**SmartCollect.ai** est une solution SaaS de gestion du poste client (Accounts Receivable) et de recouvrement intelligent. L'objectif principal est de réduire le **DSO (Days Sales Outstanding)** des entreprises en automatisant les processus de relance et en identifiant les risques de non-paiement grâce à l'IA.

---

<img width="1359" height="756" alt="image" src="https://github.com/user-attachments/assets/f79bf2a8-ec2f-4718-ba64-3562eeb28d83" />
<img width="1357" height="708" alt="image" src="https://github.com/user-attachments/assets/e4a00747-10f4-4707-9409-8ad5485df5c3" />
<img width="1362" height="714" alt="image" src="https://github.com/user-attachments/assets/e6489969-a554-48bc-ac01-599b99e0e094" />
<img width="1358" height="600" alt="image" src="https://github.com/user-attachments/assets/870a3ce9-f9ee-4009-a781-23bc7d93a621" />
<img width="1333" height="755" alt="image" src="https://github.com/user-attachments/assets/4063272b-7d92-4c94-b886-8738ca0e46da" />
<img width="1366" height="604" alt="image" src="https://github.com/user-attachments/assets/1b0dba40-d113-40af-a497-aaa52f20cc53" />
<img width="1366" height="766" alt="image" src="https://github.com/user-attachments/assets/1a4d7c46-22b8-4860-bcb9-2a8aa3533e81" />
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/6d84aabb-c59a-4903-bf82-c7d1d3bad8f1" />





## 2. Architecture Fonctionnelle

L'application est structurée autour de plusieurs piliers majeurs :

### A. Pilotage & Dashboard
*   **KPIs en temps réel :** Calcul automatisé du DSO, du montant total en retard, et du ratio de litiges.
*   **Insights IA :** Analyse proactive du comportement de paiement des clients pour détecter des changements de patterns (ex: ralentissement inhabituel des paiements).
*   **Balance Âgée :** Visualisation par tranches de retard (0-30j, 31-60j, 60j+).

### B. Cycle de Relance (Dunning)
*   **Workflow automatique :** Passage successif par plusieurs étapes : Pré-relance, Relance 1, 2, 3, Mise en demeure, et Contentieux.
*   **Canaux Mixtes :** Gestion centralisée des envois par **Email**, **SMS**, et **LRAR** (via intégration API).
*   **Tonalité Adaptative :** Ajustement automatique du ton du message (Courtois > Professionnel > Ferme > Juridique).

### C. Risque et Scoring
*   **Score de Santé Client :** Note de 0 à 100 calculée sur la base de la ponctualité historique et du volume d'affaires.
*   **Catégories de Risque :** Low, Medium, High, Critical.

### D. Gestion Financière
*   **Import FEC :** Capacité d'ingérer des Fichiers des Écritures Comptables pour synchroniser les données avec la comptabilité.
*   **Réconciliation :** Correspondance automatique entre les paiements reçus (virements) et les factures ouvertes.

---

## 3. Architecture Technique (Stack)

### Frontend
*   **Framework :** React 18 avec TypeScript (typage strict, zéro `any`).
*   **Build Tool :** Vite.
*   **Design System :** Tailwind CSS + shadcn/ui.
*   **Gestion d'état & Cache :** TanStack Query (React Query) pour les données asynchrones.
*   **Navigation :** React Router DOM.
*   **Animations :** Framer Motion.

### Backend (Supabase)
L'intégralité du backend repose sur **Supabase**, utilisant :
*   **Base de données :** PostgreSQL.
*   **Authentification :** Supabase Auth (Email/Password, Google OAuth).
*   **Stockage :** Supabase Storage pour les logos, avatars et documents (RIB, FEC).
*   **Logique Serveur :** Edge Functions (TypeScript/Deno).
*   **Automatisation :** Triggers PostgreSQL et `pg_cron` pour les tâches planifiées.

---

## 4. Modèle de Données (Base de Données)

### Tables Principales
*   **`companies`** : Entités utilisatrices du service.
*   **`profiles`** : Utilisateurs liés aux entreprises.
*   **`clients`** : Portefeuille clients des entreprises (avec stats de paiement : dso, score, etc.).
*   **`invoices`** : Factures émises, leurs échéances, statuts et scores de risque.
*   **`payments`** : Historique des paiements reçus et méthode de réconciliation.
*   **`relances`** : Journal de toutes les actions de relance effectuées (canal, contenu, statut).
*   **`alerts`** : Système de notifications internes (ex: "Paiement reçu", "Facture critique").

---

## 5. Logique Serveur (Edge Functions)

L'application utilise une série de fonctions "Serverless" pour les opérations lourdes ou sécurisées :

1.  **`cron-dunning-cycle`** : Exécutée quotidiennement pour identifier les factures arrivées à échéance et générer les relances prévues par le scénario.
2.  **`erp-webhook`** : Reçoit et traite les données envoyées par des systèmes tiers (ERP) ou expose des données via webhooks sortants.
3.  **`score-invoices`** : Moteur de calcul qui met à jour les scores de risque en fonction du temps qui passe.
4.  **`send-email` / `send-sms`** : Interfaces de communication utilisant **Resend** (Email) et **Twilio** (SMS).
5.  **`parse-fec`** : Analyse des fichiers FEC déposés par l'utilisateur pour en extraire les factures et clients.

---

## 6. Intelligence Artificielle

L'IA intervient à trois niveaux :
1.  **Analyse de Sentiment/Ton :** Pour générer des emails de relance qui maximisent les chances de réponse sans dégrader la relation client.
2.  **Prédiction de Retard :** Analyse statistique des délais de paiement pour prévoir les problèmes de cash-flow avant qu'ils n'arrivent.
3.  **Conseiller Stratégique :** Le module "AI Insights" sur le dashboard propose des actions quotidiennes basées sur les données réelles.

---

## 7. Sécurité
*   **RLS (Row Level Security) :** Chaque ligne de la base de données est protégée. Une entreprise ne peut **jamais** accéder aux données d'une autre entreprise.
*   **Chiffrement :** Données sensibles (clés API tierces) stockées de manière sécurisée ou via les "Secrets" de Supabase.
*   **Audit Trail :** Chaque action de relance et chaque changement de statut de facture est loggé avec un horodatage.

---
*Documentation générée par l'équipe SmartCollect AI - Mars 2026*
