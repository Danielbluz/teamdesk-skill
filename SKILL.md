---
name: teamdesk
description: TeamDesk online database platform expertise for API integrations, formula creation, workflow automation, and database design. Use when working with TeamDesk databases including REST API calls (select, create, update, upsert, delete), writing formulas, configuring webhooks, setting up triggers and actions, designing table relationships (one-to-many, many-to-many), or integrating TeamDesk with n8n workflows and other automation tools.
---

# TeamDesk Skill

TeamDesk is a cloud-based database platform for building custom business applications without coding. This skill covers API integration, formulas, automation, and database design.

## Quick Reference

### REST API Base URL
```
https://www.teamdesk.net/secure/api/v2/{database_id}/{table}/{method}.json
```

### Authentication (Required)
```
Authorization: Bearer {token}
```

- `database_id`: Found in URL after `/db/` (e.g., `https://www.teamdesk.net/secure/db/12345/...` → `12345`)
- `token`: 32-character authorization token from Setup > Database > Integration API
- `table`: Table name (URL-encoded if contains spaces)

> **Security note:** Always use the `Authorization: Bearer` header. Avoid embedding tokens in the URL, as URLs may be logged in server/proxy logs and browser history.

### API Methods Overview

| Method | HTTP | Purpose |
|--------|------|---------|
| select | GET | Query records from table or view |
| retrieve | GET | Get specific record by ID/key |
| create | POST | Create new records |
| update | POST | Update existing records |
| upsert | POST | Create or update (match by column) |
| delete | GET/POST | Delete records |
| describe | GET | Get table/column metadata |
| attachment | GET | Download file attachments |
| create/update/upsert | POST (multipart/related) | Upload file attachments inline |

### Common API Patterns

**Select with filter and pagination:**
```
GET /api/v2/{db_id}/{table}/select.json?filter=[Status]="Active"&column=Name&column=Email&top=100&skip=0&sort=Name//ASC
Authorization: Bearer {token}
```

**Upsert (create or update):**
```
POST /api/v2/{db_id}/{table}/upsert.json?match=External_ID
Authorization: Bearer {token}
Content-Type: application/json
[{"External_ID": "ABC123", "Name": "Updated Name", "Value": 100}]
```

**Select from View:**
```
GET /api/v2/{db_id}/{table}/{view}/select.json?top=500
Authorization: Bearer {token}
```

### File Attachment Upload (multipart/related)

File attachments **can** be uploaded via the REST API using `multipart/related` encoding on Create, Update, or Upsert endpoints. There is **no** separate upload endpoint — the file is sent inline with the record data.

**Protocol:**
1. The JSON payload references the file via `"FieldName": "cid:{content_id}"`
2. The file binary is sent as a separate MIME part with matching `Content-ID`
3. `Content-Type` header must be `multipart/related;boundary={boundary}`
4. For `update.json`, use `@row.id` (NOT `Id`) to identify the record

**HTTP Request format:**
```
POST /secure/api/v2/{db_id}/{table}/update.json
Authorization: Bearer {token}
Content-Type: multipart/related;boundary=part-abc123

--part-abc123
Content-Type: application/json;charset=UTF-8

[{"@row.id": 42, "MyFileColumn": "cid:unique-id-here"}]
--part-abc123
Content-Type: application/pdf
Content-Disposition: attachment;filename*=UTF-8''report.pdf
Content-ID: unique-id-here

{binary file data}
--part-abc123--
```

**Python example (urllib, no external deps):**
```python
import json, uuid, urllib.request, urllib.parse

def upload_attachment(table, row_id, file_column, file_path, filename, token):
    """Upload file to attachment column via multipart/related."""
    content_id = str(uuid.uuid4())
    boundary = f'part-{uuid.uuid4().hex}'

    with open(file_path, 'rb') as f:
        file_data = f.read()

    record = {'@row.id': row_id, file_column: f'cid:{content_id}'}
    json_payload = json.dumps([record], ensure_ascii=False)

    import mimetypes
    mime = mimetypes.guess_type(file_path)[0] or 'application/octet-stream'
    enc_name = urllib.parse.quote(filename)

    body = b''.join([
        f'--{boundary}\r\n'.encode(),
        b'Content-Type: application/json;charset=UTF-8\r\n\r\n',
        json_payload.encode('utf-8'), b'\r\n',
        f'--{boundary}\r\n'.encode(),
        f'Content-Type: {mime}\r\n'.encode(),
        f'Content-Disposition: attachment;filename*=UTF-8\'\'{enc_name}\r\n'.encode(),
        f'Content-ID: {content_id}\r\n\r\n'.encode(),
        file_data, b'\r\n',
        f'--{boundary}--'.encode(),
    ])

    BASE = 'https://www.teamdesk.net/secure/api/v2/101885'
    url = f'{BASE}/{urllib.parse.quote(table)}/update.json'
    req = urllib.request.Request(url, data=body, headers={
        'Authorization': f'Bearer {token}',
        'Content-Type': f'multipart/related;boundary={boundary}',
    }, method='POST')

    with urllib.request.urlopen(req, timeout=60) as resp:
        return json.loads(resp.read().decode('utf-8'))
```

