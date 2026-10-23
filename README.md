# int443

> Projeto em fase inicial para explorar a deteção e substituição local de dados sensíveis em pedidos enviados a serviços de IA. As secções de produto, mercado e arquitetura abaixo registam hipóteses e decisões de exploração; não descrevem funcionalidades já disponíveis.

## Origem do nome

**int443** combina *int*, abreviação de *interceptor*, com **443**, a porta TCP normalmente usada como destino pelas ligações HTTPS de saída para serviços de IA. A porta 443 não é exclusiva de serviços de LLM e, numa ligação de saída, não é a porta de origem do computador: é a porta do serviço remoto.

## Estado do repositório

O repositório contém um esqueleto Rust com dois crates:

- `int443`: CLI em `clap`, com os comandos `run` e `check` definidos. A execução de ambos ainda termina com uma mensagem de funcionalidade não implementada.
- `int443-engine`: biblioteca reservada ao motor de deteção, substituição e restauro; ainda sem implementação funcional.

**Ainda não existe um proxy**, inspeção de tráfego, deteção ou substituição de dados, restauro de valores, configuração persistente ou instalador. Não uses este protótipo para proteger tráfego real nem assumes que os dados são filtrados.

## Começar

### Requisitos

- Rust estável 1.85 ou superior, instalado através do [rustup](https://rustup.rs).

### Explorar a CLI

```sh
cargo run -p int443 -- --help
cargo run -p int443 -- run --help
```

Estes comandos mostram apenas a interface atualmente definida. `run` e `check` ainda não executam o proxy nem validam uma configuração.

### Verificar o workspace

```sh
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```

## Estrutura do código

| Caminho | Conteúdo |
|---|---|
| `crates/int443` | Binário e interface de linha de comandos |
| `crates/int443-engine` | Biblioteca planeada para deteção, substituição e restauro |
| `CONTRIBUTING.md` | Requisitos e fluxo de contribuição |
| `guiao-es.md` | Guião de entrevista em espanhol |
| `guiao-fr.md` | Guião de entrevista em francês |

## Navegação

| Tema | Secção |
|---|---|
| Objetivo e âmbito do produto | [Objetivo](#objetivo), [Âmbito](#âmbito) |
| Proposta e arquitetura | [A promessa "enviar tudo"](#a-promessa-enviar-tudo-e-o-que-ela-exige), [Como funciona](#como-funciona-desenho-técnico-provisório), [Stack e ferramentas](#stack-e-ferramentas) |
| Mercado e validação | [Panorama de mercado](#panorama-de-mercado-2026-10-20), [Plano de validação](#plano-de-validação-4-semanas-proposto) |
| Entrevistas | [Guião de entrevista](#guião-de-entrevista-pme), [Espanhol](guiao-es.md), [Francês](guiao-fr.md) |
| Privacidade e conformidade | [Presets de deteção](#presets-de-deteção-para-portugal-pesquisa-de-2026-10-20), [RGPD](#rgpd-e-enquadramento-legal-a-validar-com-um-jurista) |
| Desenvolvimento | [CONTRIBUTING.md](CONTRIBUTING.md) |

## Objetivo

Impedir que dados sensíveis cheguem aos servidores das empresas de LLMs. O int443 é um **intermediário local** que apanha o tráfego entre o dispositivo e o fornecedor de LLM, deteta os dados sensíveis, substitui-os por um valor literal neutro (marcador) e **não guarda nada do que viu**. O fornecedor só recebe a versão já limpa.

Imagem mental: "um Wireshark, mas específico para isto" — com a diferença de que não só lê como limpa o tráfego antes de sair.

**Promessa ao utilizador (autor, 2026-10-20):** poder enviar ao LLM tudo o que quiser, sem pensar no que é sensível, porque o int443 trata disso por trás e o LLM faz o trabalho na mesma. É o que o distingue de ferramentas que avisam ou bloqueiam. O objetivo é **reduzir drasticamente o risco**, não eliminá-lo.

## Contexto

- Ideia nova. Ninguém usa hoje (decisão do autor, 2026-10-20).
- O int443 é para ser um **negócio novo** (autor, 2026-10-20) — não um projeto interno nem só pessoal. Por isso o critério de avanço é haver quem pague, e não apenas funcionar tecnicamente.
- O problema de partida: dados sensíveis enviados para servidores de LLMs ficam fora do controlo de quem os enviou.
- Pesquisa de mercado feita a 2026-10-20: **a ideia não está vazia** — ver "Panorama de mercado".

## Quem e recursos (2026-10-20)

| Tema | Estado |
|---|---|
| Autor | CEO e founder do int443 |
| Sócios e equipa | Não definido |
| Tempo, dinheiro e competências disponíveis | Não definido |
| O que é sucesso (receita, prazo, negócio principal ou secundário) | Não definido |
| Datas (início da validação, decisão go / no-go) | Não definidas |
| Roadmap do autor | Por definir; não recebido |
| Notas das entrevistas | Por preencher quando acontecerem |

## Âmbito

**Dentro:**
- Corre **localmente** (decidido).
- Interceta **só o tráfego para serviços de LLM** (lista de hosts); o resto do tráfego passa sem ser aberto (decidido, 2026-10-20).
- Deteta e substitui dados sensíveis no tráfego para serviços de LLM.
- Não persiste conteúdo sensível.
- Destinado a **empresas de todos os tipos, com foco em PME e pequenas empresas**, no contexto do **RGPD** (decidido, 2026-10-20). O cliente define o que é sensível; o int443 traz presets alinhados com o RGPD (ex.: NIF).

**Fora / por decidir:** ver "Perguntas em aberto".

## A promessa "enviar tudo" e o que ela exige

O cliente compra poder usar IA sem medo. A promessa só se cumpre se o int443 satisfizer estes requisitos (propostas, a decidir):

1. **Avisar logo quando falha, e mostrar o que fez.** Decisão do autor (2026-10-20): se o proxy tiver problemas, o utilizador é avisado de imediato de que o tráfego poderá passar sem filtragem (o pedido não é bloqueado). Quem sabe que o int443 protege envia mais, por isso o aviso tem de ser impossível de ignorar (notificação do Windows com a causa) e o int443 tenta recuperar sozinho (reinício automático do serviço). Risco a ter presente: durante a falha os dados sensíveis saem em claro, que é exatamente o cenário em que o utilizador confia no int. Proposta: modo configurável por cliente (avisar e deixar passar, ou bloquear), com registo só de metadados da janela sem filtragem. Na comunicação comercial: o objetivo é **reduzir drasticamente o risco**, nunca "garantir conformidade".
2. **Cobertura acima da precisão.** Preferir falsos positivos a falsos negativos, com regras do cliente para afinar.
3. **Restauro em memória como base.** Para o utilizador receber a resposta certa, os valores trocados têm de voltar na resposta. O mapeamento vive só em memória e por conversa. As APIs reenviam o histórico em cada pedido, por isso a correspondência tem de ser consistente. Em respostas em streaming, um marcador pode chegar partido entre pedaços; o proxy tem de o reconstruir antes de restaurar.
4. **Substitutos que não estraguem a tarefa.** Marcador (`[PESSOA_1]`) ou valor falso coerente? Valores falsos com aparência real (um NIF com dígito de controlo válido) ajudam o modelo a raciocinar mas podem confundir quem lê. A decidir com testes.
5. **O int443 retira identificadores, não a substância.** "Analisa este contrato" funciona sem o nome da pessoa, mas as cláusulas, os valores e a informação de saúde continuam a ir. Informação confidencial da empresa (estratégia, código por lançar) e segredos técnicos (chaves de API) são problemas diferentes de dados pessoais. O âmbito tem de dizer o que cobre. Por decidir.
6. **Ficheiros, imagens e voz.** "Tudo" inclui anexos (PDF, Excel, capturas de ecrã) que o proxy teria de abrir e, nas imagens, ler por OCR. Numa primeira versão: só texto, com anexos bloqueados ou avisados. Caso contrário a promessa é falsa.

## Entrega: conta web + agente local (.exe com linha de comandos)

Modelo descrito pelo autor (2026-10-20): alguém cria uma conta na web, configura, e descarrega um `.exe`. O mesmo executável é o serviço e uma ferramenta de linha de comandos para gerir os dados sensíveis. Exemplos do autor (significado a confirmar):

```
int25 --set --add --sensdata "keyvaultvalue"
int25 --cg --sensdata keyvaultvalue --to "12346"
```

Leitura provisória: o primeiro acrescenta um valor sensível definido pelo cliente; o segundo altera o valor de substituição para `12346` (`--cg` talvez "change").

Pontos de desenho (propostas):

1. **Onde vivem os valores sensíveis.** A conta web guarda só configuração (presets, regras, políticas, licença, faturação). Os valores sensíveis em si (nomes de clientes, NIF específicos) **ficam só no dispositivo**, num armazém cifrado local, nunca sincronizados. Se a conta web os guardasse, o fabricante passava a deter os dados do cliente: quebra a promessa "local", torna-o subcontratante para o RGPD e alvo de fugas.
2. **Nunca passar o valor na linha de comandos.** Os argumentos ficam no histórico do PowerShell, na lista de processos e, com auditoria ativa, nos registos de eventos do Windows. Alternativas: pedir o valor de forma oculta, ler do stdin ou de um ficheiro, ou usar referências por nome (`--sensdata nome_cliente`) em vez do valor.
3. **Quem pode usar a CLI.** Fala com o serviço por um canal local (named pipe) com permissões: só administradores, ou configuração assinada vinda da conta web, podem acrescentar, alterar ou desligar regras. Senão qualquer programa no PC desativa a proteção.
4. **O instalador parece malware.** Um `.exe` descarregado da web que instala uma CA de raiz e interceta TLS é o que antivírus e EDR bloqueiam e o que os trabalhadores aprendem a não abrir. Precisa de assinatura de código, reputação junto do SmartScreen/Defender e de pacote MSI para Intune/GPO. Numa PME sem TI, um aviso do SmartScreen pode matar a instalação.
5. **O canal de atualização é o maior risco de segurança do negócio.** Se alguém o comprometer, instala software malicioso, numa máquina com CA de raiz, em todos os clientes. Exige atualizações assinadas, distribuição por fases e chaves de assinatura bem guardadas.
6. **Dicionário do cliente.** Valores definidos pelo cliente com substituição explícita (`→ 12346`) têm mapeamento persistente e local, por isso o restauro funciona mesmo depois de um reinício (resolve, para estas entradas, o problema do mapeamento em memória). Limites: maiúsculas, acentos, plurais, erros de escrita e correspondências parciais precisam de normalização.
7. **Guia de configuração no site (autor, 2026-10-20).** O site traz uma espécie de README que ensina a configurar. Para uma PME sem TI, isto faz parte do produto: em português, com passos curtos (criar conta, descarregar, instalar, adicionar o primeiro valor sensível, testar), a referência da CLI e o que fazer quando aparece o aviso de falha. Hipótese: acrescentar um kit RGPD (modelo de DPIA, texto de informação aos trabalhadores, descrição do que o int443 faz e não faz), porque a monitorização de trabalhadores é um dos maiores bloqueios à adoção.

## Distribuição e licença (open source)

> **Atualização (2026-10-20): o autor prefere não ser open source.** Caminho em vigor: **código fechado** (opção B1), com token e cobrança por dado. O resto desta secção fica como registo das opções e dos riscos que levaram a essa escolha. Abrir mais tarde (fonte disponível ou open core) continua possível; fechar depois de publicar não é. A documentação pública (o guia de configuração no site) mantém-se, sem expor o código.

**Sem código aberto: como ganhar confiança.** Um programa que instala uma CA e abre TLS, e que ninguém pode auditar, é uma venda difícil. Substitutos possíveis:
- **Auditoria de segurança e teste de intrusão por uma entidade independente**, com resumo público.
- **Especificação pública do que o int443 envia** (só metadados de contagem, validação da licença e atualizações). O cliente pode verificá-la ele próprio com uma captura de rede, porque o int443 corre na máquina dele.
- **Registo local do que foi substituído** (metadados), visível ao cliente.
- **Versões assinadas e `SECURITY.md`** com contacto para divulgação de falhas e, mais tarde, um programa de recompensas. Nada disto exige código aberto.
- **Mais tarde:** certificações (ISO 27001, SOC 2), e a possibilidade de auditoria do código por clientes sob acordo de confidencialidade.

**O que o código fechado custa:** a biblioteca para programadores perde força (preferem componentes abertos), por isso a prioridade passa a ser o agente para PME; e a cobrança assenta em clientes honestos e em contrato, porque o binário local pode ser adulterado por quem se dê ao trabalho.

Ideia do autor (2026-10-20): distribuir pelo GitHub, com README público e licença permissiva ("MIT 2.0" não existe: é **MIT** ou **Apache-2.0**), e depois o int443 poder ser usado como ferramenta por outras apps.

**O que ganha**
- **Confiança verificável.** Um programa que instala uma CA e abre TLS pede confiança. Com o código público, "não guarda nada" pode ser auditado. Alinha com o diferencial "local e auditável".
- Adoção por programadores e empresas de informática, e sinais de procura (estrelas, descargas, issues).

**O que custa**
- **O código deixa de ser o que se vende.** Com licença permissiva, qualquer um o pode usar, incluindo concorrentes e empresas de informática que o instalam aos clientes sem pagar. O Presidio (da Microsoft) já é MIT e o LocalAI e o contextio são abertos: o int443 entra num espaço onde já há código gratuito.
- **Passa a cobrar-se outra coisa:** gestão central (a conta web), instalador assinado e MSI, presets e deteção de melhor qualidade, kit RGPD, suporte e garantia. É o modelo "open core"; o que fica fechado tem de ser escolhido com cuidado.
- **A licença não protege a ideia, só o código.** Com Apache-2.0 qualquer pessoa pode copiar, alterar (mesmo só a cor), rebatizar e vender o código, até em versão fechada; só tem de manter os avisos de autor e de licença, e não pode usar a tua marca. O direito de autor também não protege a ideia: qualquer pessoa pode reimplementá-la do zero, e com ferramentas de IA isso ficou barato. A proteção real está noutro lado: marca registada, partes que não estão no repositório (regras e atualizações no servidor, builds assinados), contratos com clientes, confiança e rapidez. Copyleft (AGPL) impede versões fechadas mas não a concorrência; fonte disponível (FSL/BSL) proíbe o uso concorrente por um período, mas fazer cumprir é caro para uma empresa pequena.
- **A licença é difícil de mudar depois:** as versões já publicadas ficam com a licença antiga, e mudar exige o acordo de quem contribuiu.

**Licenças a considerar** (não sou jurista; validar):

| Licença | A favor | Contra |
|---|---|---|
| MIT | Muito simples, adoção máxima | Sem cláusula de patentes nem de marca |
| Apache-2.0 | Como a MIT, com concessão explícita de patentes e sem direitos sobre a marca; habitual em projetos para empresas | Um pouco mais longa |
| AGPL-3.0 | Quem oferecer o int443 como serviço tem de partilhar as alterações; dissuade grandes fornecedores | Muitas empresas evitam AGPL; atrapalha o uso como biblioteca em apps fechadas |
| Fonte disponível (ex.: FSL, BSL) | Impede o uso concorrente durante um período | Não é open source; perde parte da confiança e da adoção |

**"Ferramenta para apps e tudo":** são duas audiências diferentes — PME (proxy com instalador) e programadores (biblioteca). A biblioteca compete diretamente com Presidio, LLM Guard e plugins de gateways. Sugestão: um **motor** comum (deteção, substituição, restauro, presets PT) com duas formas — agente para o utilizador final e biblioteca/CLI para programadores. Começar pelo motor.

**Repositório público**
- **Este README é interno** (estratégia, preços, concorrência). O README público do repositório é outro: o que faz, como instalar, como configurar, modelo de segurança e o que não garante. As notas de estratégia ficam fora do repositório.
- Nunca incluir dados de clientes, dicionários nem chaves. Ter um `SECURITY.md` com contacto para divulgação de falhas.
- A licença não dá direitos sobre a marca. "int" é genérico e pode colidir com outros usos; verificar o nome e registar a marca.
- Lançamentos assinados e reproduzíveis: mitigam o risco de uma atualização comprometida.

## Como funciona (desenho técnico, provisório)

- **Não é um editor de pacotes: é um proxy que termina o TLS.** Para ler e alterar HTTPS é preciso receber a ligação do cliente com um certificado próprio (uma CA instalada em cada dispositivo) e abrir uma segunda ligação cifrada para o fornecedor. Alterar bytes "a meio" sem isto não é possível (tamanhos, números de sequência e integridade do TLS deixam de bater). O Wireshark só decifra com as chaves de sessão.
- **O router não é o ponto crítico.** O router só vê tráfego cifrado. O que importa é limpar **antes de o tráfego sair do dispositivo/rede** — logo, um proxy no próprio dispositivo é mais simples e apanha mais do que uma caixa na rede.
- **Zonas onde falha:** apps com certificate pinning ou com conjunto de certificados próprio, QUIC/HTTP3, e streams SSE/WebSocket das respostas. Os produtos comerciais metem estas apps em listas de exceção.
- **Substituição:** irreversível (a resposta traz marcadores) ou reversível só em memória durante a sessão (restaura-se na resposta, o mapeamento desaparece no fim). Por decidir.
- **A qualidade da deteção é o produto.** Regex apanha NIF/IBAN; nomes, moradas e termos internos pedem modelos de linguagem. Falsos negativos deixam passar dados; falsos positivos estragam o prompt.
- **O int443 é o componente mais sensível:** vê tudo em claro, e a chave privada da CA dá acesso a tudo o que passa. Usado sobre tráfego de terceiros (funcionários, família) implica RGPD e regras de monitorização.

## Panorama de mercado (2026-10-20)

Quase tudo vem de blogs de fornecedores; as afirmações comerciais não estão verificadas. A pesquisa só devolve resultados dos EUA.

| Família | Exemplos | Nota |
|---|---|---|
| Proxy MITM local de código aberto | [LocalAI MITM proxy](https://localai.io/docs/features/mitm-proxy/), [contextio](https://github.com/larsderidder/contextio) | **Mais próximos do int.** Redação de PII no tráfego do Claude Code/Codex CLI, só em hosts de LLM permitidos. Foco em ferramentas de programação |
| Inspeção inline comercial (SSE) | Netskope, Zscaler, Cloudflare | DLP sobre prompts; Netskope diz que redige em vez de bloquear. Enterprise, pago |
| DLP no endpoint/browser | dope.security, Prompt Security, Harmonic, Nightfall, Microsoft Purview | Foco em impedir que funcionários colem dados |
| Extensões de browser de anonimização (grátis) | Caviard, PrivacyKit, ONYRI Sanitize, Private Prompt, MaskMyPrompt | Substituem por marcadores (algumas reversíveis), no browser, sem gestão central nem auditoria. Fixam o preço-base para uso individual |
| Gateways com redação de PII | LiteLLM, Kong AI Gateway, Gravitee, Cloudflare AI Gateway | Exigem controlar a aplicação que chama o LLM |
| Tokenização reversível | Protecto (a partir de ~250 $/mês, fonte secundária), Presidio, LLM Guard, Limina | Mapeamento guardado fora do modelo |
| Inferência confidencial (TEE) | Confident Security (4,2 M$ seed, jul. 2025), Tinfoil, Confer, Privatemode | Protege do lado do fornecedor; sobretudo modelos open-weight |
| Big tech, produtos próprios | Apple Private Cloud Compute, Google Private AI Compute, WhatsApp Private Processing | Não disponível a terceiros |

Observação: não encontrei violação confirmada em servidores de um grande fornecedor de LLM em 2026. As fugas encontradas foram em apps wrapper, gateways (LiteLLM) e credenciais roubadas por infostealers. Convém não basear a proposta de valor em "os fornecedores vazam".

**Espaço possível (hipóteses, não achados):** local e auditável (assim "não guardo nada" é verificável); PME europeias sem orçamento de plataforma enterprise (Netskope/Zscaler); cobertura de identificadores portugueses (ver "Presets de deteção para Portugal" — o Microsoft Purview e o Azure já cobrem NIF e Cartão de Cidadão, por isso isto sozinho não diferencia o int); ângulo RGPD (menos dados pessoais a sair para fornecedores fora da UE).

## A Microsoft já faz algo parecido? (2026-10-20)

**Sim, em parte — e a caminhar para a mesma arquitetura do int.** Resumo de uma pesquisa em documentação da Microsoft e análises de terceiros; as datas de lançamento e as licenças variam entre fontes.

| Camada | O que faz | Limites encontrados |
|---|---|---|
| Edge for Business (proteção inline) | Inspeciona em tempo real o texto escrito ou colado em prompts e pode avisar ou bloquear (com ou sem justificação do utilizador) antes de sair do browser. Apps de IA de consumo listadas: ChatGPT, Copilot (consumo), DeepSeek, Gemini; a lista vai crescer | Só no Edge. Não vi o Claude na lista. A ação é avisar ou bloquear, não redigir |
| Endpoint443 DLP | Bloqueia colagens e uploads para sites de IA em dispositivos geridos | Só nos browsers suportados; segundo fonte secundária não avalia apps de desktop. Licença E5 ou equivalente |
| Network data security (Entra Global Secure Access + Purview) | Inspeciona tráfego HTTP/HTTPS no dispositivo, com inspeção TLS, incluindo prompts e respostas de IA, e aplica políticas DLP; em teoria apanha browsers, apps e APIs | Exige inspeção TLS, dispositivos ligados ao Entra e M365 E5/E7 + Entra Internet Access. Falha com QUIC, DNS-over-HTTPS, certificate pinning e browsers não suportados — os mesmos problemas do int. Segundo uma página da Microsoft, disponibilidade geral em setembro de 2026; as fontes divergem nas datas |
| Copilot | Não responde se o prompt tiver um tipo de dado sensível | Só Copilot; bloqueia, não redige |

**Licenças.** Pelas tabelas oficiais, o Business Premium (a licença típica de PME) não inclui Endpoint443 DLP nem a proteção inline de browser/rede. Inclui DLP para Exchange, SharePoint443 e OneDrive. Existe um add-on Purview Suite para Business Premium (limite de 300 utilizadores; cerca de 10 $/utilizador/mês segundo uma resposta da comunidade, não verificado). Outra análise de 2026 contradiz isto, e a Microsoft descreve a proteção do Edge para apps não geridas como pay-as-you-go. Confirmar com a licença real de cada PME.

**Não encontrei nenhuma ação nativa de redação** (substituir e continuar) nos prompts, só avisar ou bloquear. Não é prova de que não exista.

**O que isto significa para o int443**
- **Valida a abordagem técnica:** a Microsoft usa inspeção TLS no dispositivo e tem os mesmos pontos fracos.
- **Confirma um concorrente forte e em movimento** (proteção de rede disponível desde setembro de 2026). Pode acrescentar redação mais tarde.
- **Diferenciais possíveis:** redigir e continuar em vez de bloquear; processamento local (segundo um blogue, a Microsoft encaminha elementos do tráfego para o Purview classificar); preço de PME sem E5; funcionar fora do ecossistema Microsoft (Mac, Google Workspace); presets PT.
- **Hipótese a testar nas entrevistas:** quando se bloqueia, os funcionários contornam a ferramenta; quando se redige, continuam a usar a IA de forma segura.

## Vale a pena avançar? (avaliação de 2026-10-20)

**Veredicto provisório:** não construir já um produto; **validar primeiro, com tempo e dinheiro limitados (2–4 semanas)**. A pergunta decisiva — haverá PME a pagar? — não tem resposta, e não encontrei nenhum estudo sobre disposição de PME para pagar por segurança de IA.

**A favor**
- Problema real e crescente: estudos de fornecedores indicam que a maioria dos trabalhadores cola dados em prompts (LayerX 2025: 77%) e que cerca de 40% do que vai para ferramentas de IA é sensível (Cyberhaven 2026). Poucos estudos independentes por trás destes números.
- **Possível lacuna (revista a 2026-10-20):** a Microsoft já cobre prompts no browser (Edge) e, na camada de rede, também apps e APIs, mas só em licenças E5 ou com add-on, e só avisa ou bloqueia — não encontrei redação. Fica espaço para quem redige e deixa continuar, a preço de PME e fora do ecossistema Microsoft. Ver "A Microsoft já faz algo parecido?". A lacuna "apps de desktop não cobertas" que tinha apontado antes ficou mais fraca.
- "Local" é um argumento de confiança e de RGPD; um preço de PME abaixo das plataformas enterprise.

**Contra**
- **Concorrência em todos os níveis:** código aberto (LocalAI, contextio, Presidio), extensões grátis, Cloudflare (grátis até 50 utilizadores), Microsoft (DLP + Edge for Business). Qualquer um pode descer para a PME.
- **Sem fosso técnico:** o padrão de proxy com TLS terminado é conhecido. O valor está na qualidade da deteção, na instalação sem fricção e na confiança — copiável.
- **Suporte e responsabilidade altos:** CA em cada PC, apps com pinning, monitorização de trabalhadores.
- **Economia por cliente:** a 5–7 $/utilizador/mês, uma PME de 10 pessoas dá 50–70 $/mês; 200 PME dariam 10–14 mil $/mês (aritmética ilustrativa). Exige vendas em volume, provavelmente através de empresas de informática/MSP (hipótese).
- **Tendência:** planos enterprise dos fornecedores de LLM com zero data retention e a integração da Microsoft reduzem o problema para quem já os usa.

**Critérios para parar (testes baratos)**
1. **Teste técnico no Windows:** se as apps de desktop de LLM e as ferramentas de linha de comandos recusarem a CA, a lacuna desaparece → parar, ou ficar só em browser (onde há mais concorrência).
2. **Licenciamento Microsoft nas PME portuguesas:** pesquisa feita (ver "A Microsoft já faz algo parecido?"); falta a licença real de cada PME entrevistada. Se já têm E5 ou o add-on Purview, o espaço encolhe.
3. **Falar com 8–10 PME ou empresas de informática em Portugal:** usam IA no trabalho; quem gere o IT; pagariam quanto por utilizador/mês. Sugestão de limiar: parar se menos de 3 em 10 pagariam.
4. **Consulta jurídica rápida** sobre monitorização de trabalhadores.

Limites desta avaliação: pesquisas de uma só ronda, só resultados dos EUA, fontes quase todas de fornecedores, licenciamento Microsoft com fontes contraditórias, e não verifiquei concorrentes portugueses para além das ferramentas gratuitas.

## Plano de validação (4 semanas, proposto)

**Hipótese a testar:** PME portuguesas (10–50 pessoas) usam IA no trabalho, receiam fugas de dados pessoais por causa do RGPD, não têm hoje solução e pagam 3–7 € por dispositivo/mês por uma proteção local, de instalação sem esforço. (O preço é uma hipótese.)

| Semana | O que fazer | Saída |
|---|---|---|
| 1 | 5 entrevistas com donos/responsáveis de PME e 2–3 empresas de informática (MSP) que lhes gerem o IT. Perguntar que licença Microsoft usam (pesquisa de 2026-10-20 em "A Microsoft já faz algo parecido?"). Marcar a consulta jurídica | Notas das conversas; resposta sobre a Microsoft |
| 2 | Mais 5 entrevistas. Teste técnico no Windows: proxy mínimo com CA própria contra as apps de LLM | Lista das apps que aceitam e das que recusam o proxy |
| 3 | Página de espera em português que descreve o problema (sem mecanismo nem nome) e pede pré-registo; enviar a quem foi entrevistado e a 1–2 canais | Número de inscrições e de pré-compromissos |
| 4 | Decisão go / no-go segundo os critérios abaixo, escrita aqui | Decisão |

**Datas:** não definidas (autor, 2026-10-20). Falta fixar a data de início; a decisão go / no-go fica 4 semanas depois.

**Critérios de go (sugestão):**
- pelo menos 3 em 10 empresas dizem que pagariam, com um valor próximo da hipótese;
- pelo menos 1 empresa de informática disposta a pilotar ou revender;
- o teste técnico cobre as apps que essas empresas realmente usam;
- a Microsoft não resolve já o caso nessas PME.

No-go se duas ou mais falharem.

**Perguntas das entrevistas:** ver "Guião de entrevista (PME)" mais abaixo.

**Proteger a ideia nas entrevistas** (preocupação do autor, 2026-10-20):
- Entrevistar o **problema**, não o produto. Não mencionar proxy, TLS, redação, restauro, preço nem nome.
- Enquadrar como "ferramentas de IA e RGPD na vossa empresa". Os blocos 1 a 4 do guião não revelam nada de proprietário.
- Nada por escrito: sem apresentações, demos nem esquemas.
- Entrevistar primeiro as **PME** (os clientes). Falar com empresas de informática depois e com menos detalhe: são quem melhor poderia construir algo parecido.
- Um acordo de confidencialidade costuma afastar as pessoas e não é preciso se a solução não for revelada. Só se justifica se algum dia mostrares algo concreto.
- Antes de falar com terceiros: registar o domínio, pedir a marca e guardar notas datadas das conversas.
- Limite: não há proteção total. O risco de cópia é real mas pequeno, comparado com o de passar meses a construir algo que ninguém paga; e a ideia já não é secreta (ver "Panorama de mercado").

**Antes de investir mais tempo (negócio):**
- Estrutura por decidir: fundador(es), empresa, financiamento.

## Guião de entrevista (PME)

**Objetivo:** perceber comportamento real, e não opiniões, sobre uso de IA, dados pessoais e decisão de compra, **sem revelar o produto**. Duração: 25–30 minutos. Com quem: dono, gerente ou responsável de TI/RGPD de uma PME (10–50 pessoas). A versão para empresas de informática está no fim.

### Antes
- Convite com este enquadramento: *"Estou a estudar como as pequenas empresas estão a lidar com ferramentas de IA e com o RGPD. São 25 minutos e não estou a vender nada."* Tem de ser verdade.
- Pedir autorização para tomar notas. Só gravar com autorização expressa. Não anotar nomes de funcionários nem dados pessoais.
- Ter a folha de notas aberta (modelo abaixo).
- **Nunca mencionar:** proxy, TLS, redação, restauro, preço, nome, nem o que estás a construir. Não perguntar "usaria?" ou "gostava de?" (hipóteses valem pouco).

### Abertura (2 min)
*"Obrigado pelo tempo. Não tenho nada para vender nem para mostrar; quero perceber como é na vossa empresa. Não há respostas certas. Posso tomar notas?"*

Depois: *"Fale-me da empresa: quantas pessoas são, que tipo de trabalho fazem e quem trata da informática?"*

### Bloco 1 — Uso de IA (5 min)
1. *"Os vossos funcionários usam ferramentas como o ChatGPT, o Claude ou o Copilot no trabalho? Como é que sabe?"*
   - Seguimentos: quais? No browser, na aplicação do computador ou no telemóvel? Contas da empresa ou pessoais? Pagas ou gratuitas?
2. *"Conte-me a última vez que viu alguém usar uma destas ferramentas no trabalho. Para quê?"*
   - Seguimentos: o que escreveu ou colou? Que tipo de documentos?

### Bloco 2 — Dados e risco (7 min)
3. *"Que tipo de informação já foi parar a essas ferramentas, até onde sabe?"* (clientes, contratos, faturas, saúde, recursos humanos)
4. *"Alguma vez houve um incidente, uma dúvida ou um receio? O que aconteceu?"*
5. *"O RGPD já entrou nesta conversa?"* (DPO, consultor, CNPD, um cliente que perguntou)

### Bloco 3 — O que fizeram (6 min)
6. *"O que fizeram até agora sobre isto?"* (nada, proibir, política escrita, formação, ferramenta)
   - **Se proibiram ou bloquearam:** *"O que aconteceu na prática? As pessoas deixaram de usar? Como contornaram?"* É a pergunta mais importante do guião: testa a hipótese de que o bloqueio faz as pessoas contornar.
   - **Se não fizeram nada:** *"Porquê? O que os impediu?"*
7. *"Quanto tempo ou dinheiro gastaram nisto?"*

### Bloco 4 — Decisão e orçamento (5 min)
8. *"Quando precisam de uma ferramenta nova de segurança ou conformidade, quem decide e quem paga?"*
9. *"Que ferramentas de segurança pagam hoje, e mais ou menos quanto por pessoa ou por mês?"* Perguntar também: *"Que subscrição de Microsoft 365 ou Google Workspace têm?"*
10. *"Quem instala e gere os computadores? Têm uma empresa externa de informática?"* (canal)
11. *"Quando foi a última vez que compraram algo assim? Como correu?"*

### Bloco 5 — O problema, em termos genéricos (3 min, só se a conversa o justificar)
12. *"Imagine que havia forma de as pessoas usarem estas ferramentas sem que dados pessoais de clientes saíssem da empresa. Isso era um problema que valesse a pena resolver?"*
13. *"O que o preocuparia numa solução assim? O que teria de ver para confiar?"*
14. *"Quanto seria razoável pagar por pessoa e por mês para resolver isto?"* **Perguntar sem ancorar.** Só se a pessoa não souber responder, oferecer faixas ("menos de 3 €", "3 a 7 €", "mais de 7 €").

Este bloco é um sinal fraco (é uma hipótese). Pesa menos do que as respostas dos blocos 1 a 4.

### Fecho (2 min)
- *"O que é que eu não perguntei e devia ter perguntado?"*
- *"Conhece alguém que lide com isto e queira falar comigo?"*
- *"Posso voltar a contactá-lo?"*
- Agradecer. Não prometer nada e não dizer o que vais construir.

### Durante a conversa
- Perguntar sobre o **passado** ("conte-me a última vez"), não sobre o futuro.
- Deixar o silêncio trabalhar. Seguir com *"e depois?"*, *"porquê?"*, *"pode dar um exemplo?"*.
- Não defender nem corrigir (nada de "isso não é seguro").
- Se perguntarem *"o que está a construir?"*: *"Ainda estou a perceber o problema; não há nada para mostrar."*
- Anotar o que fazem (comportamento, valores, nomes de ferramentas), e não o que dizem que fariam.

### Folha de notas (uma por entrevista)

| Campo | Notas |
|---|---|
| Empresa, setor, nº de pessoas | |
| Cargo de quem respondeu | |
| Usam IA? Quais, onde, contas | |
| Dados que já foram parar | |
| Incidente ou receio | |
| O que fizeram | |
| Se bloquearam: contornaram? | |
| Quem decide e paga | |
| Subscrição Microsoft / Google | |
| Gasto atual em segurança | |
| Valor que dizem pagar (bloco 5) | |
| Objeções e condições de confiança | |
| Quem instala e gere os PCs | |
| Sinal: forte / médio / fraco | |

### Como ler os sinais (para os critérios de go)
- **Sinal forte:** já gastou tempo ou dinheiro no problema; houve um incidente real; indica quem decide; aceita nova conversa.
- **Sinal fraco:** "seria interessante", "talvez", sem nenhuma ação passada.
- Para o critério "3 em 10 pagariam", só contam as pessoas que deram um valor concreto **e** têm orçamento ou poder de decisão. Respostas educadas não contam.
- Registar quantas contornaram um bloqueio (pergunta 6): é o número que mais pesa na hipótese "redigir em vez de bloquear".

### Registo de entrevistas

| Nº | Empresa | Quem | Segmento | Estado | Notas |
|---|---|---|---|---|---|
| 1 | Sumcap (sumcap.xyz) | CEO | Gestora de ativos DeFi e "venture builder" Web3; sede em Lisboa segundo uma ficha pública (a confirmar). **Atípica** face ao alvo PME geral | A marcar | Equipa provavelmente técnica, capaz de construir algo parecido: revelar o mínimo. Ouvir o que consideram sensível (estratégias, dados de investidores, chaves e segredos) e não só dados pessoais |
| 2 | eSegurança | CEO | Não identificada: a pesquisa não devolveu resultados; confirmar com a pessoa (site, o que vende) | A marcar | Se for uma empresa de segurança: possível canal e possível concorrente. Usar a variante para empresas de informática, com o mínimo de detalhe |
| 3 | DevScope (Porto) | Diretor de I&D | Empresa de software, dados e IA, parceira Microsoft (Data & AI, Modern Workplace), com um produto de extração de documentos com IA e clientes grandes. **Atípica** face ao alvo PME | A marcar | Capaz de construir algo parecido: revelar o mínimo. Perguntar o que os clientes pedem sobre IA e RGPD, o que a Microsoft oferece e onde falha, e como avaliam ferramentas de terceiros |

**Nota (2026-10-20):** as três primeiras entrevistas são de empresas técnicas ou de segurança. Dão informação sobre canal, concorrência e Microsoft, mas **não contam** para o critério das PME ("3 em 10 pagariam"). Ainda faltam as PME: no mínimo 8 entrevistas (de preferência fora do círculo próximo), o teste técnico e a consulta jurídica. Pistas para chegar a PME: contabilistas e consultores de RGPD (conhecem muitas), associações empresariais, LinkedIn. Cuidado com amigos e conhecidos: tendem a dizer que sim por simpatia.

Dos três contactos já disponíveis, dois são experientes em IT e um é leigo em informática. O **leigo é o mais parecido com o comprador** (dono ou gerente de PME): fazê-lo primeiro, sem jargão, e anotar as palavras que usa (servem para a página de espera). Só conta para o critério das PME se for dono ou decisor de uma PME com dados pessoais. Os dois experientes servem de especialistas e de canal: no fim, pedir-lhes apresentações a 2 ou 3 clientes PME cada um, sem revelar a solução.

**Empresas estrangeiras (ideia do autor, 2026-10-20):** servem para ver outras perspetivas, mas o RGPD só se aplica na UE. Marcar sempre o país no registo. Contam para o critério das PME só se forem PME sob o RGPD (UE) ou regime comparável (por exemplo o Brasil, com a LGPD); as restantes servem para comparar mercados. Limitar a 3 ou 4 para não dispersar antes de teres validado Portugal. Versões do guião (2026-10-20): **espanhol** em `guiao-es.md` e **francês** em `guiao-fr.md`, nesta pasta; o português, no README, é a versão de referência. Inglês e alemão não foram pedidos: se uma entrevista suíça ou belga for noutra língua (alemão, neerlandês), avisar. Manter as mesmas regras de sigilo.

Países previstos (autor, 2026-10-20): Suíça, França, Espanha e Bélgica. Enquadramento de memória, a confirmar com um jurista:

| País | Enquadramento | Conta como | Notas |
|---|---|---|---|
| Espanha | RGPD (autoridade AEPD) | Validação (PME sob RGPD) | Mais próxima de Portugal. O Presidio já tem reconhecedores espanhóis (ES_NIF, ES_NIE), logo existe concorrência gratuita |
| França | RGPD (autoridade CNIL) | Validação | Mercado grande, com língua e identificadores próprios (por exemplo o número de segurança social e o SIREN) |
| Bélgica | RGPD (autoridade belga) | Validação | Três línguas oficiais (francês, neerlandês, alemão): perguntar qual preferem. Número nacional de 11 dígitos |
| Suíça | Lei federal de proteção de dados revista (em vigor desde setembro de 2023), parecida com o RGPD mas não é o RGPD | Comparação, até um jurista confirmar | Sensibilidade alta à privacidade e preços mais altos. O RGPD aplica-se a quem trate dados de pessoas da UE |

Uma entrevista por país não permite comparar nada: tratar como exploratório. Prioridade: Espanha e França.

### Variante para empresas de informática (menos detalhe)
Só depois de teres falado com PME, e sem revelar a solução. Perguntar sobre:
- quantos clientes PME gerem e que subscrições Microsoft 365 ou Google lhes vendem;
- se os clientes já lhes perguntaram sobre IA e RGPD, e o que recomendam hoje;
- como cobram um serviço novo (por utilizador e por mês, por projeto);
- o que os faz adotar uma ferramenta nova e quanto tempo de instalação por cliente aceitam;
- como distribuem software (Intune, ferramenta de gestão remota).

Para quem constrói ou integra IA para clientes (por exemplo, a DevScope), acrescentar:
- como tratam dados pessoais nos projetos com LLMs (Azure OpenAI, outros) e o que os clientes lhes pedem;
- o que a Microsoft oferece para isto e onde falha na prática (bloquear em vez de limpar, licenças, apps de desktop);
- como avaliam ferramentas de terceiros (revisão de segurança, compras) e o que teria de existir para confiarem numa;
- se constroem, compram ou adaptam código aberto quando precisam de uma peça assim.

Não perguntar ainda por revenda ou piloto: o critério de go "uma empresa de informática disposta a pilotar" pede que se revele algo, por isso fica para depois de teres a marca pedida e de saberes a resposta das PME.

## Presets de deteção para Portugal (pesquisa de 2026-10-20)

O cliente define o que é sensível, mas o int443 traz presets prontos para dados pessoais portugueses. O que existe hoje nas ferramentas de referência:

| Tipo | Validação | Purview | Azure AI Language | Presidio |
|---|---|---|---|---|
| NIF | 9 dígitos; controlo mod 11 (pesos 9 a 2 nos 8 primeiros; resto < 2 dá 0, senão 11 − resto). Fontes concordam; as regras do primeiro dígito divergem entre sites → confirmar com a AT | Sim | Sim | Não encontrei recognizer PT |
| Cartão de Cidadão | Tem dígito de controlo; algoritmo não verificado. Uma pergunta no Microsoft Q&A diz que o Azure só deteta com o checksum correto | Sim | Sim | Não encontrei |
| Passaporte | — | Sim | Não encontrei | Não encontrei |
| Carta de condução | — | Sim | Não encontrei | Não encontrei |
| NISS | 11 dígitos; algoritmo não verificado | Não encontrei | Não encontrei | Não encontrei |
| IBAN PT | Norma IBAN (25 caracteres em PT, validação mod 97) — da norma geral, não da pesquisa; confirmar | Não verificado | Não verificado | Não verificado |
| Telefone, email, morada, código postal, matrícula | — | Não verificado | Não verificado | Não verificado |

Notas:

- **A validação do checksum é só aritmética:** confirma que o número tem forma válida, não que pertence a alguém. Como o dígito de controlo é um só, cerca de 1 em 10 sequências aleatórias de 9 dígitos passa como NIF. Sem contexto ("NIF", "contribuinte", "fatura"), um número de encomenda dispara falso positivo.
- **Nomes e moradas em texto livre não se apanham com checksum:** exigem um modelo de linguagem para português. É onde a qualidade da deteção mais pesa.
- **O Presidio não tem recognizer PT que eu tenha encontrado** (tem para Espanha); aceita recognizers próprios (`PatternRecognizer`). A lista oficial só ficou parcialmente visível, por isso confirmar na página dele.
- **Conclusão:** a hipótese de que as ferramentas dos EUA cobrem mal o PT estava demasiado forte. A lacuna provável está em NISS/IBAN (por confirmar), no texto livre em português, e em estar localmente no tráfego de LLM por um preço de PME.

## RGPD e enquadramento legal (a validar com um jurista)

Notas minhas, não aconselhamento jurídico:

- **Intercetar o tráfego dos trabalhadores é, em si, tratamento de dados e monitorização.** Na UE isto tende a pedir avaliação de impacto (DPIA), informação prévia aos trabalhadores e base legal clara. Em Portugal, ver também as regras do Código do Trabalho sobre meios de vigilância à distância e confidencialidade das comunicações (arts. 20.º a 22.º) e a posição da CNPD. É provavelmente o maior risco comercial: o cliente tem de poder instalar o int443 sem violar a lei.
- **Se o int443 é local e não envia nada, o fabricante não acede aos dados pessoais** — isto é um argumento de venda forte. Deixa de ser verdade se houver telemetria ou faturação que leve conteúdo.
- **Pseudonimização não é anonimização.** Se a empresa guarda o mapeamento para restaurar valores, os dados continuam a ser pessoais para ela. Redação irreversível é mais forte para o RGPD; a reversível dá melhores respostas. Escolha com consequências legais.
- **Acórdão do TJUE de 4/9/2025 (C-413/23 P, EDPS/SRB):** segundo resumos de escritórios de advogados (não li o texto do acórdão), dados pseudonimizados podem deixar de ser pessoais para um destinatário que não os consegue reidentificar razoavelmente, mas continuam a sê-lo para quem tem a chave; avalia-se caso a caso. Se se confirmar, o fornecedor de LLM receber só marcadores jogaria a favor do int. Mas texto livre com contexto ("o diretor da empresa X em Braga") pode reidentificar. A validar com jurista.
- **Não prometer "conformidade RGPD".** A deteção tem falsos negativos; o int443 reduz o risco, não o elimina. O texto comercial tem de refletir isso.

## Modelo de negócio (provisório)

Intenção do autor: **cobrar 1 dólar por cada dado sensível limpo.** O valor é negociável (autor, 2026-10-20): primeiro ver o que o mercado pratica. A mensalidade por dispositivo é a hipótese em cima da mesa; o modelo (por dado ou por dispositivo) ainda não está decidido. Se o código for aberto (ver "Distribuição e licença"), o que se cobra deixa de ser o código e passa a ser gestão, suporte e responsabilidade.

### Referências de preço (2026-10-20)

Fontes secundárias; preços de lista ou de contratos públicos. A maioria dos fornecedores só dá orçamento. Confirmar antes de usar.

| Produto | Preço indicado | Nota |
|---|---|---|
| Cloudflare Zero Trust | Grátis até 50 utilizadores (DLP limitado); 7 $/utilizador/mês; DLP completo sob orçamento | Estimativas de terceiros para o pacote com DLP: 10–25 $/utilizador/mês (não oficial) |
| Netskope | Endpoint443 DLP 28–42 $/utilizador/ano (contrato público da Florida, 2025) | Estimativas de terceiros: 12–18 $/utilizador/mês para a plataforma; mais de 200 000 $/ano para 1000 utilizadores com DLP completo |
| Zscaler | Módulos de proteção de dados 50–60 $/utilizador/ano (preço de lista); plataforma 200–240 $ | Escalões para menos de 100 utilizadores: 7 500–25 000 $/ano (Vendr, intervalo geral) |
| Nightfall | Orçamento anual por utilizador | Estimativas de terceiros: 5 000–15 000 $/ano para começar (não verificado) |
| Protecto | A partir de ~250 $/mês | Fonte secundária |
| APIs de deteção de PII (Amazon Comprehend, Sapling) | ~0,005–0,01 $ por 1000 palavras | Custo da deteção em bruto: frações de cêntimo |
| Presidio, LocalAI MITM proxy, contextio | Gratuitos | Código aberto |

**Leitura:** o mercado cobra **por utilizador**, tipicamente **2–20 $/mês**, e a deteção em si custa frações de cêntimo. 1 $ por dado fica ordens de grandeza acima do custo da deteção e muito acima do que uma PME paga por utilizador. Para PME, a âncora é o plano Cloudflare: grátis até 50 utilizadores, 7 $/utilizador/mês. Hipótese a validar: um preço por dispositivo/mês abaixo desse valor, com a contagem de dados limpos mostrada como prova de valor e não como unidade de cobrança.

### Mecânica de cobrança descrita pelo autor (2026-10-20)

- O utilizador **cria um token** para poder usar o int; para o criar **paga através de um link**. Só depois de pagar o int443 fica disponível.
- **Fidelização:** ao fim de 1 ano de uso, o preço por dado limpo desce de 1 $ para 0,80 $.

Pontos a resolver (propostas):

1. **Cobrança por dado e código aberto não se conciliam.** Se o motor corre localmente com licença MIT ou Apache, qualquer pessoa pode retirar a verificação do token e a contagem. Só é aplicável o que depende de algo no servidor (atualizações de regras e presets, consola de gestão, builds assinados, suporte). Opções: (a) **open core:** motor aberto, cobra-se gestão e atualizações por dispositivo/mês, aplicável porque o valor está no servidor; (b) **código fechado ou fonte disponível:** token validado online e cobrança por dado, assente em clientes honestos e em contrato B2B; (c) tudo aberto e cobra-se só suporte. **Com a decisão de código fechado, aplica-se a (b).**
2. **O que acontece quando o saldo acaba?** Num modelo pré-pago por dado, ficar sem saldo pode significar enviar dados sem filtragem ou bloquear o acesso à IA. Precisa de avisos antecipados por limiar, recarga automática, uma margem de segurança e uma regra por omissão, ligada ao aviso de falha já decidido.
3. **Contar sem expor.** Faturar por dado obriga o int443 a comunicar contagens: só metadados (tipo, quantidade, hora), nunca valores, e um registo local que o cliente possa auditar. É preciso definir "um dado" (ocorrência ou valor distinto por sessão) e o tratamento de falsos positivos.
4. **Previsibilidade.** A 1 $ por dado, os maiores utilizadores pagam mais e são os que mais provavelmente saem antes de completar 1 ano e receber o desconto. Alternativas: pacotes pré-pagos com preço por dado decrescente com o volume, limite mensal por dispositivo, ou subscrição base + dados incluídos + excedente. Fidelização por créditos de bónus em vez de desconto por antiguidade.
5. **O token é uma credencial:** guardar no Windows (Credential Manager), nunca em argumentos da linha de comandos, com revogação e rotação; decidir se é por dispositivo ou por conta.
6. **Pagamento e faturação:** link de pagamento de um fornecedor que trate do IVA na UE (um "merchant of record") e faturação certificada em Portugal. Validar com um contabilista.

## Por verificar

Afirmações deste README que **não estão confirmadas**. Não as tratar como factos.

| Afirmação | Origem e problema | Como verificar |
|---|---|---|
| Licenciamento Microsoft: o Business Premium não inclui Endpoint443 DLP nem a proteção inline; add-on Purview a cerca de 10 $/utilizador/mês; Edge "pay-as-you-go" | Tabelas da Microsoft e respostas da comunidade que se contradizem | Documentação oficial de licenciamento ou um revendedor Microsoft |
| Proteção de rede (Entra Global Secure Access) disponível desde setembro de 2026 | Fontes com datas diferentes | Página "What's new" da Microsoft |
| A Microsoft não faz redação nos prompts | "Não encontrei", não é prova | Testar num tenant; perguntar à DevScope |
| O Claude não consta da lista de sites de IA do Edge | Listas parciais | Documentação do Edge for Business |
| Licenças FSL e BSL (prazos, termos) | De memória | Texto das licenças e um jurista |
| Lei suíça de proteção de dados (2023), autoridades de França, Bélgica e Espanha, e o que "conta como" validação | De memória | Um jurista |
| Código do Trabalho, arts. 20.º a 22.º, sobre vigilância à distância | De memória | Um jurista |
| Acórdão C-413/23 P (EDPS/SRB) sobre pseudonimização | Só resumos de escritórios | Texto do acórdão e um jurista |
| Algoritmos do NISS e do Cartão de Cidadão; regras do primeiro dígito do NIF; validação do IBAN PT | Não verificados ou fontes de terceiros | Fontes oficiais (AT, Segurança Social, IRN) |
| Identificadores de França, Bélgica e Espanha (NIR, SIREN, número nacional belga, DNI/NIE) | De memória | Fontes oficiais de cada país |
| O Presidio não tem reconhecedor PT | A lista oficial ficou parcialmente visível | Página oficial e repositório |
| Preços (Nightfall, Protecto, Netskope, Zscaler, Cloudflare com DLP completo) | Blogs, contratos públicos e estimativas de terceiros | Pedir orçamentos |
| Estatísticas de "shadow AI" (LayerX 77%, Cyberhaven cerca de 40%) | Vêm de fornecedores que vendem a solução | Ler os relatórios originais |
| Concorrentes portugueses | Não pesquisados | Pesquisa em português |
| Sumcap: sede em Lisboa, empresa de 2021 | Uma ficha pública | Perguntar na entrevista |
| eSegurança | Não identificada | Confirmar com a pessoa |

## Perguntas em aberto

1. ~~Dispositivo ou caixa na rede?~~ **Respondida (2026-10-20):** serviço no próprio dispositivo (serviço Windows / daemon macOS). Sistema operativo inicial: **Windows**. Por decidir: como desviar o tráfego para o proxy (ver "Stack e ferramentas").
2. ~~Âmbito: só LLMs ou todo o tráfego?~~ **Respondida (2026-10-20):** só tráfego de LLMs, com lista de hosts permitidos.
3. ~~Quem é o utilizador?~~ **Respondida (2026-10-20):** empresas de todos os tipos, com RGPD. Perfil prioritário: **PME e pequenas empresas**.
4. ~~O que conta como sensível?~~ **Respondida (2026-10-20):** define-o o cliente (ex.: NIF), com o RGPD como guia. Proposta: presets de dados pessoais RGPD/PT + regras próprias do cliente.
5. Restaurar os valores na resposta, ou basta o marcador? Autor: "depende do RGPD" — ver "RGPD e enquadramento legal". Proposta revista (2026-10-20): com a promessa "enviar tudo e receber a resposta certa", o restauro em memória passa a ser a base; marcador sem restauro fica como modo mais estrito. A confirmar com um jurista.
6. **Preço:** o que é "um dado"? Uma ocorrência, um valor distinto, um prompt? (As APIs de chat reenviam o histórico inteiro em cada pedido, por isso o mesmo nome aparece repetido.)
7. **Faturação local:** como contar e cobrar sem que o int443 envie informação para fora? (Tensão com a promessa de privacidade.) Pista a validar: um registo só de metadados (tipo de dado, contagem, hora — nunca o valor) serviria para faturar e também como prova de responsabilidade perante o RGPD.
8. **Geografia:** só Portugal, ou também outros países da UE e o Brasil (LGPD)? As entrevistas a empresas estrangeiras ajudam a decidir. Hipótese: o Brasil partilha a língua (os modelos de linguagem em português servem) mas exige outros identificadores (CPF, CNPJ) e tem outro enquadramento legal; a UE partilha o RGPD mas não a língua.
9. **Âmbito do que é sensível:** só dados pessoais (RGPD), ou também informação confidencial da empresa e segredos técnicos (chaves de API, chaves privadas)? A conversa com a Sumcap pode ajudar a perceber.
10. **Modelo de cobrança:** por dado (1 $, 0,80 $ ao fim de 1 ano), por dispositivo e por mês, ou híbrido (pacotes pré-pagos, limite mensal)? Depende das entrevistas.
11. **Nome e marca:** "int" é genérico; verificar disponibilidade e registar a marca e o domínio.
12. **Roadmap do autor:** o autor referiu um roadmap (2026-10-20) que não chegou a ser colado. Quando existir, comparar com "Próximos passos" no resumo.
13. **Recursos, sucesso e datas:** sócios, tempo, dinheiro, competências, o que é sucesso e as datas da validação não estão definidos (autor, 2026-10-20). Sem isto, "vale a pena" fica sem critério fechado.

## Estado atual

O produto continua em validação de conceito e ainda não tem funcionalidades operacionais. Este repositório já contém o esqueleto da CLI e da biblioteca Rust descritos em [Estado do repositório](#estado-do-repositório). As notas de entrevistas e de mercado abaixo refletem o estado registado em 2026-10-20; confirma-as antes de as tratar como atuais.

Próximos passos: ver "Resumo" no topo e "Plano de validação".

Pendente: proteger esta pasta com um cofre cifrado (VHDX + BitLocker). Decidido, por fazer. O script de criação estava numa pasta temporária da sessão e pode perder-se; recria-se a pedido. Depois de correr o script, esta pasta passa a ser uma unidade, por isso as sessões do Claude Code têm de arrancar a partir dela.

## Estrutura da pasta e documentação de descoberta

| Caminho | O que é |
|---|---|
| `README.md` | Contexto do produto, notas de descoberta e guião de entrevista em português |
| `guiao-es.md` | Guião de entrevista em espanhol (tradução do que está no README) |
| `guiao-fr.md` | Guião de entrevista em francês (tradução do que está no README) |

## Stack e ferramentas

**Forma de execução (decidido):** serviço em cada dispositivo — serviço Windows ou daemon (launchd) no macOS. O repositório usa Rust; as bibliotecas para rede e inspeção ainda estão por escolher.

Desenho provisório, a validar:

- **Desviar o tráfego.** Opção simples: definir o proxy do sistema com uma lista de hosts de LLM (PAC) — apps que ignoram o proxy do sistema não são apanhadas. Opção completa: interceção transparente (no Windows um driver de filtragem de rede assinado; no macOS uma Network Extension, que exige entitlement da Apple e aprovação do utilizador). A segunda cobre mais mas custa muito mais a construir e a manter. Sugestão: começar pela simples.
- **CA própria, uma por dispositivo** (nunca partilhada entre clientes), com restrição de nomes (name constraints) aos domínios de LLM permitidos, para limitar o estrago se a chave for roubada. Chave privada protegida pelo SO (DPAPI/TPM no Windows, Keychain no macOS). Instalação em empresa por GPO/Intune (Windows) ou perfil MDM (macOS).
- **Cada runtime tem o seu conjunto de certificados:** Firefox, Node (`NODE_EXTRA_CA_CERTS`), Python (certifi / `REQUESTS_CA_BUNDLE`) e Java não leem o do sistema. Cada um precisa de configuração própria.
- **Privilégios mínimos:** o serviço gere o ciclo de vida; o proxy que vê o conteúdo corre com uma conta sem privilégios, não como LocalSystem/root.
- **Proteção contra desativação:** um utilizador pode desligar o proxy do sistema; em empresa isto resolve-se com políticas MDM.
- **Recuperação automática (decidido, 2026-10-20):** o serviço reinicia sozinho em caso de falha. Pontos de desenho:
  - O Windows reinicia serviços que terminam com erro (opções de recuperação do serviço), mas **não deteta um serviço vivo e bloqueado**. É preciso um verificador de saúde (heartbeat).
  - **O aviso ao utilizador não pode depender do proxy**, porque quando o proxy falha está morto. Um segundo processo pequeno, separado, vigia e mostra a notificação.
  - **Falhar aberto é uma propriedade do encaminhamento, não só do serviço.** Se o proxy do sistema aponta para uma porta morta, o browser mostra erro e o tráfego fica **bloqueado**, não é deixado passar. Para deixar passar, o encaminhamento tem de ter alternativa direta (por exemplo, um ficheiro PAC com `PROXY 127.0.0.1:porta; DIRECT`).
  - **Ciclo de falhas:** reinícios com espera crescente; ao fim de N falhas seguidas, escalar o aviso e aplicar o modo do cliente (bloquear ou deixar passar).
  - **O reinício apaga o mapeamento em memória.** Numa conversa a meio, as respostas podem aparecer com marcadores por restaurar. Pior: um mapeamento novo pode reutilizar `[PESSOA_1]` para outra pessoa e restaurar o nome errado. Proposta: marcadores derivados de uma chave local (não numerados por ordem de aparição), para nunca colidirem, e tratar marcadores desconhecidos como "não restaurar" com aviso.
- **Atualizações:** o serviço precisa de um mecanismo de atualização assinado (as regras de deteção também evoluem).
- **Sistema operativo inicial: Windows** (decidido, 2026-10-20). macOS fica para depois — duas plataformas duplicam o trabalho.
- **Instalação em PME:** muitas não têm MDM/GPO. O instalador (MSI assinado) tem de instalar a CA e configurar tudo com duplo clique, e também ser distribuível por Intune/GPO quando exista. O custo de suporte à instalação pesa muito no preço de uma PME.

## Decisões

| Data | Decisão | Porquê |
|---|---|---|
| 2026-10-20 | O int443 corre localmente | Alinhado com a promessa de não enviar conteúdo sensível para lado nenhum |
| 2026-10-20 | Preço inicial: 1 $ por dado sensível limpo (provisório) | Intenção do autor; por validar |
| 2026-10-20 | Cofre cifrado para esta pasta, a fazer mais tarde | Proteger a ideia de quem aceda ao disco |
| 2026-10-20 | Mercado-alvo: empresas de todos os tipos, no enquadramento RGPD | Decisão do autor |
| 2026-10-20 | Corre como serviço em cada dispositivo (Windows service / macOS daemon) | Decisão do autor; alinhado com "local" |
| 2026-10-20 | Windows primeiro; macOS depois | Decisão do autor |
| 2026-10-20 | Cliente-alvo: PME e pequenas empresas | Decisão do autor |
| 2026-10-20 | O que é sensível define-o o cliente, com o RGPD como guia | Decisão do autor |
| 2026-10-20 | Preço por definir; o valor é negociável; primeiro estudar o mercado | 1 $/dado fica muito acima das referências encontradas |
| 2026-10-20 | Interceta só tráfego de LLMs (lista de hosts) | Decisão do autor; limita a monitorização e a superfície legal |
| 2026-10-20 | O int443 é um negócio novo | Esclarecido pelo autor; o critério passa a ser validar a procura antes de construir |
| 2026-10-20 | Promessa central: o utilizador envia ao LLM o que quiser sem pensar no que é sensível | Esclarecido pelo autor; define os requisitos em "A promessa 'enviar tudo'" |
| 2026-10-20 | Se o proxy falhar: avisar o utilizador de imediato de que o tráfego pode passar sem filtragem | Decisão do autor; substitui a proposta de falhar fechado. Risco descrito em "A promessa 'enviar tudo'" |
| 2026-10-20 | O serviço reinicia sozinho em caso de falha | Decisão do autor; complementa o aviso imediato. Consequências em "Stack e ferramentas" |
| 2026-10-20 | Entrega: conta web para configurar + agente local (.exe que é serviço e CLI) | Descrição do autor; pontos de desenho em "Entrega" |
| 2026-10-20 | ~~Distribuição pelo GitHub com licença permissiva~~ Substituída: o autor prefere não ser open source; código fechado por agora | O token pago e a cobrança por dado só são aplicáveis com código fechado. A rever mais tarde (fonte disponível) |
| 2026-10-20 | Acesso por token, criado após pagamento por link; fidelização: ao fim de 1 ano o preço por dado desce de 1 $ para 0,80 $ | Descrição do autor; por validar, ver "Mecânica de cobrança" |
| 2026-10-20 | Validar a procura em 4 semanas antes de construir | Concorrência forte e nenhuma prova de que PME pagam |
| 2026-10-20 | Entrevistas a empresas estrangeiras: Suíça, França, Espanha, Bélgica; guiões em português, espanhol e francês | Decisão do autor; a Suíça conta como comparação, as outras como validação |

## Glossário

| Termo | Significado |
|---|---|
| Proxy MITM | Intermediário que termina o TLS do cliente e abre outro para o servidor, para poder ler e alterar o conteúdo |
| CA | Autoridade certificadora; aqui, uma própria instalada nos dispositivos para o proxy poder emitir certificados |
| Certificate pinning | Uma app só aceita certificados específicos e recusa o do proxy |
| PII | Informação pessoal identificável |
| TEE | Ambiente de execução isolado em hardware (inferência confidencial) |
| RGPD | Regulamento Geral sobre a Proteção de Dados (UE) |
| DPIA | Avaliação de impacto sobre a proteção de dados, exigida em tratamentos de risco elevado |
| MDM / GPO | Gestão central de dispositivos (Intune, etc.) / Group Policy do Windows; formas de instalar a CA em todos os PCs da empresa |
| LLM | Modelo de linguagem de grande dimensão, o que está por trás do ChatGPT, do Claude e do Gemini |
| MSP | Empresa de informática que gere os computadores e a segurança de outras empresas |
| QUIC / HTTP3 | Protocolo que os browsers usam para ligações rápidas e que um proxy tradicional não apanha |
| PAC | Ficheiro que diz ao browser por onde enviar cada ligação (pelo proxy ou diretamente) |
| SSE | Forma de a resposta de um LLM chegar aos pedaços, em tempo real |
| EDR | Software de segurança que vigia comportamentos suspeitos no computador |
| MSI | Formato de instalador do Windows que as empresas distribuem por Intune ou GPO |
| SmartScreen | Aviso do Windows ao abrir programas pouco conhecidos |
| Falhar aberto / fechado | Deixar passar o tráfego quando a proteção falha / bloquear o tráfego |

## Ligações

- [LocalAI — MITM proxy](https://localai.io/docs/features/mitm-proxy/)
- [contextio](https://github.com/larsderidder/contextio)
- [Microsoft Purview — Sensitive information type entity definitions](https://learn.microsoft.com/en-us/purview/sit-sensitive-information-type-entity-definitions)
- [Azure AI Language — PII entity categories](https://learn.microsoft.com/en-us/azure/ai-services/language-service/personally-identifiable-information/concepts/entity-categories-list)
- [Presidio — PII entities supported](https://presidio.dataprivacystack.org/supported_entities/)
- [Microsoft Purview — gerir dados em outras apps de IA](https://learn.microsoft.com/en-us/purview/ai-other-apps)
- [Microsoft Purview — bloquear dados sensíveis para apps de IA (shadow AI, passo 3)](https://learn.microsoft.com/en-us/purview/deploymentmodels/depmod-data-leak-shadow-ai-step3)
- [Microsoft Security Blog — controlos Purview para browser e rede](https://techcommunity.microsoft.com/blog/microsoft-security-blog/building-layered-protection-new-microsoft-purview-data-security-controls-for-the/4395071)
- [Microsoft Purview — descrição do serviço e licenças](https://learn.microsoft.com/en-us/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-tenantlevel-services-licensing-guidance/microsoft-purview-service-description)
- [Purview network DLP no Entra Global Secure Access](https://maxime.hiez.ca/en/blog/2026-10-13-purview-network-layer-dlp-global-secure-access-ga)
- [Caviard — anonimizar prompts no browser](https://www.caviard.ai/anonymize-chatgpt-prompts)
- [Shadow AI statistics 2026 (dope.security)](https://dope.security/post/shadow-ai-statistics)
- [Validar NIF português (contribuinte.pt)](https://contribuinte.pt/validar-nif)
- [Cloudflare Zero Trust pricing 2026](https://zerotrustcost.com/cloudflare-zero-trust-pricing)
- [Netskope pricing guide 2026](https://underdefense.com/blog/netskope-pricing-guide/)
- [Zscaler pricing 2026](https://www.venn.com/learn/zscaler-pricing/)
- [Nightfall AI — RecaTools](https://recatools.com/ai-directory/nightfall-ai/)
- [PII redaction pricing: Comprehend, Azure, Google, Presidio](https://www.beri.net/article/pii-redaction-pricing-comprehend-azure-google-presidio-cost-per-million-characters)
- [Sidley — TJUE, conceito de dado pessoal (EDPS/SRB)](https://datamatters.sidley.com/2025/10/10/eu-court-of-justice-issues-landmark-judgment-on-concept-of-personal-data/)
- [Confident Security — lançamento](https://www.businesswire.com/news/home/20250717403863/en/Confident-Security-Launches-The-First-Enterprise-Grade-System-for-Provably-Private-AI-Interactions)
