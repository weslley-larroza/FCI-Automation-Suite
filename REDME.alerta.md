# AlertaFCI - Sistema de Monitoramento e Notificações por E-mail

## 📋 Visão Geral do Projeto
O **AlertaFCI** é um serviço de monitoramento autônomo em background desenvolvido em Python para supervisionar o fluxo de processos de Ficha de Conteúdo de Importação (FCI). O script realiza varreduras periódicas em tempo real nas tabelas do Supabase, identifica gargalos operacionais — como pedidos marcados como incorretos ou lotes aguardando intervenção — e despacha alertas consolidados por e-mail de forma segura via protocolo SMTP/SSL, atualizando o banco para evitar duplicidade de notificações.

---

## 🎯 Desafio de Negócio e Resultados Positivos
* **Proatividade Operacional:** Elimina a necessidade de checar manualmente o painel ou o banco de dados para identificar falhas ou travamentos no fluxo de FCI.
* **Alertas Consolidados e Inteligentes:** Agrupa todas as pendências de múltiplas tabelas em um único resumo consolidado, evitando a poluição da caixa de entrada dos gestores e da equipe técnica.
* **Segurança e Privacidade no Envio:** Dispara o e-mail de forma individual para cada destinatário mapeado (ocultando a lista de réplicas e preservando a privacidade), utilizando autenticação segura SSL.
* **Prevenção de Spam / Duplicidade:** O sistema controla o estado de notificação no banco de dados (`email_notif_sent`), garantindo que cada alerta seja enviado exatamente uma vez até que o status do processo seja alterado.

---

## ⚙️ Arquitetura e Tecnologias Utilizadas
* **Linguagem Principal:** Python 3.x
* **Gerenciamento de Banco de Dados & API REST:** Supabase (PostgreSQL) consumido via requisições HTTP (`requests`) com tratamento de parâmetros de filtro.
* **Protocolo de Comunicação de E-mail:** Bibliotecas nativas `smtplib` e `email.message` (suporte a SSL na porta 465).
* **Segurança de Credenciais:** Carregamento de variáveis sensíveis protegidas por arquivo `.env` via `python-dotenv`.
* **Logging e Persistência:** Sistema de logs dedicado (`monitor.log`) para rastrear ciclos de varredura, status de envio e exceções de rede.

---

## 🚀 Principais Funcionalidades

1. **Varredura Cíclica Multitabela:**
   * Executa um loop contínuo de monitoramento com intervalos configurados (ex: a cada 5 minutos).
   * Consulta concorrentemente diferentes tabelas e estados críticos:
     * `fci_pedidos_validacao` com status `"Pedido Incorreto"` e sinalizador de notificação pendente.
     * `fci_soollar_sp` com status `"Aguardando"` e sinalizador pendente.

2. **Consolidação Inteligente de Alertas:**
   * Agrupa todas as pendências encontradas em um único corpo de mensagem quantitativo (ex: contagem de pedidos incorretos e packs aguardando).
   * Se houver ocorrências, monta o alerta consolidado; caso contrário, registra o ciclo limpo sem disparar ruídos desnecessários.

3. **Disparo Seguro via SMTP SSL:**
   * Autentica no servidor de e-mail configurado e envia a mensagem de forma individualizada para cada destinatário listado na variável de ambiente.

4. **Sincronização Atômica com o Banco de Dados:**
   * Após o sucesso do envio dos e-mails, executa chamadas PATCH em lote no Supabase atualizando a coluna de controle (`email_notif_sent = true`) para os registros processados.
   * Conta com tratamento de exceções robusto (`try/except`) e logging de erros para evitar interrupções no loop principal em caso de instabilidades de rede.
