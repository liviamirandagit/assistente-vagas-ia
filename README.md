# 🤖 Assistente Multiagente para Busca de Vagas com IA
### 💻 Automação Inteligente e Análise Estratégica de Oportunidades Profissionais

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenRouter_API-LLM-7C3AED?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Node.js-Scraping-339933?style=for-the-badge&logo=nodedotjs&logoColor=white" />
  <img src="https://img.shields.io/badge/Firecrawl-Web_Scraping-FF4500?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Zed_IDE-Development-000000?style=for-the-badge" />
</p>

---

## 📌 Visão Geral
O **Assistente de Vagas IA** é um sistema multiagente desenvolvido para automatizar a busca, raspagem e análise de compatibilidade de vagas de emprego no mercado de tecnologia. 

A partir do mapeamento do perfil técnico do candidato por meio de um quiz estruturado, o ecossistema orquestra agentes autônomos para coletar dados em tempo real na web, comparar com as habilidades do usuário e gerar relatórios estratégicos de aderência e *gaps* de competência.

---

## 📸 Demonstração do Projeto
<img width="786" height="636" alt="image" src="https://github.com/user-attachments/assets/491ec638-e450-4d0e-a278-721384e73f46" />
<img width="786" height="636" alt="image" src="https://github.com/user-attachments/assets/20b8852d-414d-4f42-a184-4b6c017b528b" />
<img width="786" height="636" alt="image" src="https://github.com/user-attachments/assets/77deeaea-55c4-4ecf-a052-db1501da8f3e" />
<img width="786" height="636" alt="image" src="https://github.com/user-attachments/assets/bad4e342-415b-48a2-9e96-6b5952979737" />
<img width="792" height="239" alt="image" src="https://github.com/user-attachments/assets/a147eee0-5e82-4d91-a65e-08d647643e05" />

<p align="center">
  <img src="COLE_O_LINK_DA_SUA_IMAGEM_AQUI" alt="Demonstração do Assistente Multiagente" width="100%">
</p>

---

## 📐 Arquitetura dos Agentes

```text
┌─────────────────────────┐      ┌─────────────────────────┐      ┌─────────────────────────┐
│   1. QUIZ DE PERFIL     │ ───► │   2. AGENTE MAESTRO     │ ───► │    3. AGENTE SCOUT      │
│  (Mapeamento Técnico)   │      │  (Orquestrador / LLM)   │      │ (Firecrawl + Node.js)   │
└─────────────────────────┘      └─────────────────────────┘      └─────────────────────────┘
                                                                               │
                                                                               ▼
┌─────────────────────────┐                                       ┌─────────────────────────┐
│  5. RELATÓRIO DE DE-PARA│ ◄──────────────────────────────────── │  4. MECANISMO FALLBACK  │
│ (% Fit + Skill Gaps)    │                                       │ (Resiliência de Extração)│
└─────────────────────────┘                                       └─────────────────────────┘
