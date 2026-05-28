# Guia Do Dashboard Power BI ARED 2027.1

Este arquivo sera nossa referencia de trabalho para montar o painel no Power BI com apoio do visual `HTML Content`.

Objetivo principal do painel:

- identificar quais unidades e cursos precisam de acao
- permitir filtro facil por `Nucleo Regional`
- mostrar apenas as unidades da regional selecionada
- destacar `turmas em alerta`
- manter uma tabela classica para usuarios que preferem leitura tabular
- criar uma pagina com mapa das regionais de Sao Paulo

## 1. Estrutura recomendada do relatorio

### Pagina 1 - Visao Geral

Objetivo:

- abrir o painel com leitura executiva
- mostrar rapidamente quantas turmas estao em alerta
- destacar os niveis `N3`, `N2`, `N1` e `N0`
- deixar o filtro de regional bem visivel

Visuais recomendados:

- slicer de `Nucleo Regional`
- visual `HTML Content` para cabecalho executivo
- tabela resumida ou matriz de unidades prioritarias
- grafico de barras por `Diagnostico`

### Pagina 2 - Regional e Unidades

Objetivo:

- quando a pessoa filtrar uma regional, enxergar so as unidades daquela regional
- comparar rapidamente a situacao entre unidades

Visuais recomendados:

- slicer de `Nucleo Regional`
- tabela de unidades
- cards ou HTML com resumo da regional

### Pagina 3 - Cursos Em Acao

Objetivo:

- manter uma visao mais classica
- permitir navegação detalhada por curso e turma

Colunas recomendadas:

- `Nucleo Regional`
- `Unidades do CEETEPS`
- `Habilitacao/Curso`
- `Turma`
- `Vagas`
- `Inscritos`
- `Demanda`
- `Diagnostico`

### Pagina 4 - Mapa Das Regionais

Objetivo:

- mostrar o estado de Sao Paulo dividido por regional
- permitir clique na regional para filtrar o restante da pagina ou do relatorio

Base de apoio identificada na planilha:

- aba `unidades`
- campos uteis:
  - `municipio`
  - `regiao_administrativa`
  - `regiao_governo`
  - `nucleo_regional`
  - `nome_unidade`
  - `codigo_unidade`

## 2. Logica de negocio definida

### Hierarquia de diagnostico

- `N3` = `Alto Nivel de Evasao`
- `N2` = `Medio Nivel de Evasao`
- `N1` = `Baixo Nivel de Evasao`
- `N0` = `Sem Evasao`

### O que entra em alerta

Para este painel, vamos considerar alerta como:

- `N3`
- `N2`

### Medida principal do topo

- `Turmas em alerta`

## 3. Layout visual recomendado

Direcao visual escolhida:

- institucional e elegante

Principios:

- fundo claro
- pouco ruido visual
- destaque forte apenas para o que exige acao
- filtro principal no topo
- HTML usado como camada premium, nao como substituto de todos os visuais

Paleta recomendada:

- `N3`: vermelho sobrio
- `N2`: dourado/amarelo queimado
- `N1`: verde suave
- `N0`: cinza azulado
- neutros: bege claro, off-white, cinza quente

## 4. Ordem de construcao recomendada

Montar nesta ordem evita confusao:

1. verificar se a tabela principal esta carregada no modelo
2. criar a pagina `Visao Geral`
3. adicionar slicer de `Nucleo Regional`
4. adicionar uma tabela simples com as colunas principais
5. criar as medidas DAX basicas
6. instalar ou adicionar o visual `HTML Content`
7. criar o cabecalho HTML
8. montar a pagina de unidades
9. montar a pagina de cursos
10. montar o mapa das regionais

## 5. Checklist tecnico antes de comecar

Confirmar no Power BI:

- a tabela principal existe com o nome `ARED2027 1`
- a coluna `Nucleo Regional` esta disponivel
- a coluna `Diagnostico` esta disponivel
- a coluna `Nivel` esta disponivel
- a coluna `Unidades do CEETEPS` esta disponivel
- o visual `HTML Content` esta instalado

## 6. Passo a passo detalhado da Pagina 1

### Passo 1 - Criar a pagina

- abra o arquivo `Ared.pbix`
- crie uma nova pagina
- renomeie para `Visao Geral`

### Passo 2 - Inserir o slicer principal

- adicione um visual `Slicer`
- arraste a coluna `Nucleo Regional`
- posicione no topo da pagina
- se quiser, use o modo dropdown para ficar mais limpo

Configuracao recomendada:

- titulo: `Regional`
- selecao unica: desligada
- busca: ligada

### Passo 3 - Inserir uma tabela provisoria

Essa tabela serve para garantir que os filtros estao funcionando antes de embelezar o painel.

