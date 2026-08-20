# Estoque Inteligente — protótipo de TCC

App Android (Kotlin + Jetpack Compose) para leitura de NF-e via QR Code/código de barras
e gestão de estoque. Este projeto é um protótipo funcional, organizado para ser explicado
e expandido facilmente na defesa do TCC.

## Como abrir

1. Abra a pasta `EstoqueInteligente/` no Android Studio (versão recente, Hedgehog+).
2. Deixe o Gradle sincronizar (ele vai baixar Compose, Room, CameraX e ML Kit).
3. Rode em um dispositivo/emulador com Android 7.0 (API 24) ou superior.

## Ordem de desenvolvimento (como pedido)

O código já está pronto para as 3 etapas, mas o **fluxo padrão do app hoje usa a Etapa 1**
(arquivo XML local), porque ainda não existe API autorizada configurada:

1. **Etapa 1 — XML local.** Na tela inicial, toque em "Ler NF-e (arquivo XML local)" e
   selecione o arquivo em `exemplos-teste/nfe-exemplo-FICTICIO-para-teste.xml` (dados
   fictícios, só para validar o parser) ou um XML de NF-e real que você já tenha.
2. **Etapa 2 — Câmera.** O botão "Ler NF-e (câmera / QR Code)" já abre a câmera e
   reconhece a chave de 44 dígitos (`scanner/LeitorCodigoScreen.kt`), mas como ainda
   não há API configurada, ele mostra a mensagem explicando que falta a integração —
   isso é intencional, para não inventar dados.
3. **Etapa 3 — API autorizada.** Veja a seção "Onde plugar a API" abaixo.

## Estrutura de pastas (mapeada 1:1 com o que foi pedido)

```
data/model/     -> Produto, Fornecedor, Entrada, Lote, NotaFiscal, ProdutoNFe (estruturas de banco)
data/db/        -> Room: AppDatabase + DAOs (banco de dados)
nfe/            -> NFeXmlParser (parser de XML) e NFeService (serviço de NF-e)
scanner/        -> LeitorCodigoScreen (leitor de código de barras / QR, CameraX + ML Kit)
estoque/        -> EstoqueManager (gerenciamento de estoque: soma quantidades, evita duplicar NF-e)
notificacoes/   -> AlertaService + AlertaWorker (estoque baixo, validade — Etapa 13)
ui/             -> HomeScreen, ConferenciaScreen, EstoqueScreen, HistoricoScreen
MainActivity.kt -> navegação e "cola" entre as camadas acima
```

## Onde plugar a API autorizada de NF-e (Etapa 3)

Tudo está concentrado em **um único arquivo**: `nfe/NFeService.kt`.

1. Abra a classe `NFeServiceApi` nesse arquivo.
2. Preencha `BASE_URL` com a URL da API/provedor autorizado.
3. Preencha `API_KEY` — **não deixe a chave real escrita direto no código**. Em vez
   disso: crie uma entrada em `local.properties` (que não vai pro Git), leia via
   `BuildConfig` no `app/build.gradle.kts`, ou use um cofre de segredos.
4. Implemente a chamada HTTP dentro de `obterNotaFiscalPorChave(...)` (sugestão:
   adicionar Retrofit/OkHttp nas dependências) e converta a resposta para o objeto
   `NotaFiscal`.
5. No fim de `NFeService.kt`, dentro de `NFeServiceProvider.get(...)`, troque
   `NFeServiceLocalXml()` por `NFeServiceApi(context)`.

Nenhuma outra tela ou classe precisa mudar — todas conversam com `NFeService`
pela interface, nunca com a implementação concreta.

**Importante:** o app nunca acessa a SEFAZ diretamente. `NFeServiceApi` só deve
apontar para um provedor/serviço já autorizado/homologado para isso.

## Banco de dados

SQLite via Room (`data/db/AppDatabase.kt`), com as 4 tabelas pedidas: `produtos`,
`fornecedores`, `entradas` e `lotes` (esta última já criada e pronta, mas ainda não
usada ativamente no fluxo principal — é o ponto de partida da Etapa 14).

## Funcionalidades futuras (Etapa 14) já com "gancho" no código

- Controle de lote/validade: tabela `lotes` e campos `controlaLote`/`controlaValidade`
  em `Produto` já existem.
- Alertas de validade: método `verificarValidadesEAvisar()` em `AlertaService.kt`
  já tem a estrutura pronta, com um `TODO` indicando exatamente onde ligar a lógica
  assim que o cadastro de lotes for usado no fluxo principal.
- FEFO, previsão de consumo, sugestão de compras, comparação de fornecedores,
  desperdício/excesso: ainda não implementados — a separação em camadas (
  `estoque/`, `data/db/`) foi pensada para que essas regras entrem como novas
  classes sem precisar reescrever o que já existe.

## Sobre os dados de teste

O arquivo em `exemplos-teste/` é **fictício**, feito só para você testar o parser
sem esperar uma NF-e real. Ele fica fora de `app/src` de propósito (não é
empacotado dentro do app) para não ser confundido com dado de produção. Assim que
tiver um XML real, use-o no lugar — o parser já foi escrito seguindo o layout
oficial de tags da NF-e (`infNFe`, `ide`, `emit`, `det`/`prod`).
