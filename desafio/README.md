# Desafio Fullstack — Editor e Renderizador de Vídeo

## Contexto

No Autoclipper, criadores de conteúdo editam e renderizam clips de vídeo diretamente na plataforma. Isso envolve um **editor visual** no frontend (onde o usuário monta a composição) e um **backend** que recebe essa composição e renderiza o vídeo final. O maior desafio técnico é a **renderização em escala** — cada render consome CPU, memória e tempo, e precisamos servir milhares de usuários simultaneamente.

Neste desafio, você vai construir uma versão simplificada desse sistema e propor como escalá-lo para produção.

---

## Parte 1 — Frontend: Editor de Composição

Construa um editor visual simples onde o usuário monta uma composição de vídeo.

### 1.1 Timeline

- Renderize uma barra de timeline horizontal representando a duração total da composição (ex: 30 segundos).
- A barra deve ser responsiva e se adaptar à largura do container.
- Exiba marcadores de tempo na timeline (a cada 5 segundos, por exemplo).

### 1.2 Adição de elementos

O usuário deve poder adicionar pelo menos **dois tipos de elementos** à composição:

- **Texto**: título ou legenda com conteúdo editável.
- **Imagem/Cor de fundo**: uma imagem via URL ou uma cor sólida como fundo de um trecho.

Cada elemento deve ter:
- Tempo de início e fim (posição na timeline).
- As propriedades editáveis relevantes (texto, cor, tamanho, posição, etc.).

### 1.3 Preview

- Exiba uma **área de preview** que mostra como a composição vai ficar no momento atual da timeline.
- O usuário deve poder navegar pela timeline (clicar ou arrastar um playhead) e ver o preview atualizar.
- **Bônus**: implementar play/pause que percorre a timeline automaticamente.

### 1.4 Exportação da composição

- Ao clicar em "Renderizar", o frontend deve enviar a composição para o backend como um JSON estruturado.
- Exemplo de payload:
  ```json
  {
    "durationInSeconds": 30,
    "fps": 30,
    "width": 1920,
    "height": 1080,
    "elements": [
      {
        "type": "text",
        "content": "Bem-vindo ao Autoclipper!",
        "startInSeconds": 0,
        "endInSeconds": 5,
        "style": { "fontSize": 48, "color": "#FFFFFF", "x": 100, "y": 200 }
      },
      {
        "type": "solid",
        "color": "#1E40AF",
        "startInSeconds": 0,
        "endInSeconds": 10
      }
    ]
  }
  ```

---

## Parte 2 — Backend: API + Renderização

Construa uma API que recebe composições do editor e renderiza vídeos.

### 2.1 Endpoint de criação de render

`POST /renders`

- Recebe o JSON da composição (payload do frontend).
- Valida o payload (duração, elementos, etc.).
- Salva o projeto no banco de dados com status `queued`.
- Enfileira um job de renderização.
- Retorna `{ id: string, status: "queued" }`.

### 2.2 Fila de renderização

