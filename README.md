[README_SACRE.md](https://github.com/user-attachments/files/34101043/README_SACRE.md)
# Calculadora SACRE — Financiamento Imobiliário

Simulador web, em um único arquivo `index.html`, do **Sistema de Amortização Crescente (SACRE)**, o sistema usado em financiamentos imobiliários da Caixa Econômica Federal. A ferramenta mostra como a **TR (Taxa Referencial)** corrige o saldo devedor todo mês e permite comparar o efeito de **amortizações extraordinárias**: reduzir o prazo, reduzir a parcela ou fazer várias amortizações ao longo do contrato.

🔗 **Acesse online:** <https://lbemm3.github.io/calculadora-sacre/>

> ⚠️ **Simulação educacional.** Os resultados são estimativas baseadas em premissas simplificadas (veja [Premissas e limitações](#premissas-e-limitações)) e não substituem o contrato, o extrato do banco nem a orientação de um especialista financeiro.

---

## Sumário

- [Funcionalidades](#funcionalidades)
- [Como usar](#como-usar)
- [Abas do simulador](#abas-do-simulador)
- [Entradas e valores padrão](#entradas-e-valores-padrão)
- [Como o cálculo funciona](#como-o-cálculo-funciona)
- [Exemplo com os valores padrão](#exemplo-com-os-valores-padrão)
- [Exportação em CSV e PDF](#exportação-em-csv-e-pdf)
- [Tecnologias](#tecnologias)
- [Estrutura do código](#estrutura-do-código)
- [Premissas e limitações](#premissas-e-limitações)
- [Executando localmente](#executando-localmente)
- [Publicação via GitHub Pages](#publicação-via-github-pages)
- [Autoria e licença](#autoria-e-licença)

---

## Funcionalidades

- **Simulação completa do SACRE** mês a mês, com amortização, juros, taxa de administração, seguro, correção pela TR, prestação total e saldo devedor, além da linha de totais.
- **Recálculo anual da amortização** (a cada 12 meses), destacado na tabela.
- **Redução de prazo**: encontra o novo prazo após uma amortização extraordinária, mantendo a prestação próxima da atual.
- **Redução da parcela**: mantém o prazo e calcula a nova prestação.
- **Comparação lado a lado** das duas modalidades com uma única digitação dos dados.
- **Amortizações múltiplas**: qualquer número de eventos (mês + valor), por exemplo o 13º salário todo dezembro.
- **Sensibilidade à TR**: três cenários (otimista, histórico e pessimista) sobre a economia obtida.
- **Gráfico** da evolução do saldo devedor, com e sem amortização.
- **Cartões-resumo**: novo saldo, novo prazo, 1ª prestação, economia total e percentual economizado.
- **Exportação em CSV e PDF** das tabelas.
- **Funciona no celular**: layout responsivo, sem instalação.

## Como usar

1. Acesse a página publicada (ou abra o `index.html` no navegador, com internet).
2. Escolha uma das cinco abas.
3. Preencha os campos. Os valores padrão já formam um exemplo funcional.
4. Clique no botão de cálculo da aba.
5. Analise os cartões, o gráfico e a tabela, e exporte em CSV ou PDF se quiser.

> Os resultados só são atualizados ao clicar no botão de cálculo. Depois de alterar um campo, clique no botão novamente.

## Abas do simulador

| Aba | Para que serve | Resultado |
|---|---|---|
| 📊 **Simulador SACRE** | Simula um financiamento novo, do início ao fim. | Tabela completa de prestações, com totais e destaque (↺) nos meses de recálculo. |
| ⚡ **Redução de Prazo** | Simula uma amortização extraordinária mantendo a prestação. | Novo prazo, meses reduzidos, 1ª prestação, economia, gráfico, cenários de TR e tabela. |
| 💸 **Redução da Parcela** | Simula uma amortização extraordinária mantendo o prazo. | Nova 1ª prestação, redução da parcela, economia, gráfico, cenários de TR e tabela. |
| ⚖️ **Comparação** | Compara as duas modalidades lado a lado. | Duas colunas com saldo, prazo, prestação, juros, TR e economia, mais gráfico. |
| 🔄 **Múltiplas Amortizações** | Simula vários aportes extras em meses diferentes. | Total amortizado, economia, redução de juros + TR, comparação e tabela destacando os meses com extra (💰). |

## Entradas e valores padrão

| Campo | Unidade | Aba 1 | Abas 2 a 5 |
|---|---|---|---|
| Valor do financiamento / Saldo devedor atual | R$ | 300.000 | 191.000 |
| Prazo (total ou restante) | meses | 360 | 97 |
| Taxa de juros | % ao ano | 9,6 | 6,785 |
| TR (Taxa Referencial) | % ao mês | 0,1255747 | 0,1255747 |
| Taxa de administração | R$/mês | 25 | 25 |
| Seguro do financiamento | R$/mês | 50 | 50 |
| Prestação atual (abas 2 e 4) | R$ | — | 3.155 |
| Amortização extraordinária (abas 2, 3 e 4) | R$ | — | 50.000 |
| Eventos de amortização (aba 5) | mês + R$ | — | mês 12 e mês 24, R$ 10.000 cada |

O valor padrão da TR (0,1255747% ao mês) é a média histórica indicada na própria ferramenta para o período 2004–2025.

## Como o cálculo funciona

A função central é `simularSACRE`. Notação: `S` = saldo, `i` = taxa mensal, `τ` = TR mensal, `n` = meses restantes, `A` = amortização, `T` = taxa de administração e `G` = seguro.

**Taxa mensal.** A taxa anual informada é dividida por 12: `i = (taxa anual) / 100 / 12`.

**Amortização.** No mês 1, `A = S₀ / prazo`. A cada 12 meses (meses 13, 25, 37…), a amortização é recalculada:

```
A = S / n          (saldo atual ÷ meses restantes)
```

**Cada mês:**

```
juros     = S · i
prestação = A + juros + T + G
S         = S − A
correção  = S · τ
S         = S · (1 + τ)
n         = n − 1
```

Ou seja, a TR incide sobre o saldo **depois** da amortização do mês. É ela que faz o saldo crescer e "anula" parte da amortização, razão pela qual o sistema não é um SAC puro.

**Amortização extraordinária (abas 2 a 5).**

- *Redução de prazo:* o novo saldo é `S − extra`. O simulador percorre `n` do prazo atual até 1 e escolhe o primeiro `n` tal que `S'/n + S'·i ≥ prestação atual`. Se nenhum `n` satisfizer, mantém o prazo original.
- *Redução da parcela:* o prazo é mantido e a nova 1ª prestação é `S'/prazo + S'·i + T + G`.
- *Múltiplas amortizações:* no mês de cada evento, o valor é abatido do saldo **antes** do cálculo dos juros e a amortização é recalculada como `saldo ÷ meses restantes`. Se o saldo chegar a zero (≤ R$ 0,01), a simulação termina.

**Economia total:**

```
economia = total pago sem amortização − (total pago com amortização + valor amortizado)
```

**Cenários de TR** (abas 2 e 3), refazendo a simulação com:

| Cenário | TR mensal |
|---|---|
| Otimista | 0% |
| Histórica | 0,1256% |
| Pessimista | 0,5% |

Para cada um, a tabela mostra economia total, redução de juros, redução de correção TR e o novo prazo (aba 2) ou a nova 1ª parcela (aba 3).

## Exemplo com os valores padrão

**Aba 1 — Simulador SACRE** (R$ 300.000, 360 meses, 9,6% a.a., TR 0,1255747% a.m., adm. R$ 25, seguro R$ 50):

| Mês | Amortização | Juros | Correção TR | Prestação | Saldo |
|---|---|---|---|---|---|
| 1 | R$ 833,33 | R$ 2.400,00 | R$ 375,68 | R$ 3.308,33 | R$ 299.542,34 |
| 12 | R$ 833,33 | R$ 2.359,47 | R$ 369,32 | R$ 3.267,81 | R$ 294.470,04 |
| 13 ↺ | R$ 846,18 | R$ 2.355,76 | R$ 368,72 | R$ 3.276,94 | R$ 293.992,58 |

Totais ao longo dos 360 meses: prestações somam **R$ 914.462,76**, dos quais **R$ 508.267,85** são juros e **R$ 79.305,81** são correção pela TR. No mês 13 a amortização foi recalculada de R$ 833,33 para R$ 846,18.

**Abas 2 e 3** (saldo R$ 191.000, 97 meses, 6,785% a.a., prestação atual R$ 3.155, extra de R$ 50.000):

| Modalidade | Resultado |
|---|---|
| ⚡ Redução de prazo | Prazo cai de 97 para **59 meses** (−38), 1ª prestação de R$ 3.262,07 e economia de **R$ 40.491,13**. |
| 💸 Redução da parcela | Prazo mantido em 97 meses, 1ª prestação cai **R$ 798,17** (para R$ 2.325,85) e economia de **R$ 17.627,66**. |

**Aba 5:** com dois aportes de R$ 10.000 (meses 12 e 24) nos mesmos dados, a economia estimada é de **R$ 5.798,34**.

*Esses valores foram obtidos executando a própria função `simularSACRE` com os campos padrão.*

## Exportação em CSV e PDF

| Aba | Arquivo gerado |
|---|---|
| Simulador SACRE | `SACRE_Simulacao_Completa` (.csv / .pdf), com todos os meses |
| Redução de Prazo | `SACRE_Reducao_Prazo` |
| Redução da Parcela | `SACRE_Reducao_Parcela` |
| Múltiplas Amortizações | `SACRE_Amortizacoes_Multiplas` |

- **CSV:** separador `;`, codificação UTF-8 com BOM, colunas *Mês; Amortização; Juros; Correção TR; Prestação; Saldo*. Os números usam **ponto decimal**; ao abrir no Excel em português, pode ser necessário importar o arquivo definindo o separador decimal.
- **PDF:** página em paisagem, com título, linhas de resumo e tabela gerada com jsPDF + AutoTable.
- Nas abas 2, 3 e 5 a tabela exportada traz os **12 primeiros meses**, o **último mês** e os **meses com amortização extra**, e não o contrato inteiro.
- A taxa de administração e o seguro estão embutidos na coluna *Prestação* nas exportações.
- A aba **Comparação** não possui exportação.

## Tecnologias

Aplicação 100% no navegador, sem build nem servidor. As bibliotecas são carregadas por CDN:

| Biblioteca | Versão | Uso |
|---|---|---|
| React + ReactDOM | 18.2.0 | Interface e estado |
| Babel Standalone | 7.23.2 | Compila o JSX no navegador |
| Tailwind CSS (CDN) | — | Classes utilitárias |
| Chart.js | 4.4.0 | Gráfico de saldo devedor |
| jsPDF | 2.5.1 | Geração de PDF |
| jsPDF-AutoTable | 3.5.28 | Tabelas no PDF |
| Google Fonts | — | Syne, DM Sans e DM Mono |

## Estrutura do código

Tudo está em `index.html`, em seções comentadas:

| Trecho | Conteúdo |
|---|---|
| **UTILS** | `fmtBRL`, formatação em reais (pt-BR). |
| **NÚCLEO** | `simularSACRE`, o motor de cálculo. |
| **EXPORTAÇÕES** | `exportarCSV`, `exportarPDF` e o componente `BotoesExport`. |
| **CAMPO** | Componente de entrada numérica com rótulo e dica. |
| **GRÁFICO** | `GraficoSaldo`, gráfico de linhas do Chart.js. |
| **CENÁRIOS TR** | `CenariosTR`, tabela de sensibilidade à TR. |
| **RESULTADO** | `ResultadoAmortizacao`, usado nas abas de redução de prazo e de parcela. |
| **ABAS 1 a 5** | `CalculadoraSACRE`, `ReducaoPrazo`, `ReducaoParcela`, `Comparacao`, `AmortizacoesMultiplas`. |
| **APP PRINCIPAL** | `App`, com cabeçalho, avisos e navegação por abas. |

## Premissas e limitações

- **Precisa de internet.** As bibliotecas vêm de CDN. Sem conexão, a página não carrega.
- **Desempenho de carregamento.** O JSX é compilado no navegador pelo Babel Standalone, o que deixa a primeira abertura mais lenta que a de um site com build.
- **Modelo simplificado.** Juros mensais = taxa anual ÷ 12 (não é taxa efetiva equivalente), TR constante durante todo o contrato, taxa de administração e seguro fixos (na prática o seguro costuma variar com o saldo e a idade).
- **Resíduo no último mês.** Como a amortização é recalculada só a cada 12 meses e a TR continua corrigindo o saldo, o saldo da última parcela pode ficar levemente acima de zero (no exemplo padrão da aba 1, cerca de R$ 110,90 no mês 360). A linha de totais exibe saldo zero.
- **Comparação da prestação no prazo reduzido.** A busca do novo prazo compara a prestação atual informada com a 1ª prestação **sem** taxa de administração e seguro (amortização + juros). A 1ª prestação exibida inclui essas tarifas.
- **Eventos sem validação.** Na aba de múltiplas amortizações, meses fora do intervalo do prazo ou valores maiores que o saldo não geram aviso.
- **Sem persistência.** Os dados não são salvos: ao recarregar a página, voltam aos valores padrão.
- **Fora do escopo.** Não considera IOF, seguros específicos, portabilidade, FGTS nem outras regras contratuais.
- **Não é o simulador oficial** da Caixa Econômica Federal.

## Executando localmente

Não há instalação nem build. Baixe o arquivo e abra no navegador (com internet):

```bash
git clone https://github.com/lbemm3/calculadora-sacre.git
cd calculadora-sacre
open index.html        # macOS
xdg-open index.html    # Linux
start index.html       # Windows
```

Ou use um servidor local:

```bash
python3 -m http.server 8000
# acesse http://localhost:8000/
```

## Publicação via GitHub Pages

O arquivo já se chama `index.html`, então a página abre direto na raiz do endereço.

1. No repositório, vá em **Settings → Pages**.
2. Em *Build and deployment*, selecione **Deploy from a branch**.
3. Escolha a branch principal (`main`) e a pasta `/ (root)`, e clique em **Save**.
4. Após alguns minutos, o simulador estará em <https://lbemm3.github.io/calculadora-sacre/>.

## Autoria e licença

Desenvolvido por **Laerte Bemm** (Departamento de Matemática, Universidade Estadual de Maringá – UEM).

Licença: defina a licença do repositório (por exemplo, MIT) e inclua o arquivo `LICENSE`.
