# SGA Fiocruz/COC — Netlify + Google Sheets

Arquitetura: front-end estático (Netlify) → Google Apps Script → Google Sheets como banco de dados.

## O que foi corrigido nesta versão

1. **Tabela de temporalidade vazia** — o `script.js` que você tinha só continha
   um item de exemplo (`/* ... insira os demais itens aqui ... */`). Este
   `script.js` já vem com os 607 códigos completos embutidos.
2. **CORS quebrando o POST** — os `fetch(...)` de salvar/excluir enviavam
   `headers: {'Content-Type':'application/json'}`. Isso faz o navegador
   disparar um *preflight* `OPTIONS`, que o Apps Script não responde, e a
   chamada falha silenciosamente. Removido o header — o corpo continua sendo
   JSON, só que o Apps Script lê `e.postData.contents` direto, então funciona
   igual.
3. **Não existia backend** — criei o `Code.gs` completo (login, listar,
   salvar, excluir, importar em lote).
4. **Tela de login** — refeita (ver seção "Sobre o design" abaixo).

## Passo a passo

### 1. Google Sheets
Crie uma planilha nova com duas abas, **com esses nomes exatos**:

**Usuarios**
| usuario | senha | nome | papel |
|---|---|---|---|
| maria.arquivista | 12345 | Maria Silva | arquivista |

**Documentos** — apenas o cabeçalho, nesta ordem:
```
id | caixa | codigo | classificacao | referencia | processo | dataRegistro | dataLimite | prazoCorrente | prazoIntermediario | destinacao | localizacao | estado | termo
```

### 2. Apps Script
Na planilha: **Extensões → Apps Script** → apague o conteúdo → cole o
`Code.gs` deste pacote → **Implantar → Nova implantação → App da Web**
- Executar como: **Eu**
- Quem pode acessar: **Qualquer pessoa**

Copie a URL gerada (termina em `/exec`).

### 3. config.js
Cole a URL no lugar de `COLE_AQUI_A_URL_DO_APPS_SCRIPT`.

### 4. Netlify
Arraste a pasta inteira (`index.html`, `login.html`, `config.js`, `script.js`,
`style.css`) para app.netlify.com/drop, ou conecte um repositório Git. Não
precisa de build — é só HTML/CSS/JS estático.

## Sobre a segurança do login

Vale ser transparente: senha comparada em texto puro numa planilha não é uma
autenticação de verdade — qualquer pessoa com acesso de edição à planilha vê
as senhas. Para uso interno de baixo risco (equipe pequena, sem dados
sensíveis de terceiros) costuma ser aceitável como primeira versão. Se o
sistema crescer ou passar a guardar dado sensível, o próximo passo é migrar
o login para algo com hash de senha de verdade (mesmo dentro do Apps Script,
com `Utilities.computeDigest`) ou trocar por um provedor de autenticação
(Google/Firebase Auth), mantendo o Sheets só como banco de documentos.

## Sobre o design da tela de login

A anterior era um cartão genérico centralizado — funcional, mas sem
identidade. A nova mantém a mesma paleta "arquivo/ledger" do resto do
sistema (tinta escura, latão, serifada para títulos) e adiciona um elemento
de assinatura: ao autenticar com sucesso, um **selo de carimbo** ("ACESSO
CONCEDIDO") bate na tela antes de entrar — uma referência direta ao gesto
físico de carimbar documentos no arquivo, em vez de mais uma barra de
progresso genérica. Fundo com linhas horizontais sutis lembrando papel
pautado de livro-razão.
