# tuya-neway-automacoes

Nós para o Node-RED que controlam dispositivos Tuya (Smart Life / Tuya Smart) pela nuvem da Tuya: lâmpadas, tomadas, interruptores, relés e outros. O login é por QR code no próprio editor do Node-RED, como a integração Tuya do Home Assistant, **sem criar projeto na Tuya IoT Platform**, sem Access ID e sem Access Secret.

Neway Automações.

Aviso: este pacote usa o mesmo acesso "device-sharing" que o Home Assistant usa. Não é uma API pública com garantia. Se a Tuya mudar esse acesso, o pacote precisa de atualização.

Dependências: `mqtt` e `qrcode-generator` (instaladas junto com o pacote).

## O que vem no pacote

Dois nós, no grupo **tuya neway automacoes** da paleta:

- **Tuya** (`tuya-dispositivo`): lê o estado, consulta e envia comandos a um dispositivo.
- **Conta Tuya** (`tuya-conta`, nó de configuração): guarda o login e a ligação com a nuvem. Todos os nós Tuya usam uma conta.

O estado dos dispositivos chega em tempo real pelo MQTT da própria Tuya (sem consultar de tempos em tempos). Os comandos vão pela API da Tuya.

## Instalar

A instalação é feita dentro do próprio Node-RED.

1. Baixe o arquivo `.tgz` deste repositório.
2. No Node-RED, abra o menu (três barras), **Gerenciar paleta**, aba **Instalar**, e envie o arquivo `.tgz` (botão de enviar/upload).
3. Confirme a instalação. Na paleta aparece o grupo **tuya neway automacoes** com o nó **Tuya**. Se não aparecer, reinicie o Node-RED.

Se você já tinha o pacote anterior `tuya-neway-nuvem`, desinstale-a antes (veja Desinstalar): os tipos de nó são os mesmos e conflitam.

## Conta (feita uma vez)

1. Arraste um nó **Tuya** para o fluxo, dê duplo clique e, no campo **Conta**, clique no lápis para criar uma conta.
2. Preencha:

| Campo | O que é |
|---|---|
| Nome | qualquer nome, só para você reconhecer |
| User Code | no app Smart Life (ou Tuya Smart): **Eu**, ícone de engrenagem, **Conta e Segurança**, **User Code** |

3. Clique em **Gerar QR**. O QR aparece no editor.
4. No app, escaneie o QR (ícone de escanear no app) e confirme o login. O editor detecta sozinho e preenche o login.
5. Clique em **Adicionar/Atualizar** e faça o **Deploy**.

A etiqueta da conta mostra `online (N disp.)` quando está funcionando, com a quantidade de dispositivos encontrados. O login fica salvo e se renova sozinho; ele sobrevive a reinícios e a atualizações do pacote. O botão **Ver dados brutos** (no editor da conta) mostra as últimas respostas da Tuya, útil para diagnóstico.

## Usar o nó Tuya

Escolha:

- **Conta:** a conta criada acima.
- **Dispositivo:** a lista traz os dispositivos da sua conta (todas as casas).
- **Código:** o ponto de dados do aparelho (por exemplo `switch_1`, `switch_led`, `bright_value_v2`). A lista mostra os códigos do dispositivo escolhido. Em "Todos os códigos", o nó trata todos.
- **Tipo:** define as portas do nó.

### Tipo: Status (só saída)

Emite uma mensagem por código que mudou, em tempo real. Com **Emitir ao iniciar**, o estado atual sai assim que a conta carrega.

Saída:

| Campo | Conteúdo |
|---|---|
| `msg.payload` | o valor (`true`/`false`, número ou texto) |
| `msg.topic` | o código (ex.: `switch_1`) |
| `msg.device` | id do dispositivo |
| `msg.name` | nome do dispositivo |
| `msg.status` | todos os códigos e valores atuais |

### Tipo: Leitura (entrada e saída)

Qualquer mensagem na entrada consulta o aparelho na nuvem e devolve na saída o valor atual do código escolhido (ou de todos). Útil para ler o estado sob demanda. Se a consulta falhar, devolve o último estado conhecido.

### Tipo: Comando (só entrada, sem saída)

Envia o comando ao aparelho. O nó não tem saída.

- **Com um Código escolhido:** `msg.payload` é o valor. `ON`/`true` viram verdadeiro, `OFF`/`false` viram falso e texto numérico vira número.
- **Com o Código em "Todos":** o código vem de `msg.topic` e o valor de `msg.payload`.
- **Vários códigos de uma vez:** `msg.payload` pode ser um objeto, como `{"switch_1": true, "switch_2": false}`, ou uma lista `[{"code":"switch_1","value":true}]`.