- adicione um visual `Tabela`
- arraste estas colunas:
  - `Nucleo Regional`
  - `Unidades do CEETEPS`
  - `Habilitacao/Curso`
  - `Turma`
  - `Vagas`
  - `Inscritos`
  - `Demanda`
  - `Diagnostico`

Teste esperado:

- ao selecionar uma regional no slicer, a tabela deve mostrar apenas linhas daquela regional

### Passo 4 - Criar as medidas basicas

Crie estas medidas na tabela `ARED2027 1`.

#### Total de turmas

```DAX
Total Turmas =
COUNTROWS ( 'ARED2027 1' )
```

#### Turmas em alerta

```DAX
Turmas Em Alerta =
CALCULATE (
    COUNTROWS ( 'ARED2027 1' ),
    'ARED2027 1'[Nível] IN { "N3", "N2" }
)
```

#### Total N3

```DAX
Total N3 =
CALCULATE (
    COUNTROWS ( 'ARED2027 1' ),
    'ARED2027 1'[Nível] = "N3"
)
```

#### Total N2

```DAX
Total N2 =
CALCULATE (
    COUNTROWS ( 'ARED2027 1' ),
    'ARED2027 1'[Nível] = "N2"
)
```

#### Total N1

```DAX
Total N1 =
CALCULATE (
    COUNTROWS ( 'ARED2027 1' ),
    'ARED2027 1'[Nível] = "N1"
)
```

#### Total N0

```DAX
Total N0 =
CALCULATE (
    COUNTROWS ( 'ARED2027 1' ),
    'ARED2027 1'[Nível] = "N0"
)
```

### Passo 5 - Adicionar o HTML Content

- insira o visual `HTML Content`
- deixe uma area larga no topo para ele
- depois vamos alimentar esse visual com um measure HTML

### Passo 6 - Criar o measure do cabecalho HTML

```DAX
HTML_Header_ARED =
VAR RegionalSelecionada =
    SELECTEDVALUE ( 'ARED2027 1'[Núcleo Regional], "Todas as Regionais" )

VAR TurmasAlerta =
    CALCULATE (
        COUNTROWS ( 'ARED2027 1' ),
        'ARED2027 1'[Nível] IN { "N3", "N2" }
    )

VAR N3 =
    CALCULATE (
        COUNTROWS ( 'ARED2027 1' ),
        'ARED2027 1'[Nível] = "N3"
    )

VAR N2 =
    CALCULATE (
        COUNTROWS ( 'ARED2027 1' ),
        'ARED2027 1'[Nível] = "N2"
    )

VAR N1 =
    CALCULATE (
        COUNTROWS ( 'ARED2027 1' ),
        'ARED2027 1'[Nível] = "N1"
    )

VAR N0 =
    CALCULATE (
        COUNTROWS ( 'ARED2027 1' ),
        'ARED2027 1'[Nível] = "N0"
    )

RETURN
"
<div style='
    font-family: Segoe UI, Arial, sans-serif;
    background: linear-gradient(135deg, #f7f4ef 0%, #f3efe8 100%);
    border: 1px solid #d9d1c5;
    border-radius: 20px;
    padding: 24px 28px;
    color: #2f2a24;
'>
    <div style='display:flex; justify-content:space-between; align-items:flex-start; gap:24px; flex-wrap:wrap;'>
        <div>
            <div style='font-size:13px; text-transform:uppercase; letter-spacing:1.6px; color:#8a7963;'>
                Monitoramento de evasao
            </div>
            <div style='font-size:34px; font-weight:700; margin-top:6px; color:#1f1a15;'>
                Painel ARED 2027.1
            </div>
            <div style='font-size:16px; margin-top:8px; color:#5c5145;'>
                Regional selecionada: <strong>" & RegionalSelecionada & "</strong>
            </div>
        </div>

        <div style='min-width:220px; background:#fffaf2; border:1px solid #e4dacb; border-radius:16px; padding:16px 18px;'>
            <div style='font-size:12px; text-transform:uppercase; letter-spacing:1.3px; color:#8d7c67;'>
                Turmas em alerta
            </div>
            <div style='font-size:42px; font-weight:800; line-height:1; color:#8f2d1f; margin-top:8px;'>
                " & FORMAT ( TurmasAlerta, "#,0" ) & "
            </div>
            <div style='font-size:14px; margin-top:8px; color:#625648;'>
                Considera niveis N3 e N2
            </div>
        </div>
    </div>

    <div style='display:flex; gap:14px; flex-wrap:wrap; margin-top:22px;'>
        <div style='flex:1; min-width:140px; background:#fff; border-radius:14px; padding:14px 16px; border-top:4px solid #a6382b;'>
            <div style='font-size:12px; color:#7b6d5d; text-transform:uppercase;'>N3 Critico</div>
            <div style='font-size:28px; font-weight:700; color:#a6382b; margin-top:6px;'>" & FORMAT ( N3, "#,0" ) & "</div>
            <div style='font-size:13px; color:#5f5449;'>Alto nivel de evasao</div>
        </div>

        <div style='flex:1; min-width:140px; background:#fff; border-radius:14px; padding:14px 16px; border-top:4px solid #c28a17;'>
            <div style='font-size:12px; color:#7b6d5d; text-transform:uppercase;'>N2 Atencao</div>
            <div style='font-size:28px; font-weight:700; color:#9a6a00; margin-top:6px;'>" & FORMAT ( N2, "#,0" ) & "</div>
            <div style='font-size:13px; color:#5f5449;'>Medio nivel de evasao</div>
        </div>

        <div style='flex:1; min-width:140px; background:#fff; border-radius:14px; padding:14px 16px; border-top:4px solid #4f7a5a;'>
            <div style='font-size:12px; color:#7b6d5d; text-transform:uppercase;'>N1 Estavel</div>
            <div style='font-size:28px; font-weight:700; color:#40654a; margin-top:6px;'>" & FORMAT ( N1, "#,0" ) & "</div>
            <div style='font-size:13px; color:#5f5449;'>Baixo nivel de evasao</div>
        </div>

        <div style='flex:1; min-width:140px; background:#fff; border-radius:14px; padding:14px 16px; border-top:4px solid #7d8791;'>
            <div style='font-size:12px; color:#7b6d5d; text-transform:uppercase;'>N0 Sem evasao</div>
            <div style='font-size:28px; font-weight:700; color:#65707b; margin-top:6px;'>" & FORMAT ( N0, "#,0" ) & "</div>
            <div style='font-size:13px; color:#5f5449;'>Situacao preservada</div>
        </div>
    </div>
</div>
"
```

