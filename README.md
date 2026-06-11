# 🤖 Documentação Técnica: Bot Tabela de Valores

**Autor:** Kalel Souza Barros  
**Última Atualização:** Junho de 2026  
**Status do Projeto:** `[Ativo / Em Produção / Em Testes]`  
**Repositório Git:** `[Inserir o link do repositório aqui]`

---

## 1. Resumo do Projeto

### O que ele é:
O **Bot Tabela de Valores** é uma automação em background desenvolvida para interagir com a plataforma de CRM de forma autônoma, simulando as ações de um operador humano na gestão de tabelas de preços de empreendimentos imobiliários.

### O que ele resolve:
Ele elimina um processo mensal exaustivo e altamente repetitivo que era realizado manualmente pela equipe de vendas. A automação localiza tabelas de preços administrativas estáticas, clona toda a estrutura de valores de venda e gera uma nova versão parametrizada como **"Dinâmica (Valor Unidades)"**. 

**Impacto de Negócio:** Essa conversão de formato é mandatória e resolve uma limitação técnica crítica: a API que extrai os dados para alimentar o **Dashboard Comercial (Business Intelligence)** não possui suporte para leitura de tabelas estáticas. Sem a execução deste bot, o dashboard de tomada de decisões da diretoria fica completamente desatualizado.

---

## 2. Arquitetura do Projeto

### Tecnologias Utilizadas e Versões
* **Linguagem Base:** Java `[Inserir versão ex: JDK 17]`
* **Gerenciador de Dependências / Build:** `[Inserir Maven ou Gradle e sua respectiva versão]`
* **Framework de Automação Web:** `[Inserir Selenium WebDriver ou similar e a versão]`
* **Navegador Alvo:** Google Chrome e ChromeDriver `[Inserir a versão estável homologada]`

### Organização das Pastas (Estrutura do Diretório)
Abaixo está a representação da estrutura de arquivos necessária no ambiente de execução local:

```text
BotTabelaValores/
├── .env                              # Arquivo com variáveis de ambiente e credenciais secretas
├── bot-tabelas_de_preco-1.0-all.jar   # Artefato executável principal compilado em Java
├── Abrir Chrome.bat                  # Script para inicializar o navegador em modo de depuração
├── Executar_Bot.bat                  # Script para disparar a execução do fluxo Java
├── cache do chrome/                  # Diretório local para persistência de cookies e sessões
└── logs/                             # Pasta destinada aos arquivos de histórico e rastreamento (.log)

```

---

## 3. Execução do Projeto

### Pré-requisitos de Instalação

Para configurar e rodar o projeto localmente ou em uma nova máquina de automação, certifique-se de possuir:

1. **AnyDesk** instalado (caso o acesso à máquina `1085005551` precise ser feito remotamente).
2. **Java Runtime Environment (JRE)** instalado e configurado nas variáveis de ambiente globais do sistema operacional.
3. **Google Chrome** instalado na mesma versão do arquivo `ChromeDriver` contido na aplicação.

### Configuração do Arquivo `.env`

Crie um arquivo texto nomeado exatamente como `.env` na raiz do projeto. Preencha o arquivo seguindo o template de chaves abaixo:

```env
# URL de Acesso aos Sistemas Externos
CV_CRM_URL=[https://suaempresa.cvcrm.com.br](https://suaempresa.cvcrm.com.br)
API_EMPREENDIMENTOS_URL=[https://api.exemplo.com/v1/empreendimentos](https://api.exemplo.com/v1/empreendimentos)

# Credenciais de Autenticação do Robô
BOT_USER=usuario_do_bot
BOT_PASSWORD=senha_segura_do_bot

```

> ⛔ **Nota de Segurança:** Nunca envie o arquivo `.env` com senhas reais para o repositório Git. Mantenha apenas este template documentado.

### Instruções para Execução Local (Passo a Passo)

* **Passo 1:** Abra o terminal na pasta do projeto e certifique-se de que o arquivo `.env` está configurado corretamente.
* **Passo 2:** Execute o arquivo `Abrir Chrome.bat`. Uma janela isolada do navegador Chrome será aberta.
* **Passo 3:** Na janela do navegador que se abriu, realize o login manual ou valide se a sessão do sistema **CV CRM** está ativa utilizando as credenciais contidas no `.env`.
* **Passo 4:** Retorne à pasta do projeto e execute o arquivo `Executar_Bot.bat`.

> 🚨 **Importante:** Não feche a janela preta do terminal que foi aberta e evite utilizar as mesmas credenciais em outro dispositivo simultaneamente enquanto o bot estiver processando, sob o risco de derrubar a sessão ativa e quebrar o fluxo de automação.

---

## 4. Funcionalidades

O projeto é constituído por um fluxo lógico de ciclo fechado, composto pelas seguintes subfunções detalhadas:

### A. Consumo de API de Empreendimentos

O robô realiza uma requisição HTTP do tipo `GET` para o endpoint configurado na variável `API_EMPREENDIMENTOS_URL`. A resposta retorna um payload contendo todos os locais cadastrados na construtora. O sistema aplica um filtro em memória para isolar apenas os empreendimentos cuja situação comercial seja diferente de `'unidades vendidas'`.

### B. Mapeamento em Memória

Os registros filtrados na etapa anterior têm seus identificadores únicos (IDs) mapeados e organizados dentro de uma estrutura de dados indexada do tipo Chave-Valor (Dicionário/Map), preparando a fila sequencial de execução.

### C. Varredura e Busca no CRM

O bot inicia uma iteração navegando pela interface do CV CRM. Para cada ID presente na fila de mapeamento, ele acessa o painel de tabelas de preços correspondente e realiza uma busca de texto tentando localizar tabelas nomeadas estritamente como `"uso administrativo"` associadas ao mês corrente.

### D. Clonagem e Configuração Dinâmica

Ao localizar a tabela correta, o robô executa as seguintes subetapas:

1. Aciona o comando interno para copiar integralmente a matriz de valores de venda das unidades.
2. Cria uma nova tabela de preços no sistema aplicando o padrão de nomenclatura: `[Nome do Empreendimento] + [Mês/Ano Vigente] + dashboard`.
3. Altera a propriedade técnica do tipo de geração de tabela para **"Dinâmica (Valor Unidades)"**.
4. Realiza a colagem em lote de todos os valores salvos em memória.

### E. Finalização do Ciclo e Inativação

Com a nova tabela gerada com sucesso, o bot executa a alteração de status da tabela `"uso administrativo"` antiga para **Inativa** (garantindo que o sistema use apenas a nova versão de dashboard). O fluxo limpa a memória temporária e avança de forma sequencial para o próximo ID mapeado na lista até concluir a fila.

```

### 💡 Dica para a sua gestão:
Usando esse bot como espelho, o Kalel conseguirá facilmente adaptar os outros três bots (**BotDCSLauncher**, **BotDisparoReserva** e **BotValidaContratos**) exatamente para essa mesma estrutura, pois agora ele tem um guia claro de profundidade técnica e organização textual.

```