- Use **BullMQ + Redis** para gerenciar a fila de renders.
- O worker deve:
  - Pegar o job da fila.
  - Renderizar o vídeo a partir da composição. Você pode:
    - **(A)** Usar [Remotion](https://www.remotion.dev/) para renderizar server-side (ideal).
    - **(B)** Simular a renderização com um `sleep` proporcional à duração do vídeo (aceitável — foque na arquitetura).
  - Atualizar o status no banco (`rendering` → `completed` ou `failed`).
  - Salvar o arquivo de saída (pode ser local, ex: `/tmp/renders/{id}.mp4`).

### 2.3 Endpoint de status

`GET /renders/:id`

- Retorna o estado atual do render:
  ```json
  {
    "id": "abc-123",
    "status": "completed",
    "createdAt": "2025-01-15T10:30:00Z",
    "completedAt": "2025-01-15T10:31:45Z",
    "outputUrl": "/renders/abc-123.mp4"
  }
  ```
- Status possíveis: `queued`, `rendering`, `completed`, `failed`.

### 2.4 Resiliência

- **Retry**: 3 tentativas com backoff exponencial para renders que falharem.
- **Concorrência**: limite de workers concorrentes (ex: 2 por instância).
- **Timeout**: se um render exceder um tempo máximo (ex: 5 minutos), marcar como `failed`.

---

## Requisitos técnicos

- **Stack livre**: você pode usar qualquer linguagem, framework ou biblioteca que preferir — tanto no frontend quanto no backend. Escolha as ferramentas que você domina e que fazem sentido para o problema.
- **Único requisito obrigatório**: o projeto deve rodar com `docker-compose up`. Inclua um `Dockerfile` e um `docker-compose.yml` funcionais na raiz do seu projeto. O avaliador não vai instalar dependências manualmente — se não rodar com Docker Compose, não será avaliado.

---

## Parte 3 — Proposta de Escalabilidade (Obrigatória)

Esta é a parte mais importante do desafio para candidatos sênior. No `README.md` da sua solução, inclua uma seção **"Proposta de Escalabilidade"** abordando:

### Perguntas que esperamos que você responda:

1. **Gargalos**: Quais são os gargalos do sistema atual? O que quebra primeiro quando o número de usuários cresce?

2. **Escala horizontal**: Como você escalaria os workers de renderização? Considere:
   - Múltiplas instâncias de workers
   - Auto-scaling baseado no tamanho da fila
   - Separação de recursos entre API e workers

3. **Armazenamento**: Como gerenciar os vídeos renderizados em escala?
   - Armazenamento local vs. object storage (S3, GCS)
   - CDN para distribuição
   - Política de retenção/limpeza

4. **Custo**: Renderizar vídeo é caro. Como otimizar custos?
   - Spot/preemptible instances
   - Priorização de filas (usuários pagos vs. free)
   - Caching de renders idênticos

5. **Observabilidade**: Como monitorar o sistema em produção?
   - Métricas (tempo de render, tamanho da fila, taxa de falha)
   - Alertas
   - Logs estruturados

### Formato esperado

Não precisa ser um documento extenso. Pode ser uma lista de tópicos com 2–3 parágrafos cada, ou um diagrama ASCII com explicações. O que importa é demonstrar que você **entende os problemas reais** de escalar um sistema de renderização de vídeo.

---

## Testes

Escreva testes para os componentes mais críticos:

- **Frontend**: teste do componente de timeline ou da serialização da composição.
- **Backend**: teste do endpoint de criação de render e da lógica do worker (pode usar render simulado nos testes).
- **Integração** (bônus): teste end-to-end que cria uma composição e verifica que o render é concluído.

---

## Critérios de avaliação

| Critério | Peso | Descrição |
|----------|------|-----------|
| **Funcionalidade** | Alto | O editor funciona? O render é executado e o status é reportado? |
| **Proposta de escalabilidade** | Alto | A proposta é realista, pragmática e demonstra profundidade técnica? |
| **Qualidade de código** | Médio | Organização, tipagem, separação de responsabilidades |
| **UX do editor** | Médio | O editor é intuitivo e usável? |
| **Testes** | Médio | Cobertura dos cenários principais |
| **Resiliência** | Médio | Retry, dead-letter, timeout funcionam? |
| **DevX** | Baixo | O projeto roda com `docker-compose up` sem fricção? |

---

## Dicas

- **Use a stack que você domina.** Não existe resposta certa — queremos ver você no seu melhor. Python, Go, Ruby, Node, Rust... escolha o que fizer sentido.
- **Comece pelo mais simples que funciona.** Um editor com 2 inputs e um botão de render que funciona é melhor que um editor elaborado que não renderiza.
- **A proposta de escalabilidade vale tanto quanto o código.** Mesmo que sua implementação seja simples, uma proposta bem pensada mostra maturidade técnica.
- **Garanta que o Docker Compose funciona.** Esse é o único requisito técnico obrigatório. Se o avaliador rodar `docker-compose up` e não funcionar, o desafio não será avaliado.

## Tempo estimado

4–6 horas.
