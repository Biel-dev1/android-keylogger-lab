# Laboratório para estudo de um Keylogger para Android

**⚠️ Aviso de Ética e Uso Responsável ⚠️**

Este projeto foi desenvolvido com fins **estritamente educacionais** para o estudo de vulnerabilidades em sistemas Android. O objetivo é entender como os Serviços de Acessibilidade podem ser explorados e quais contramedidas os sistemas operacionais implementam. O uso deste software em dispositivos sem o consentimento explícito do proprietário é ilegal e antiético. O autor não se responsabiliza por qualquer uso indevido.

---

## 1. Visão Geral do Projeto

Este repositório contém o código-fonte de um laboratório de cibersegurança composto por duas partes principais:

1.  **Aplicativo Android (Agente):** Um aplicativo desenvolvido em C# com .NET MAUI que, ao ser ativado, utiliza os Serviços de Acessibilidade do Android para capturar eventos de texto (simulando um *keylogger*).
2.  **Servidor de Comando e Controle (C2):** Um script em Python que atua como um servidor "ouvinte" (*listener*), recebendo e registrando os dados enviados pelo aplicativo Android em tempo real.

O objetivo deste laboratório é demonstrar na prática como uma funcionalidade legítima do sistema pode ser abusada para fins de monitoramento e, mais importante, analisar as defesas que o Android emprega para mitigar esse risco, especialmente em telas seguras como a de desbloqueio.

## 2. Arquitetura do Laboratório

O fluxo de dados ocorre da seguinte maneira:

1.  O **Aplicativo Android** é instalado e seu Serviço de Acessibilidade é ativado manualmente pelo usuário.
2.  Ao ser ativado, o app envia um sinal de "alerta" para o servidor, confirmando que está ativo.
3.  O app passa a monitorar eventos de acessibilidade no dispositivo. Quando um texto é digitado, ele é capturado.
4.  Cada texto capturado é enviado via TCP para o **Servidor Python** rodando na máquina Kali Linux.
5.  O servidor recebe os dados, adiciona um *timestamp* (data e hora), exibe no terminal e salva em um arquivo de log (`keylog.txt`) para análise posterior.

## 3. Como Configurar e Executar o Laboratório

### Pré-requisitos

*   **Para o App Android:**
    *   Visual Studio 2026 com a carga de trabalho .NET MAUI.
    *   Um dispositivo ou emulador Android para testes.
*   **Para o Servidor:**
    *   Uma máquina com Python 3 (ex: Kali Linux).

### Passo a Passo

#### Parte 1: Configurar o Servidor (Kali Linux)

1.  Clone este repositório ou copie o arquivo `recebimento.py` para sua máquina.
2.  Abra um terminal e execute o servidor:
    ```bash
    python3 recebimento.py
    ```
3.  O servidor começará a escutar na porta 9001. Anote o endereço IP da sua máquina Kali.

#### Parte 2: Configurar e Executar o Aplicativo (Android)

1.  Abra a solução do projeto no Visual Studio.
2.  Navegue até o arquivo `Platforms/Android/MyAccessibilityService.cs`.
3.  Localize a constante `KALI_IP` e **altere o valor** para o endereço IP da sua máquina Kali:
    ```csharp
    private const string KALI_IP = "SEU_IP_DO_KALI_AQUI";
    ```
4.  Compile e execute o aplicativo no seu dispositivo de teste.
5.  No dispositivo Android, navegue até **Configurações > Acessibilidade > Aplicativos Instalados**.
6.  Encontre o seu aplicativo na lista e **ative o serviço**. Você precisará aceitar os avisos de segurança do Android.
7.  Imediatamente após a ativação, você deverá ver uma mensagem de `[ALERTA]` no terminal do seu servidor Python.

#### Parte 3: Testar a Captura

1.  Com o serviço ativo, digite em qualquer campo de texto no dispositivo Android (navegador, app de notas, etc.).
2.  Observe o terminal do servidor. As teclas digitadas devem aparecer em tempo real, com data e hora.
3.  Um arquivo `keylog.txt` será criado na mesma pasta do script `recebimento.py`, armazenando um log de tudo o que foi capturado.

## 4. Descobertas e Análise

Durante os testes, observamos um comportamento de segurança interessante:

*   **Em campos de texto normais:** O keylogger captura o texto digitado.
*   **Na tela de bloqueio:** O sistema operacional "ofusca" a entrada. Em vez de enviar os caracteres reais da senha, o teclado envia dados de preenchimento (ex: "ABC" em vez de "2"), tornando a captura direta da senha mais "difícil".

Isso demonstra uma contramedida de "segurança" deliberada por parte do Android para proteger dados sensíveis, mesmo de serviços com permissões elevadas.