A etiqueta do nó mostra o resultado do comando (verde) ou o erro (vermelho).

## Exemplos

**Tomada ou interruptor.** Dispositivo: a tomada. Código: `switch_1`. Tipo: Comando. Um nó *inject* com `payload` = `ON` liga; outro com `OFF` desliga.

**Lâmpada smart.** Os códigos mais comuns (confira na lista do seu aparelho, que varia por modelo):

| Código | O que faz | Valor |
|---|---|---|
| `switch_led` | liga e desliga | `true` / `false` |
| `work_mode` | modo | `white`, `colour`, `scene` ou `music` |
| `bright_value_v2` | brilho | 10 a 1000 (modelos antigos: `bright_value`, 25 a 255) |
| `temp_value_v2` | temperatura da luz branca | 0 (quente) a 1000 (fria) |
| `colour_data_v2` | cor | texto JSON `{"h":0-360,"s":0-1000,"v":0-1000}` |

Com Código em "Todos", um nó *function* pode montar: `msg.topic = "bright_value_v2"; msg.payload = 500; return msg;`. Para a cor, mande o valor como texto JSON (`JSON.stringify({h:120,s:1000,v:1000})`) e não como objeto.

**Reagir a um sensor ou a um botão.** Tipo: Status. Ligue a saída a um nó *switch* ou *function*. Cada mudança de código vira uma mensagem.

**Ler o estado quando quiser.** Tipo: Leitura. Um *inject* na entrada e um *debug* na saída.

## Etiqueta e estados

A etiqueta do nó mostra `código=valor` (verde quando verdadeiro), `offline` em vermelho, e os avisos `sem conta` e `sem dispositivo` quando falta configurar. A etiqueta da conta mostra: `sem login`, `carregando`, `online (N disp.)`, `mq caiu, reconectando` ou o erro.

## Problemas comuns

| Sintoma | O que fazer |
|---|---|
| Conta em `sem login` | gerar o QR de novo e escanear |
| Lista de dispositivos vazia | fazer o Deploy da conta e esperar `online`; depois reabrir o nó |
| Instalou e aparecem nós duplicados ou erro de tipo | desinstalar o pacote anterior `tuya-neway-nuvem` (veja Desinstalar) |
| Editor mostra o nó como era antes da atualização | reiniciar o Node-RED |
| Comando recusado | conferir o código e o valor aceitos pelo aparelho (veja *Ver dados brutos*) |
| Parou de funcionar de repente | a Tuya pode ter mudado o acesso; abra uma issue neste repositório com a etiqueta da conta e o *Ver dados brutos* (apague tokens e senhas antes de colar) |

## Atualizar

Baixe o `.tgz` atual e instale do mesmo jeito (Gerenciar paleta, Instalar, enviar o arquivo). Se a atualização mudar os campos do nó, reinicie o Node-RED para o editor mostrar a mudança. O login da conta é mantido.

## Desinstalar

Antes de remover o pacote, apague tudo que o usa. O Node-RED não remove um pacote que ainda tem nós em uso, e a conta guarda o login:

1. Apague **todos os nós Tuya** de todos os fluxos.
2. Apague a **conta Tuya salva**: menu, **Configuration nodes**, escolha a conta Tuya e exclua.
3. Faça o **Deploy**.
4. Menu, **Gerenciar paleta**, aba **Nós**, ache `tuya-neway-automacoes` e clique em **Remover**.

Isso apaga o login salvo no Node-RED. Se quiser voltar a usar depois, será preciso gerar o QR de novo.
## Segurança

O login (tokens) fica nas credenciais do Node-RED e em um arquivo `tuya_sessao_<id>.json` na pasta de dados. Quem tem acesso à sua pasta de dados ou ao editor do Node-RED consegue controlar seus aparelhos Tuya. Proteja o administrador do Node-RED com senha se ele estiver exposto na rede. O botão *Ver dados brutos* mostra ids de dispositivos e estados; não cole o conteúdo em lugares públicos sem apagar o que for pessoal.

## Limites conhecidos

- Controle pela nuvem: precisa de internet.
- Cenas da Tuya ainda não são suportadas.
- Os valores saem como a Tuya envia (sem conversão de escala).
- A Leitura (consulta de detalhe na nuvem) foi escrita de acordo com o que a API devolve, mas pode precisar de ajuste em alguns aparelhos.
