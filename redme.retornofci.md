# RetornoFCI - Automação de Consulta e Importação de Retornos FCI

## 📋 Visão Geral do Projeto
O **RetornoFCI** é um robô de automação inteligente desenvolvido em Python para fechar o ciclo fiscal de Fichas de Conteúdo de Importação (FCI). O sistema rastreia protocolos transmitidos pendentes no banco de dados, acessa de forma autônoma o portal da Secretaria da Fazenda (SEFAZ-SP) via **Playwright**, realiza a navegação e o download do pacote de retorno (ZIP), extrai e normaliza os arquivos TXT oficiais, e executa o upload automático no painel de gestão interno da empresa (`/admin/fci/create`), atualizando o status do ciclo fiscal de ponta a ponta na nuvem.

---

## 🎯 Desafio de Negócio e Resultados Positivos
* **Fechamento Automático do Ciclo Fiscal:** Elimina a necessidade de consultas manuais cotidianas no portal da SEFAZ para verificar a liberação de códigos e recibos de FCI.
* **Integração End-to-End com Sistema Interno:** Automatiza o download, a extração, o tratamento dos arquivos e o preenchimento/upload no sistema web interno de administração, garantindo que os dados fiquem imediatamente disponíveis para a operação.
* **Resiliência e Tratamento de Fila via Banco de Dados:** Consome filas do Supabase de forma transacional (utilizando bloqueio de estados como "Em Processamento"), tratando exceções críticas de forma isolada e marcando registros inválidos automaticamente.
* **Auditoria e Logs Centralizados:** Registro detalhado de cada ciclo de execução em arquivos de log diários (`logs_return/`), com captura automática de telas de erro em caso de falhas de rede ou interface.

---

## ⚙️ Arquitetura e Tecnologias Utilizadas
* **Linguagem Principal:** Python 3.x
* **Automação Web Moderna (Headless/Headful):** `Playwright` para navegação síncrona robusta, manipulação de elementos DOM e captura de downloads automatizados.
* **Auxiliares de Automação de Tela:** `PyAutoGUI` para confirmações e interações com janelas de certificados digitais e SSO quando necessário.
* **Banco de Dados & API REST:** Supabase (PostgreSQL) com requisições HTTP diretas via `requests` para filtragem de filas por tempo (`timedelta`) e atualização de status em tempo real.
* **Manipulação de Arquivos e ZIP:** `zipfile`, `os`, `shutil` para descompactação e renomeação padronizada baseada no número do protocolo.
* **Logging Avançado:** Biblioteca padrão `logging` configurada com dupla saída (arquivo e terminal).

---

## 🚀 Principais Funcionalidades

1. **Gestão Inteligente da Fila de Protocolos:**
   * Consulta registros na tabela do Supabase com status pendentes ou inválidos, respeitando janelas de tempo de espera (ex: protocolos com mais de 20-22 minutos após o envio).
   * Altera o status dinamicamente para `"Em Processamento"` para reservar a linha e evitar concorrência de execução.

2. **Automação de Navegação e Certificação na SEFAZ:**
   * Acessa a página de Consulta de Código FCI da SEFAZ-SP.
   * Valida se a sessão já está autenticada ou aciona o fluxo de login por Certificado Digital (SSO) com suporte a interações do PyAutoGUI.
   * Navega de forma dinâmica pelos menus suspensos (`Consulta` -> `Códigos de FCI`), preenche o campo de protocolo e dispara a busca.

3. **Download, Extração e Normalização de Pacotes:**
   * Intercepta o evento de download do arquivo `.zip` fornecido pela SEFAZ.
   * Salva o arquivo em diretório de histórico estruturado (`historico_retornos/`), descompacta o conteúdo e localiza o arquivo `.txt` oficial de retorno.
   * Renomeia o arquivo TXT utilizando o número do protocolo correspondente para padronização.

4. **Upload Automatizado no Sistema Interno:**
   * Acessa a rota administrativa do sistema (`/admin/fci/create`), gerencia sessões expiradas realizando login automático com credenciais seguras.
   * Injeta o arquivo TXT processado diretamente no formulário de importação e submete o cadastro.

5. **Atualização de Status e Resiliência contra Falhas:**
   * Em caso de sucesso absoluto, atualiza o status de retorno no Supabase para `"Enviado"`.
   * Em caso de exceções críticas ou falhas de sistema, captura evidências visuais (prints de tela) e marca o registro como `"Invalida"` no banco de dados, garantindo que a fila nunca trave de forma silenciosa.
