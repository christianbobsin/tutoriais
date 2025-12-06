
# 🦙 Guia rápido do Ollama no Ubuntu

Versão 1.0.0 - 05/12/2025


Pequeno tutorial para usar o **Ollama no Ubuntu** (22.04+).  
Tudo aqui é via **terminal**.

---

## ✅ Como verificar se o Ollama está instalado

No terminal:

```bash
ollama --version
```

* Se aparecer algo como `ollama version 0.x.x`, ele está instalado.
* Se aparecer `command not found`, você ainda precisa instalar.

Opcional:

```bash
which ollama      # mostra o caminho do binário, se existir
ollama list       # lista modelos locais, se o servidor estiver rodando
```

---

## 📥 Como instalar o Ollama no Ubuntu

1. Atualize e garanta que o `curl` esteja instalado:

```bash
sudo apt update
sudo apt install -y curl
```

2. Execute o script oficial de instalação:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

3. Confirme a instalação:

```bash
ollama --version
```

---

## ▶️ Como ativar o Ollama

Você pode rodar o Ollama de duas formas: **direto no terminal** ou como **serviço em background**.

### 3.1. Modo simples (somente enquanto o terminal estiver aberto)

```bash
ollama serve
```

* O comando “segura” o terminal.
* Para parar, use `Ctrl + C`.

### 3.2. Modo serviço (systemd, recomendado)

Se o instalador configurou o serviço `ollama`:

```bash
sudo systemctl start ollama      # inicia o serviço
sudo systemctl status ollama     # verifica se está rodando
```

Opcional para iniciar junto com o sistema:

```bash
sudo systemctl enable ollama
```

---

## 📚 Como listar modelos (locais)

Modelos **já baixados** (no seu PC):

```bash
ollama list
# ou
ollama ls
```

Exemplo de saída:

```text
NAME           ID        SIZE   MODIFIED
llama3.2:3b    ...       2.5 GB 2 hours ago
mistral:7b     ...       4.1 GB 1 day ago
```

---

## ⏬ Como instalar modelos (baixar para o PC)

Use `ollama pull`:

```bash
# Exemplo: baixar o Llama 3.2
ollama pull llama3.2
```

Ou simplesmente:

```bash
ollama run llama3.2
```

Se o modelo não existir localmente, o Ollama **faz o download automático** antes de rodar.

---

## 🌍 Como descobrir modelos disponíveis na internet

Atualmente não há um comando tipo `ollama search` para listar todos os modelos remotos.
O caminho recomendado é usar a **biblioteca oficial de modelos**:

