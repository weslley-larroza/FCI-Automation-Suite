# FCI-Automation-Suite

> **Nota de contexto:** este repositório é uma versão de demonstração, reescrita do zero, baseada em um projeto real de automação fiscal que desenvolvi em ambiente corporativo. Nomes de tabelas, rotas, endpoints e regras de negócio específicas foram genericizados por motivo de confidencialidade. A arquitetura, as decisões técnicas e a lógica de automação refletem fielmente o projeto original.

## 📋 Visão Geral do Projeto

O **FCI-Automation-Suite** é uma solução de automação de processos robóticos (RPA) integrada a serviços em nuvem, desenvolvida para gerenciar, validar, transmitir e auditar o preenchimento de fichas fiscais obrigatórias junto a um sistema governamental oficial. O sistema automatiza integralmente a comunicação com o software validador e o sistema de transmissão do órgão regulador, eliminando tarefas manuais repetitivas, reduzindo erros operacionais e garantindo compliance fiscal com alta confiabilidade.

Este é o primeiro módulo da suíte (**Geração & Transmissão**). Os módulos de consulta de retorno e de monitoramento por e-mail seguem a mesma arquitetura e estão documentados separadamente.

---

## 🎯 Desafio de Negócio e Resultados

* **Eliminação de gargalos operacionais:** o processo de exportação e transmissão das fichas dependia de dezenas de cliques manuais e conferências visuais demoradas, gerando gargalos no fluxo de expedição.
* **Redução de erros a quase zero:** validações em tempo de execução via banco de dados e visão computacional evitam o envio de arquivos corrompidos ou com inconsistências de cadastro.
* **Rastreabilidade e auditoria completa:** cada lote processado gera logs estruturados, salva evidências visuais em caso de falha de interface e atualiza de forma síncrona o status no backend em nuvem.
* **Ganho de produtividade:** o tempo de processamento manual (horas) foi convertido em execução autônoma em segundo plano, com tratamento inteligente de exceções.

---

## ⚙️ Arquitetura e Tecnologias

| Camada | Tecnologia |
|---|---|
| Linguagem principal | Python 3.x |
| Automação de interface (RPA / Visão Computacional) | `PyAutoGUI` + `OpenCV` (`confidence` match para elementos gráficos dinâmicos) |
| Banco de dados & Backend-as-a-Service | Supabase (PostgreSQL), com Row Level Security e consumo via API REST |
| Manipulação de sistema/processos | `subprocess`, `os`, `shutil`, `ctypes` (elevação automática de privilégios) |
| Segurança e configuração | Variáveis de ambiente via `python-dotenv` |
| Logging e auditoria | Logging multicanal com retenção de `traceback` e persistência em banco |

### Por que essa combinação?

O sistema-alvo (software oficial do órgão regulador) não expõe API pública — a única via de integração é a interface gráfica. Por isso a automação combina **duas camadas**: consultas diretas ao banco (via API REST do Supabase, para decisão de negócio e controle de estado) e **RPA com visão computacional** (para a interação inevitável com a interface gráfica do sistema externo). Essa separação é o que torna o processo auditável — cada decisão de negócio fica registrada no banco, não "escondida" dentro de cliques automatizados.

---

## 🚀 Principais Funcionalidades

### 1. Triagem inteligente de pedidos
Consulta pedidos pendentes de validação diretamente na base de dados. Separa automaticamente pedidos aprovados de pedidos com inconsistências, atualizando status individuais para gerenciar filas de correção.

### 2. Geração automatizada de layout fiscal (TXT)
Monta o arquivo estruturado conforme especificação normativa do órgão regulador (blocos de cabeçalho, blocos de registro e rodapé de integridade).

### 3. Automação de tela e validação visual
Abre o executável do validador oficial e o sistema de transmissão. Usa reconhecimento de imagem (`locateOnScreen`) para localizar dinamicamente botões de ação, tolerando pequenas variações de layout. Captura prints automáticos em caso de timeout, rejeição ou estado indefinido de interface.

### 4. Tratamento de retornos e protocolos
Extrai o comprovante de transmissão via expressões regulares (`RegEx`) para capturar o número de protocolo. Atualiza o banco de forma síncrona com os recibos de envio.

### 5. Resiliência e segurança operacional
- **Failsafe do mouse:** movimento rápido do cursor para o canto superior esquerdo interrompe a automação imediatamente.
- **Elevação automática:** verifica privilégios de administrador (`ctypes.windll.shell32.IsUserAnAdmin`) e solicita elevação quando necessário.

---

## 🗂️ Estrutura (exemplo)

```
fci-automation-suite/
├── main.py                  # orquestração do fluxo
├── db/
│   └── supabase_client.py   # consultas e updates via API REST
├── automation/
│   ├── screen_actions.py    # PyAutoGUI + OpenCV
│   └── templates/           # imagens de referência para locateOnScreen
├── layout/
│   └── builder.py           # geração do arquivo TXT no layout oficial
├── logs/
└── .env.example
```

---

## 🔒 Sobre este repositório

Este código é uma **reescrita de demonstração** — não é um `git clone` do projeto original nem contém dados, credenciais ou nomes reais da empresa onde foi implantado em produção. O objetivo é documentar a arquitetura e as decisões técnicas para fins de portfólio.