### Passo 7 - Alimentar o visual HTML

- arraste o measure `HTML_Header_ARED` para o campo do visual
- teste o slicer de regional
- confirme que o numero de alertas muda junto com o filtro

### Passo 8 - Adicionar um grafico simples de apoio

Sugestao:

- grafico de barras
- eixo: `Diagnostico`
- valores: `Contagem de linhas`

Objetivo:

- deixar visivel a distribuicao dos niveis logo na primeira pagina

## 7. Passo a passo detalhado da Pagina 2

### Nome da pagina

- `Regional e Unidades`

### Visuais recomendados

- slicer de `Nucleo Regional`
- tabela com unidades
- opcionalmente um card de total de unidades em alerta

Tabela recomendada:

- `Nucleo Regional`
- `Unidades do CEETEPS`
- `Turma`
- `Diagnostico`
- `Inscritos`
- `Vagas`
- `Demanda`

## 8. Passo a passo detalhado da Pagina 3

### Nome da pagina

- `Cursos Em Acao`

### Visual principal

- tabela ou matriz

Colunas:

- `Nucleo Regional`
- `Unidades do CEETEPS`
- `Habilitacao/Curso`
- `Turma`
- `Vagas`
- `Inscritos`
- `Demanda`
- `Diagnostico`

Melhorias futuras:

- formatacao condicional por diagnostico
- coluna amigavel de status:
  - `Critico`
  - `Atencao`
  - `Estavel`
  - `Sem evasao`

## 9. Passo a passo detalhado da Pagina 4

### Nome da pagina

- `Mapa Das Regionais`

### Objetivo

- clicar numa regional e depois enxergar apenas unidades daquela regional

### O que provavelmente sera necessario

- importar a aba `unidades`
- relacionar `nucleo_regional` com `Nucleo Regional`
- definir se o mapa sera por forma geografica ou por pontos

### Solucao recomendada

Se houver como obter um shape map das regionais:

- usar mapa por poligonos das regionais

Se nao houver shape map no momento:

- usar visual por municipios ou pontos de unidade como etapa intermediaria

## 10. Perguntas em aberto para fases futuras

Esses pontos podem ser resolvidos depois:

- qual medida usar para destacar `unidade X exige atencao imediata`
- se vamos criar ranking por unidade, curso ou turma
- se o mapa usara shape file customizado
- se vale construir uma tabela dimensional de regionais

## 11. Proxima acao recomendada

Se for continuar com este guia, a proxima acao pratica e:

1. abrir o Power BI
2. criar a pagina `Visao Geral`
3. inserir o slicer de `Nucleo Regional`
4. inserir a tabela provisoria
5. criar as medidas DAX basicas
6. me chamar de volta para validarmos o measure `HTML_Header_ARED`

## 12. Como usar este arquivo comigo depois

Voce pode me pedir de varias formas:

- `abra o guia do dashboard`
- `vamos continuar do passo 4`
- `me relembra a pagina 2`
- `quero montar agora o mapa`
- `vamos revisar as medidas do guia`

Eu consigo continuar a partir daqui usando este arquivo como referencia.