👉 **Model Library do Ollama**
[https://ollama.com/library](https://ollama.com/library)

Lá você encontra o nome exato para usar com:

```bash
ollama pull nome-do-modelo
ollama run nome-do-modelo
```

Exemplos de nomes que você pode encontrar lá:

* `llama3.2`, `llama3.3`, `llama4`
* `mistral`, `gemma3`, `phi4`, `deepseek-r1`
* versões especializadas (código, visão, etc.)

---

## 💬 Como fazer um prompt no terminal (modo básico)

### 7.1. Modo interativo

```bash
ollama run llama3.2
```

Você entra em um “chat” no terminal:

```text
>>> Olá, o que é o Ollama?
```

* Para sair: `/bye` ou `Ctrl + D`.

### 7.2. Modo “uma pergunta e resposta”

```bash
ollama run llama3.2 "Explique o que é um LLM em 1 frase."
```

Ou via pipe:

```bash
echo "Resuma o conceito de aprendizado de máquina em 2 frases." | ollama run llama3.2
```

---

## 🎚️ Como fazer um prompt com parâmetros (modo avançado)

Aqui usamos flags como `--temperature`, `--top-p` e `--num-predict` para controlar o comportamento do modelo.

### 8.1. Exemplo geral com parâmetros

```bash
ollama run llama3.2 \
  --temperature 0.4 \
  --top-p 0.9 \
  --num-predict 256 \
  "Resuma em 3 tópicos o que é o Ollama."
```

A seguir, explicações mais detalhadas de cada parâmetro 👇

---

### 8.1.1 🔥 `--temperature` (criatividade / variação)

Controla o quanto o modelo “arrisca” na escolha das palavras.

* Valores **baixos (0.1–0.3)** → respostas mais **secas, técnicas e estáveis**
* Valores **altos (0.7–1.0)** → respostas mais **criativas e variadas**

#### Exemplo 1 – Resumo técnico

```bash
ollama run llama3.2 \
  --temperature 0.1 \
  "Explique o que é um LLM em 2 frases, de forma técnica."
```

Você deve notar:

* Respostas bem parecidas se rodar várias vezes.
* Linguagem direta, pouca criatividade.

Agora, mudando só a temperatura:

```bash
ollama run llama3.2 \
  --temperature 0.9 \
  "Explique o que é um LLM em 2 frases, de forma técnica."
```

Você deve notar:

* Cada execução traz frases mais diferentes.
* Podem aparecer analogias e linguagem mais solta.

#### Exemplo 2 – Texto criativo

```bash
# Baixa criatividade
ollama run llama3.2 \
  --temperature 0.2 \
  "Escreva uma frase criativa sobre café."
```

Tende a vir algo simples, tipo:

> “Café é a bebida que nos mantém acordados e produtivos.”

Agora:

```bash
# Alta criatividade
ollama run llama3.2 \
  --temperature 0.9 \
  "Escreva uma frase criativa sobre café."
```

Pode vir algo como:

> “Café é o botão de ligar da alma em manhãs que ainda estão em modo de espera.”

💡 Regra prática:

* **0.1–0.3** → ideal para código, explicações técnicas, respostas previsíveis.
* **0.7–1.0** → ideal para ideias criativas, textos mais livres, brainstorming.

---

### 8.1.2 🎯 `--top-p` (controle de amostragem)

Controla **o quão “seguras” são as próximas palavras** que o modelo considera.

* `top-p` menor → o modelo considera só as palavras mais prováveis (mais conservador).
* `top-p` maior → o modelo permite alternativas menos prováveis (mais variedade).

#### Exemplo – nomes para app

```bash
# top-p baixo: bem conservador
ollama run llama3.2 \
  --temperature 0.7 \
  --top-p 0.3 \
  "Dê três sugestões de nomes para um app de lista de tarefas."
```

Provavelmente virão nomes bem óbvios:

* “Minha Lista”
* “Tarefas Hoje”
* “Lista Rápida”

Agora:

```bash
# top-p alto: mais variedade
ollama run llama3.2 \
  --temperature 0.7 \
  --top-p 0.95 \
  "Dê três sugestões de nomes para um app de lista de tarefas."
```

Podem vir nomes mais variados:

* “Caixa de Tarefas”
* “Checklistar”
* “Pronto & Feito”

💡 Regra prática:

* **0.3–0.5** → mais seguro e previsível, bom para coisas sérias.
* **0.8–0.95** → mais diversidade, bom para criatividade (especialmente com temperatura moderada/alta).

Dica:
Se a **temperatura já está baixa (0.1–0.3)**, o impacto do top-p é menor.
Se a **temperatura é alta (0.8–1.0)**, é bom segurar o `top-p` em algo como **0.7–0.9** para não ficar caótico.

---

### 8.1.3 ✂️ `--num-predict` (tamanho máximo da resposta)

Define o **máximo de tokens** (pedaços de palavras) que o modelo pode gerar.

* Valores baixos → respostas **curtas**.
* Valores altos → respostas **mais longas e detalhadas**.

#### Exemplo 1 – Resumo curto vs longo

```bash
# Resposta bem curtinha
ollama run llama3.2 \
  --num-predict 32 \
  "Explique rapidamente o que é o Ollama."
```

Tende a dar 1–3 frases curtas.

Agora:

```bash
# Resposta mais longa
ollama run llama3.2 \
  --num-predict 256 \
  "Explique rapidamente o que é o Ollama."
```

Aqui já pode virar um mini texto com vários parágrafos.

#### Exemplo 2 – Forçar uma saída curta (tipo título)

```bash
ollama run llama3.2 \
  --num-predict 16 \
  "Crie um título curto para um tutorial sobre Ollama no Ubuntu."
```

Isso força o modelo a não escrever um parágrafo inteiro.

💡 Regra prática:

* **16–64** → títulos, frases curtas, comandos.
* **128–256** → respostas padrão (explicações com exemplos).
* **512+** → textos grandes (artigos, documentação), mais custo de tempo e memória.

---

### 8.2. Combos prontos (para copiar/usar)

**Uso técnico / código:**

```bash
ollama run llama3.2 \
  --temperature 0.2 \
  --top-p 0.8 \
  --num-predict 256 \
  "Explique com exemplo como usar o parâmetro --temperature no Ollama."
```

**Brainstorm / ideias criativas:**

```bash
ollama run llama3.2 \
  --temperature 0.8 \
  --top-p 0.9 \
  --num-predict 256 \
  "Me dê ideias criativas de como usar o Ollama em projetos pessoais."
```

---

## 🌐 Como fazer prompt pela rede interna

Objetivo: usar **um PC como servidor Ollama** e acessar esse servidor de **outro PC** na mesma rede.

### 9.1. No servidor (onde o modelo está rodando)

Se estiver usando systemd, pare o serviço para não ter conflito:

```bash
sudo systemctl stop ollama  # se não existir, ignore o erro
```

Inicie o servidor expondo a porta na rede:

```bash
OLLAMA_HOST=0.0.0.0:11434 ollama serve
```

Teste localmente:

```bash
curl http://localhost:11434
# deve responder algo indicando que o Ollama está rodando
```

### 9.2. No cliente (outro PC na mesma LAN)

Descubra o IP do servidor (ex.: `192.168.0.50`) e no cliente:

```bash
export OLLAMA_HOST=http://192.168.0.50:11434

ollama list
ollama run llama3.2 "Teste de prompt via rede interna."
```

### 9.3. Usando a API HTTP direto

```bash
curl http://192.168.0.50:11434/api/chat \
  -d '{
    "model": "llama3.2",
    "messages": [{ "role": "user", "content": "Olá da rede interna!" }],
    "stream": false
  }'
```

---

## 🧹 Como remover modelos

Para liberar espaço em disco:

```bash
# remover um modelo específico
ollama rm llama3.2

# conferir se saiu
ollama list
```

Se precisar novamente, é só rodar:

```bash
ollama pull llama3.2
```

---

## ⏹️ Como desativar o Ollama

### 11.1. Se você rodou com `ollama serve` no terminal

* Use `Ctrl + C` no terminal onde está rodando.

### 11.2. Se estiver usando systemd

```bash
sudo systemctl stop ollama
```

Para não iniciar mais com o sistema:

```bash
sudo systemctl disable ollama
```

---

## 🗑️ Como remover completamente o Ollama

> ⚠️ Cuidado: isso remove o programa **e todos os modelos**.

1. Parar e remover o serviço:

```bash
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
sudo systemctl daemon-reload
```

2. Remover o binário:

```bash
sudo rm "$(which ollama)"
```

3. Remover libs/arquivos de instalação (ajuste o caminho se preciso):

```bash
sudo rm -rf /usr/lib/ollama /usr/local/lib/ollama 2>/dev/null
```

4. Remover dados do usuário (modelos, configs):

```bash
rm -rf ~/.ollama
```