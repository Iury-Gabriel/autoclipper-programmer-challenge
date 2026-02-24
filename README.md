# Desafio Técnico — Autoclipper

Bem-vindo(a) ao desafio técnico do [Autoclipper](https://autoclipper.com.br)! Somos uma plataforma de criação automatizada de clips para criadores de conteúdo, e estamos buscando programadores talentosos para se juntar ao nosso time.

## O desafio

Construa um **editor de vídeo simples com capacidade de renderização**, composto por um frontend (editor visual) e um backend (API + renderização). O diferencial deste desafio está em como você pensa **escalabilidade** — queremos ver não só código funcional, mas uma proposta clara de como essa arquitetura suportaria centenas ou milhares de renders simultâneos.

| Área | Descrição | Tempo estimado |
|------|-----------|----------------|
| [Editor + Renderizador de Vídeo](./desafio/) | Frontend (editor) + Backend (API + render) | 4–6 horas |

## Instruções gerais

1. Faça um **fork** deste repositório.
2. Implemente a solução do desafio.
3. Inclua um `README.md` dentro da pasta do desafio explicando:
   - Decisões técnicas que você tomou (incluindo escolha de stack)
   - **Proposta de escalabilidade** (seção obrigatória — veja detalhes no desafio)
   - O que você faria diferente com mais tempo
4. Inclua um `Dockerfile` e `docker-compose.yml` funcionais — o projeto **deve** rodar com `docker-compose up`.
5. Envie o link do repositório para nós quando finalizar.

> **Stack livre**: você pode usar qualquer linguagem, framework ou biblioteca. O único requisito técnico obrigatório é que o projeto rode com `docker-compose up`.

## Critérios gerais de avaliação

- **Funcionalidade**: a solução resolve o problema proposto?
- **Qualidade de código**: organização, legibilidade, nomes significativos.
- **Testes**: cobertura dos casos principais e edge cases.
- **Decisões técnicas**: justificativas claras e pragmáticas.
- **Escalabilidade**: a proposta é realista e demonstra entendimento dos gargalos?
- **Git**: commits atômicos com mensagens descritivas.

## Nossa stack

| Camada | Tecnologias |
|--------|-------------|
| **Frontend** | React 18 + TypeScript, Vite, Apollo Client (GraphQL), Tailwind CSS, shadcn/ui, Remotion |
| **Backend** | Strapi v5 (Node.js), GraphQL + REST, JWT auth |
| **Workers** | BullMQ + Redis, Node.js + Python (OpenCV, PyTorch, NLTK) |
| **Banco de dados** | PostgreSQL |
| **Infra** | Docker, Google Cloud Run, AWS S3, GitHub Actions CI/CD |

Boa sorte! Estamos ansiosos para ver sua solução.
