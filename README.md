# Bot de Clima no Telegram com n8n

Workflow do desafio prático da Pós em IA e Automação. O chatbot recebe uma cidade brasileira pelo Telegram, consulta a temperatura atual na API OpenWeather e responde com uma mensagem curta em português.

## Funcionamento

Fluxo principal:

```text
Telegram Trigger
  -> Capturar e formatar entrada (variável `queue`)
  -> Entrada válida?
      -> sim: Consultar OpenWeather
          -> Extrair e formatar dados
          -> Resposta válida?
              -> sim: Enviar temperatura
              -> não: Enviar erro
      -> não: Enviar erro
```

Exemplo de entrada:

```text
São Paulo,SP
```

O workflow normaliza a entrada para `sao paulo,sp,br` antes da consulta.

Exemplo de sucesso:

```text
🌤️ A temperatura em São Paulo é de 25°C.
```

Exemplo de erro:

```text
❌ Cidade não encontrada. Use o formato Cidade,UF,BR (ex.: São Paulo,SP,BR).
```

## Requisitos

- n8n compatível com os nós usados no workflow (recomendado: versão atual estável).
- Bot do Telegram criado pelo [BotFather](https://t.me/BotFather).
- Chave da API [OpenWeather](https://openweathermap.org/api).
- URL pública HTTPS para o webhook do Telegram. O n8n Cloud já fornece essa URL; em uma instalação local, configure um domínio ou túnel HTTPS.

## Variáveis e credenciais esperadas

| Nome | Finalidade | Onde configurar |
| --- | --- | --- |
| `OPENWEATHER_API_KEY` | Autenticar a chamada ao OpenWeather | Variável de ambiente da instância n8n |
| `TELEGRAM_BOT_TOKEN` | Criar a credencial do bot | Credencial **Telegram API** no n8n; nunca inserir no JSON |

O arquivo exportado não contém chaves, tokens ou outros segredos. Ele contém somente uma referência à credencial chamada `Telegram Bot` e a expressão `{{ $env.OPENWEATHER_API_KEY }}`.

## 1. Configurar a chave do OpenWeather

Defina `OPENWEATHER_API_KEY` no ambiente que executa o n8n e reinicie a instância.

PowerShell, apenas para a sessão atual:

```powershell
$env:OPENWEATHER_API_KEY="sua-chave-aqui"
npx n8n
```

Docker Compose:

```yaml
environment:
  - OPENWEATHER_API_KEY=${OPENWEATHER_API_KEY}
```

Crie um arquivo `.env` local, que não deve ser enviado ao GitHub:

```dotenv
OPENWEATHER_API_KEY=sua-chave-aqui
TELEGRAM_BOT_TOKEN=seu-token-aqui
```

> Em instalações que bloqueiam o acesso de nós às variáveis do ambiente, ajuste a configuração administrativa da instância ou substitua a expressão por uma credencial segura do n8n. Nunca grave a chave no workflow exportado.

## 2. Criar a credencial do Telegram

1. No Telegram, abra o BotFather e execute `/newbot`.
2. Escolha o nome e o username do bot.
3. Guarde o token recebido em local seguro.
4. No n8n, acesse **Credentials** e crie uma credencial do tipo **Telegram API**.
5. Cole o valor de `TELEGRAM_BOT_TOKEN` no campo **Access Token**.
6. Nomeie a credencial como `Telegram Bot`.

Não publique o token no GitHub. Se um token for exposto, revogue-o imediatamente no BotFather e gere outro.

## 3. Importar o workflow

1. Baixe ou clone este repositório.
2. No n8n, crie um workflow e escolha **Import from File**.
3. Selecione `workflow-chatbot-telegram.json`.
4. Abra os nós `Telegram Trigger`, `Enviar temperatura` e `Enviar erro`.
5. Selecione a credencial `Telegram Bot` nos três nós.
6. Salve o workflow.

## 4. Testar e ativar

1. No n8n, clique em **Test workflow** ou execute o `Telegram Trigger` em modo de teste.
2. Abra a conversa com o bot no Telegram e envie `/start` se necessário.
3. Envie uma cidade no formato `Cidade,UF` ou `Cidade,UF,BR`.
4. Confirme que todos os nós executaram sem erro.
5. Ative o workflow para registrar o webhook de produção.

Casos mínimos de teste:

| Entrada | Resultado esperado |
| --- | --- |
| `São Paulo,SP` | Mensagem com temperatura de São Paulo |
| `Curitiba,PR,BR` | Mensagem com temperatura de Curitiba |
| `Belo Horizonte,MG` | Mensagem com temperatura de Belo Horizonte |
| `Grifória,SP,BR` | Mensagem de cidade não encontrada |
| `São Paulo` | Mensagem orientando o formato correto |

## Nós do workflow

- **Telegram Trigger:** recebe mensagens de texto.
- **Capturar e formatar entrada:** grava o texto na variável `queue`, remove acentos excedentes, normaliza espaços, vírgulas e letras minúsculas e acrescenta `BR` quando necessário.
- **Entrada válida?:** exige `Cidade,UF` ou `Cidade,UF,BR`.
- **Consultar OpenWeather:** chama `https://api.openweathermap.org/data/2.5/weather` com `q`, `units=metric`, `lang=pt_br` e `appid={{ $env.OPENWEATHER_API_KEY }}`. O nó retorna o status HTTP completo, não interrompe o fluxo em respostas não-2xx, usa timeout de 10 segundos e faz até três tentativas em falhas de rede.
- **Extrair e formatar dados:** valida o status e os campos `main.temp` e `name`, arredonda a temperatura e produz a mensagem final.
- **Resposta válida?:** direciona sucesso e erro.
- **Enviar temperatura / Enviar erro:** respondem no mesmo `chat.id` recebido pelo trigger.

## Segurança

- Nenhum segredo real deve aparecer no JSON, README, histórico do Git ou capturas públicas.
- Use `.gitignore` para impedir o commit de `.env`.
- Revogue imediatamente qualquer token compartilhado por engano.
- O workflow permanece inativo após a importação até que as credenciais sejam configuradas e o usuário o ative.

## Entrega

O repositório público deve conter, no mínimo:

```text
README.md
workflow-chatbot-telegram.json
```