**Key notes:**
- Works with `create.json`, `update.json`, and `upsert.json`
- Multiple files: add extra MIME parts, each with unique `Content-ID`
- `@row.id` is required for update (NOT the user-defined `Id` Autonumber column)
- To get `@row.id`: it's always included in `select.json` responses automatically
- Ref: [ForeSoftCorp/TeamDesk-RESTAPI-PHP](https://github.com/ForeSoftCorp/TeamDesk-RESTAPI-PHP) `RestApi.php` → `doUpsert()`

> **Important:** The `attachment` GET endpoint is read-only (download). CSV import (`td import`) also does NOT support attachment columns. `multipart/related` on Create/Update/Upsert is the **only** programmatic way to upload files.

### Key API Notes

- Default limit: 500 records per request (use `skip` and `top` for pagination)
- Dates format: `YYYY-MM-DDTHH:MM:SS+00:00`
- Duration: Number of seconds
- Checkboxes: `true` or `false`
- Triggers execute by default on create/update/upsert (disable with `?triggers=false` if user has Manage Data privilege)

## Formula Quick Reference

### Syntax Basics
- Column reference: `[Column Name]`
- Variable: `Var[Variable Name]`
- User property: `User[Property Name]`
- Related column: `Related[Column Name]` (only in summary filters)

### Essential Functions

**Conditionals:**
```
If([Status]="Paid", "Complete", "Pending")
Case([Priority], "High", 1, "Medium", 2, 3)
```

**Date/Time:**
```
Today()                          // Current date
Now()                            // Current timestamp
Year([Date]) / Month([Date]) / Day([Date])
DateDiff("d", [Start], [End])    // Days between dates
[Date] + Days(30)                // Add duration
AdjustMonth([Date], 1)           // Add months
```

**Text:**
```
List(", ", [First], [Last])      // Join with separator
Left([Text], 5) / Right([Text], 5)
Contains([Text], "search")       // Returns checkbox
URLEncode([Text])
```

**Type Conversion:**
```
ToNumber("123.45")
ToText([Number])
ToDate("2024-01-15")
Days([Number]) / Hours([Number]) / Minutes([Number])
```

**Null Handling:**
```
IsNull([Field])
IfNull([Field], "default")
```

### Aggregation (Summary Columns)
```
Total([Amount])
Count([Id])
Average([Value])
Min([Date]) / Max([Date])
Concatenate(", ", [Name])
```

## Workflow Automation

### Trigger Types
1. **Record Change**: Fires on create/update/delete of specific columns
2. **Time-Dependent**: Fires relative to a date column (e.g., 3 days before Due Date)
3. **Periodic**: Fires on schedule (daily, weekly, monthly)

### Action Types
- Email Alert
- Record Create/Update/Delete
- Call URL (HTTP request to external API)
- Navigate
- Document Generation

### Webhooks (Receiving Data)
- Endpoint: `https://www.teamdesk.net/secure/db/{id}/hooks/{endpoint}`
- Supports JSON, XML, form-data
- Use Response() function to extract fields
- Iterators allow processing arrays (one record per item)

## Table Relationships

### One-to-Many
- Master table has Key column (usually Id)
- Detail table has Reference column pointing to master
- Creates Lookup columns (pull data from master) and Summary columns (aggregate from details)

### Many-to-Many
- Uses Link table with two reference columns
- Or use match conditions (e.g., Designer=Designer between Hours and Payments)
- Multi-reference column provides UI for selecting multiple related records

## n8n Integration Patterns

### HTTP Request Node Setup
```
Method: GET (select) or POST (create/update/upsert)
URL: https://www.teamdesk.net/secure/api/v2/{db_id}/{table}/select.json
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json
```

### Common Workflow Pattern
1. **Trigger**: Webhook, Schedule, or another app
2. **TeamDesk Select**: Get existing records
3. **Process**: Transform data with Code/Set nodes
4. **TeamDesk Upsert**: Create or update records using match column

### Pagination Loop
```javascript
// In Code node for handling >500 records
let allRecords = [];
let skip = 0;
const top = 500;
let hasMore = true;

while (hasMore) {
  const response = await $http.get(url + `?top=${top}&skip=${skip}`);
  allRecords = allRecords.concat(response.data);
  hasMore = response.data.length === top;
  skip += top;
}
return allRecords;
```

## Backup & Restore (CLI `td`)

Ferramenta cross-platform unificada para backup, restore, import e export de dados TeamDesk.

**Download:** https://teamdesk.crmdesk.com/answer.aspx?aid=22386
**Plataformas:** Windows 10+, macOS Catalina+, Linux (CentOS 7+, Debian 10+, Ubuntu 18.04+) — x64 e ARM64

### Sintaxe Base
```
td <command> <url> -u <user> -p <password> [options]
```

A `<url>` é a base da API: `https://www.teamdesk.net/secure/api/v2/{database_id}`

### Comandos

**Backup completo:**
```bash
td backup https://www.teamdesk.net/secure/api/v2/101885 -u user@email.com -p senha --verbose
```

**Restore completo:**
```bash
td restore https://www.teamdesk.net/secure/api/v2/101885 -u user@email.com -p senha
```

**Export (tabela específica com filtro):**
```bash
td export <url> -u ... -p ... --file dados.csv --table "Usina Solar" --column Nome Potencia Status --filter "not IsNull([Status])" --sort Nome//ASC
```

**Import (tabela específica):**
```bash
td import <url> -u ... -p ... --file dados.csv --table "Usina Solar" --column Nome --column Data=Date --column Criado=Timestamp
```

**Import excluindo colunas:**
```bash
td import <url> -u ... -p ... --file dados.csv --table "Usina Solar" --exclude "Data Criação"
```

### Parâmetros Principais

| Parâmetro | Abreviação | Descrição |
|-----------|------------|-----------|
| `--user` | `-u` | Email do usuário |
| `--password` | `-p` | Senha |
| `--file` | `-f` | Arquivo CSV (`-` para stdout) |
| `--table` | | Tabela alvo (import/export) |
| `--column` | | Colunas (repetir para múltiplas, aceita mapeamento `NomeCSV=Tipo`) |
| `--exclude` | | Excluir colunas do import |
| `--filter` | | Expressão de filtro (fórmulas TeamDesk) |
| `--sort` | | Ordenação (`Coluna//ASC` ou `Coluna//DESC`) |
| `--culture` | | Locale para formatação (`pt-BR`, `en-US`) |
| `--verbose` | | Saída detalhada |

### Response Files (reutilizar credenciais)

Criar arquivo `forgreen.txt`:
```
https://www.teamdesk.net/secure/api/v2/101885
--user daniel@forgreen.com.br
--password senha123
```

Usar com `@`:
```bash
td backup @forgreen.txt --verbose
td export @forgreen.txt --file usinas.csv --table "Usina Solar"
```

Combinar múltiplos: `td export @forgreen.txt @csv-options.txt`

### Configuração Global

Arquivo `td.config.yml` (YAML) para defaults:
```yaml
culture: pt-BR
verbose: true
```

### Notas
- Argumentos aceitam 3 formatos: `--user valor`, `--user:valor`, `--user=valor`
- Import de URLs: colunas de attachment baixam automaticamente o conteúdo da URL
- GUI (Windows-only) ainda disponível, mas será substituída por versão Web

## AI Data Analysis (2025)

Recurso nativo para consultar dados em linguagem natural, disponível no menu lateral do TeamDesk.

### Como funciona
1. Acesse **Data Analysis** no menu lateral (acima de dashboards/views)
2. Faça perguntas em linguagem natural sobre seus dados
3. Peça gráficos, refinamentos, agrupamentos
4. Resultados são salvos automaticamente com histórico de conversa
5. Botão "Copy" para exportar respostas para email/apresentações

### Acesso
- Disponível apenas para usuários com **Full Access setup permissions** (expansão planejada)
- Respeita permissões de tabelas/colunas existentes
- Dados processados via Microsoft Azure (East US 2)

### Instruções por Tabela (AI Instructions)
Personalize o comportamento da IA por tabela criando arquivos Markdown em **Database Resources**:

Acesse via botão **"Edit Instructions"** nas configurações da tabela. Conteúdo sugerido:

```markdown
# Instruções para tabela Geração Mensal

## Colunas importantes
- `Energia_Gerada_kWh`: valor principal de geração
- `Periodo`: usar para filtros temporais (formato YYYY/MM)
- `Instalacao`: identificador da usina

## Filtros comuns
- Excluir registros com `Status = "Cancelado"`
- Período padrão: últimos 12 meses

## Representação
- Sempre totalizar `Energia_Gerada_kWh` (não fazer média)
- Agrupar por `Instalacao` e `Periodo` (mês)
- Gráfico preferido: barras empilhadas
```

As instruções são aplicadas automaticamente em toda análise daquela tabela.

## Document Generation (MERGEFIELD Avançado)

### Update Fields (Avaliação em 2 passes)
Ativar em: Template de documento > marcar **"Update Fields"** (desativado por default).

**Passo 1**: Substitui MERGEFIELDs com valores do registro
**Passo 2**: Avalia campos Word restantes (~60 tipos)

**Exemplos avançados:**
```
{ IF { MERGEFIELD Amount } = 1 "One" "Many" }
→ Texto condicional baseado no valor

{ ={ MERGEFIELD Contract_Sum } \* CardText }
→ Converte número para texto por extenso ("one thousand")
```

## Workflow Patterns

### Gerar Registros Filhos em Lote (Indexes Pattern)
Para criar N registros filhos automaticamente (ex: parcelas de pagamento):

**Estrutura**: 3 tabelas
1. **Pai** (ex: Empréstimo): Amount, Number_of_Payments, Start_Date
2. **Filho** (ex: Parcela): Amount, Date, Index, referência ao Pai
3. **Indexes** (técnica): coluna `Index` com números sequenciais (1, 2, 3... até o máximo desejado)

**Setup**:
- Relação Many-to-Many entre Pai e Indexes com filtro: `[Index] <= Related[Number Of Payments]`
- Custom Button no Pai → Record Create no Filho com assignments:
  - `[Index]` = número sequencial
  - `ParentKey()` = link ao registro Pai
- Fórmulas calculadas no Filho:
  ```
  Amount: [Loan Amount] / [Loan Number Of Payments]
  Date:   AdjustMonth(FirstDayOfMonth([Loan Start Date]), [Index])
  ```

### Merge de Registros Duplicados
Para consolidar duplicatas mantendo registros filhos:

1. **Detectar**: Summary column com filtro `[Name]=Related[Name] and [Id]<>Related[Id]`
2. **Custom Button "Merge Master"**: Preenche `[Master Id]` nos duplicados
3. **Trigger em [Master Id]**: Atualiza registros filhos apontando para o master
4. **Deleta** duplicados após migração dos filhos

## Custom JavaScript (Nova UI v3)

### Detecção de versão
```javascript
if (window.V3) {
    // Código para nova UI
} else {
    // Código para UI legada
}
```

### APIs disponíveis (Nova UI)
```javascript
// Substitui jQuery(callback) — espera carregamento da página
TD.ready(function() {
    // seu código aqui
});

// Carregar scripts externos (sync ou async)
FS.addScript("https://cdn.example.com/lib.js", function() {
    // callback após carregamento
});
FS.addScript(["script1.js", "script2.js"], callback); // múltiplos
```

### Substituições jQuery → Nativo

| jQuery (legado) | Nativo (nova UI) |
|-----------------|------------------|
| `$('selector').on('click', fn)` | `document.querySelector('selector').addEventListener('click', fn)` |
| `$.qstring` | `new URLSearchParams(location.search)` |
| `$.cookies` | `localStorage` / `sessionStorage` |
| `$(document).ready(fn)` | `TD.ready(fn)` |

### Migração rápida (carregar jQuery na nova UI)
Criar arquivo `dbscript-v3.js` em Database Resources:
```javascript
document.write('<script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>');
document.write('<script src="res.aspx/dbscript.js"></script>');
```

## CSS Customization (Nova UI v3)

A nova UI expõe **50+ CSS custom properties** para personalização. Arquivo CSS vai em **Database Resources**.

### Variáveis mais úteis
```css
:root {
    --v3-font-face: "IBM Plex Sans";          /* Fonte principal */
    --v3-font-pref-normal: 14px;              /* Tamanho base */
    --v3-border-radius: 6px;                  /* Cantos arredondados */
    --v3-sidebar-width: 256px;                /* Largura do menu lateral */
    --v3-transition-time: 300ms;              /* Duração de animações */
    --v3-accent-color-h: 210;                 /* Cor de destaque (hue) */
    --v3-accent-color-s: 100%;                /* Cor de destaque (saturação) */
    --v3-accent-color-l: 40%;                 /* Cor de destaque (luminosidade) */
}
```

### Paleta de cores disponíveis
`blue`, `red`, `orange`, `yellow`, `green`, `teal`, `navy`, `violet`, `purple`, `pink`, `black`, `white`, `gray`

Acessar via: `--v3-palette-{nome}-color-h/s/l` (ex: `--v3-palette-green-color-h`)

### Hierarquia de cores
Tabela → Database → Domain (herança automática, tabela sem cor usa a do database)

> Referência completa de CSS properties: `references/css-properties.md`

## Reference Files

For detailed information, see:
- **references/api-reference.md**: Complete API method documentation, parameters, response formats, error codes
- **references/formula-reference.md**: Full function list organized by category with examples
- **references/css-properties.md**: Complete list of CSS custom properties for UI customization

## Limits

- Select: 500 records default (use pagination)
- Export to Excel: 10,000 records
- Export to CSV: 100,000 records
- Trigger processing: 100 records per licensed user + 1,000 for external users pack
- SQL Server backend: Millions of records per table supported
- AI Data Analysis: Rate limit (mensagem "Try again in XX seconds")
