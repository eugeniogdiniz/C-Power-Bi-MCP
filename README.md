# Configuração do Power BI Modeling MCP Server no Claude Desktop

## Visão Geral

Este documento descreve o passo a passo realizado para conectar o **Power BI Desktop** ao **Claude Desktop** via **Model Context Protocol (MCP)**, permitindo interação em linguagem natural com modelos semânticos do Power BI.

---

## O que é o MCP?

O **Model Context Protocol (MCP)** é um protocolo aberto que padroniza a comunicação entre assistentes de IA e ferramentas externas. No contexto do Power BI, ele permite que o Claude Desktop leia e modifique modelos semânticos diretamente, sem precisar copiar e colar código manualmente.

---

## Pré-requisitos

- Windows 10 ou superior
- Power BI Desktop instalado
- Claude Desktop instalado
- WinRAR ou similar para extrair arquivos `.vsix`

---

## Passo 1 — Instalar o Claude Desktop

O Claude Desktop é o **app nativo** do Claude para Windows. É diferente do acesso pelo navegador (claude.ai), que não suporta MCP.

1. Acesse: [https://claude.ai/download](https://claude.ai/download)
2. Baixe o instalador para Windows
3. Execute o instalador e siga os passos
4. **Abra o Claude Desktop ao menos uma vez** — isso cria automaticamente a pasta de configuração:

```
C:\Users\<seu_usuario>\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\
```

> ⚠️ **Atenção:** O atalho do Claude no navegador (PWA) **não é** o Claude Desktop. Certifique-se de instalar o app nativo.

---

## Passo 2 — Baixar a Extensão do Power BI Modeling MCP

A Microsoft disponibiliza o servidor MCP como uma extensão do VS Code (`.vsix`). O executável do servidor está empacotado dentro desse arquivo.

1. Acesse o VS Code Marketplace:  
   [https://marketplace.visualstudio.com/items?itemName=analysis-services.powerbi-modeling-mcp](https://marketplace.visualstudio.com/items?itemName=analysis-services.powerbi-modeling-mcp)
2. Baixe o arquivo `.vsix` da extensão **Power BI Modeling MCP Server**

---

## Passo 3 — Extrair o Executável do Servidor MCP

O arquivo `.vsix` é um ZIP renomeado. O executável do servidor está dentro dele.

1. Renomeie o arquivo `.vsix` para `.zip`
2. Extraia o conteúdo com WinRAR ou similar
3. Navegue até a pasta:
   ```
   extension\server\
   ```
4. Localize o arquivo **`powerbi-modeling-mcp.exe`**

### Arquivos presentes na pasta `server\`:

| Arquivo | Descrição |
|---|---|
| `powerbi-modeling-mcp.exe` | Executável principal do servidor MCP |
| `msalruntime.dll` | Biblioteca de autenticação Microsoft |
| `msasxpress.dll` | Biblioteca Analysis Services |
| `AnalysisServices.AppSettings.json` | Configurações do servidor |
| `Resources\` | Recursos adicionais |

5. Salve essa pasta em um local permanente no seu computador, por exemplo:
   ```
   F:\PROJECTS\C-Power-Bi-MCP\extension\server\
   ```

---

## Passo 4 — Verificar os Argumentos do Executável

Antes de configurar, verificamos os argumentos aceitos pelo executável rodando no terminal:

```bash
"F:\PROJECTS\C-Power-Bi-MCP\extension\server\powerbi-modeling-mcp.exe" --help
```

**Saída retornada:**

```
Semantic Model MCP Server
Usage:
  --start                      Start the MCP server (uses default ReadWrite mode)
  --read-only                  Start in read-only mode
  --readonly                   Start in read-only mode (alias)
  --read-write                 Start in read-write mode
  --readwrite                  Start in read-write mode (alias)
  --skip-confirmation          Skip confirmation prompts for write operations
  --compatibility=powerbi      PowerBI only compatibility (default)
  --compatibility=full         Full compatibility (PowerBI + Analysis Services)
  --version                    Show version information
  --help, -h                   Show this help message
```

O argumento necessário para iniciar o servidor é **`--start`**.

---

## Passo 5 — Localizar o Arquivo de Configuração do Claude Desktop

O arquivo de configuração do Claude Desktop fica em:

```
C:\Users\<seu_usuario>\AppData\Local\Packages\Claude_pzs8sxrjxfjjc\LocalCache\Roaming\Claude\claude_desktop_config.json
```

> 💡 **Dica:** Cole o caminho abaixo diretamente na barra de endereço do Explorador de Arquivos:
> ```
> %APPDATA%\Claude
> ```
> Se não funcionar, navegue manualmente habilitando a visualização de arquivos ocultos em:  
> **Exibir → Itens ocultos ✅**

---

## Passo 6 — Configurar o arquivo `claude_desktop_config.json`

1. Abra o arquivo `claude_desktop_config.json` com o **Bloco de Notas**
2. Adicione o bloco `mcpServers` apontando para o executável:

```json
{
  "mcpServers": {
    "powerbi-modeling": {
      "command": "F:\\PROJECTS\\C-Power-Bi-MCP\\extension\\server\\powerbi-modeling-mcp.exe",
      "args": ["--start"]
    }
  },
  "preferences": {
    "coworkWebSearchEnabled": true,
    "coworkScheduledTasksEnabled": true,
    "ccdScheduledTasksEnabled": true,
    "sidebarMode": "chat"
  }
}
```

> ⚠️ **Importante:** Use **barras duplas** `\\` no caminho, pois é exigido pelo formato JSON no Windows.

3. Salve o arquivo com **Ctrl + S**

---

## Passo 7 — Reiniciar o Claude Desktop

1. Feche o Claude Desktop completamente
2. Verifique a **bandeja do sistema** (canto inferior direito) e encerre o processo se ainda estiver rodando
3. Abra o Claude Desktop novamente

---

## Passo 8 — Testar a Conexão

1. Abra um arquivo `.pbix` no **Power BI Desktop**
2. No Claude Desktop, digite:

```
Connect to Power BI Desktop
```

ou diretamente:

```
What tables are in my model?
```

Se tudo estiver correto, o Claude vai listar as tabelas do seu modelo semântico.

---

## Problemas Encontrados e Soluções

### ❌ Erro: `Unexpected token '_' is not valid JSON`

**Causa:** O executável imprimia um banner ASCII no stdout antes de iniciar, quebrando o protocolo MCP.

**Solução:** Adicionar o argumento `--start` no campo `args` do JSON de configuração:

```json
"args": ["--start"]
```

---

### ❌ Pasta `C:\Users\...\AppData\Roaming\Claude` não encontrada

**Causa:** O Claude Desktop não estava instalado — apenas o atalho PWA do navegador estava presente.

**Solução:** Instalar o Claude Desktop nativo via [https://claude.ai/download](https://claude.ai/download) e abri-lo ao menos uma vez.

---

## Comandos Úteis após a Conexão

| Comando | O que faz |
|---|---|
| `Connect to Power BI Desktop` | Conecta ao modelo aberto no PBI Desktop |
| `What tables are in my model?` | Lista todas as tabelas |
| `What measures exist?` | Lista todas as medidas DAX |
| `Run DAX: EVALUATE TOPN(5, Sales)` | Executa uma query DAX |
| `Document this semantic model in Markdown` | Gera documentação completa do modelo |
| `Add a measure called [nome] with DAX [código]` | Cria uma nova medida |

---

## Arquitetura da Integração

```
Claude Desktop (app nativo)
        │
        │  MCP Protocol (stdio)
        ▼
powerbi-modeling-mcp.exe  (servidor local)
        │
        │  Analysis Services / XMLA
        ▼
Power BI Desktop (modelo semântico aberto)
```

---

## Referências

- [Power BI Modeling MCP Server — GitHub](https://github.com/microsoft/powerbi-modeling-mcp)
- [Power BI MCP Servers Overview — Microsoft Learn](https://learn.microsoft.com/en-us/power-bi/developer/mcp/mcp-servers-overview)
- [VS Code Marketplace — Power BI Modeling MCP](https://marketplace.visualstudio.com/items?itemName=analysis-services.powerbi-modeling-mcp)
- [Claude Desktop Download](https://claude.ai/download)

---

*Documentação gerada em 16/03/2026*