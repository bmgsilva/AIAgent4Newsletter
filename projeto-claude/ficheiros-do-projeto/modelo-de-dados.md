# Modelo de dados do Painel

Toda a informação da newsletter vive na base de dados do **Painel da Newsletter** (um artifact do Claude com a capacidade `db`). O painel, a conversa no Projeto e as tarefas agendadas leem e escrevem os mesmos documentos através da ferramenta `ArtifactData`, usando sempre o URL do painel.

Regra de acesso: só o dono do painel (o admin) e editores convidados leem e escrevem. Os emails dos destinatários nunca ficam visíveis a outras pessoas.

## `config/settings` — configuração (1 documento)

| Campo | Tipo | Exemplo | Notas |
|---|---|---|---|
| `newsletterName` | texto | `Radar IA` | Nome que aparece no cabeçalho e no assunto |
| `senderName` | texto | `Prof. Ricardo Alves` | Assinatura |
| `adminEmail` | texto | `ricardo@gmail.com` | Recebe as pré-visualizações e avisos |
| `audience` | texto | `Alunos de pós-graduação em IA` | Orienta o tom e a seleção |
| `active` | booleano | `true` | `false` = newsletter em pausa (nada é preparado nem enviado) |
| `template` | texto | `briefing` | `briefing`, `cartoes` ou `carta` |
| `prepareSchedule` | texto | `Quinta 18:00` | Apenas informativo; a hora real está na tarefa agendada |
| `sendSchedule` | texto | `Sexta 09:00` | Apenas informativo; idem |
| `timezone` | texto | `Europe/Lisbon` | |
| `focusNotes` | texto | `Mais regulação europeia, menos rondas de investimento` | Preferências editoriais livres do admin |
| `updatedAt` | ISO 8601 | | |

## `config/editorial` — regras editoriais (1 documento)

| Campo | Tipo | Notas |
|---|---|---|
| `rules` | texto (Markdown) | Conteúdo inicial do ficheiro `regras-editoriais.md`. O admin pode pedir ao Claude para o ajustar. |
| `updatedAt` | ISO 8601 | |

## `templates/<id>` — templates de email

Ids: `briefing`, `cartoes`, `carta` (e outras que o admin peça).

| Campo | Tipo | Notas |
|---|---|---|
| `name` | texto | Ex.: `Briefing Executivo` |
| `description` | texto | Uma linha |
| `html` | texto | HTML com marcadores `{{…}}` (ver ficheiros `template-*.html`) |
| `updatedAt` | ISO 8601 | |

## `recipients/<email>` — destinatários (1 documento por pessoa)

O id do documento é o email em minúsculas (ex.: `ana.costa@exemplo.pt`).

| Campo | Tipo | Notas |
|---|---|---|
| `email` | texto | |
| `name` | texto | Opcional |
| `group` | texto | Opcional, ex.: `Turma 2026` |
| `active` | booleano | `false` = já não recebe (ex.: pediu para sair) |
| `addedAt` | ISO 8601 | |
| `removedAt` | ISO 8601 | Preenchido quando passa a inativo |
| `removedReason` | texto | Ex.: `Respondeu SAIR` |

## `editions/<AAAA-Www>` — edições (1 documento por semana)

O id é a semana ISO da data de envio, ex.: `2026-W43`.

| Campo | Tipo | Notas |
|---|---|---|
| `week` | texto | `2026-W43` |
| `number` | inteiro | Nº sequencial da edição (1, 2, 3…) |
| `status` | texto | Ver estados abaixo |
| `template` | texto | Template usada para gerar o `html` |
| `subject` | texto | Assunto do email |
| `preheader` | texto | Linha de pré-visualização (até ~90 caracteres) |
| `intro` | texto | 1–2 frases de abertura |
| `numberOfWeek` | objeto | `{ value, label, url }` — o "Número da semana" |
| `items` | lista | 5 notícias (ver abaixo) |
| `discussionQuestion` | texto | "Para discutir" — pergunta para a aula |
| `html` | texto | Email final, pronto a enviar |
| `adminNotes` | texto | Pedido de alterações do admin |
| `createdAt` / `approvedAt` / `sentAt` | ISO 8601 | |
| `approvedVia` | texto | `painel`, `conversa` ou `email` |
| `recipientCount` | inteiro | Preenchido após envio |
| `previewThreadId` | texto | Fio Gmail da pré-visualização enviada ao admin |

Cada elemento de `items`:

```json
{
  "section": "geral",          // "geral" (3) ou "auto" (2)
  "title": "…",
  "summary": "…",              // 2 frases, factual
  "whyItMatters": "…",         // 1 frase: porque importa para profissionais
  "source": "Reuters",
  "url": "https://…",
  "publishedAt": "2026-10-20"
}
```

### Estados de uma edição

```
pending ──► approved ──► sent
   │  ▲
   │  └── (Claude refaz) ◄── changes_requested
   └──► skipped        (admin decidiu saltar esta semana, ou não foi aprovada a tempo)
```

| Estado | Significado | Quem muda |
|---|---|---|
| `pending` | Preparada, à espera de aprovação | Tarefa "Preparar" |
| `changes_requested` | Admin pediu alterações (ver `adminNotes`) | Painel ou conversa |
| `approved` | Aprovada, será enviada à hora marcada | Painel, conversa ou resposta "APROVO" por email |
| `sent` | Enviada | Tarefa "Enviar" (ou "envia agora" na conversa) |
| `skipped` | Não enviada | Admin, ou tarefa "Enviar" se não estava aprovada |
