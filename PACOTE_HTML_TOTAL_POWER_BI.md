# Pacote HTML Total Para O Power BI

Este arquivo organiza uma versao do painel usando **HTML Content** para quase tudo:

- filtro HTML de regionais
- cabecalho HTML premium
- tabela HTML de cursos/turmas

## Antes de tudo

Limite importante e intencional:

- para ter **filtro clicavel real** no HTML Content, o visual precisa usar **Granularity**
- por isso, o painel em HTML funciona melhor com **mais de um visual HTML Content**
- um visual unico com um measure gigante pode responder aos filtros do relatorio, mas **nao faz cross-filter por clique sozinho**

Em outras palavras:

- `sim`, da para fazer filtro HTML e tabela HTML funcionando
- `nao`, isso nao deve ficar em um unico visual se voce quiser clique real no filtro

## Referencias oficiais

- Data Roles: https://html-content.com/docs/data-roles
- Interactivity: https://html-content.com/docs/interactivity
- Stylesheet: https://html-content.com/docs/properties-stylesheet

Resumo do que a documentacao diz:

- com **single measure**, o visual tem um valor unico e fica limitado para interatividade
- com **Granularity**, o HTML Content ganha contexto por linha e pode fazer **cross-filter**

## Estrutura recomendada da pagina

Para a pagina `Visao Geral`, use 3 visuais `HTML Content`:

1. `Filtro HTML Regional`
2. `Resumo HTML Premium`
3. `Tabela HTML`

Layout sugerido:

- topo esquerdo: `Resumo HTML Premium`
- topo direito: `Filtro HTML Regional`
- abaixo, largura inteira: `Tabela HTML`

## Medidas que ja devem existir

Estas ja foram criadas no relatorio:

- `Turmas Em Alerta`
- `Total N3`
- `Total N2`
- `Total N1`
- `Total N0`
- `Unidade Mais Critica`
- `Qtd Alerta Unidade Mais Critica`

## 1. Filtro HTML De Regional

### 1.1 Measure do item do filtro

Crie esta medida:

```DAX
HTML Filtro Regional Item =
VAR Regional =
    SELECTEDVALUE ( 'ARED2027 1'[Núcleo Regional], "Sem regional" )

VAR AlertasRegional =
    [Turmas Em Alerta]

RETURN
"<div style='
    display:flex;
    justify-content:space-between;
    align-items:center;
    gap:12px;
    padding:12px 14px;
    border-radius:14px;
    background:rgba(255,255,255,0.72);
    border:1px solid rgba(223,210,190,0.7);
    color:#43382e;
'>
    <div style='font-size:14px; line-height:1.35; font-weight:600;'>
        " & Regional & "
    </div>
    <div style='
        min-width:28px;
        text-align:center;
        font-size:12px;
        line-height:1;
        padding:7px 8px;
        border-radius:999px;
        background:#f4ebdf;
        color:#6d5e4e;
    '>
        " & FORMAT ( AlertasRegional, "#,0" ) & "
    </div>
</div>"
```

### 1.2 Measure do CSS do filtro

Crie esta medida:

```DAX
CSS Filtro Regional =
"#htmlContent{
    display:grid;
    gap:10px;
}

.htmlViewerEntry{
    cursor:pointer;
    transition:transform .18s ease, opacity .18s ease;
}

.htmlViewerEntry:hover{
    transform:translateX(2px);
}

.htmlViewerEntry.unselected{
    opacity:.38 !important;
}
"
```

### 1.3 Como configurar o visual do filtro

Adicione um novo `HTML Content` e configure assim:

- `Values`: `HTML Filtro Regional Item`
- `Granularity`: `Núcleo Regional`
- `Stylesheet`: usar a medida `CSS Filtro Regional` na formatacao condicional do stylesheet, se quiser

Propriedades importantes:

- habilitar interatividade/cross-filter do visual
- desligar ou reduzir a transparencia padrao de itens nao selecionados, se quiser usar o CSS

O resultado esperado:

- cada regional aparece como um item HTML
- ao clicar em uma regional, os outros visuais HTML da pagina devem responder

## 2. Resumo HTML Premium

Crie esta medida:

```DAX
HTML Visao Geral Premium =
VAR RegionalSelecionada =
    SELECTEDVALUE ( 'ARED2027 1'[Núcleo Regional], "Todas as Regionais" )

VAR TurmasAlerta =
    [Turmas Em Alerta]

VAR N3 =
    [Total N3]

VAR N2 =
    [Total N2]

VAR N1 =
    [Total N1]

VAR N0 =
    [Total N0]

VAR UnidadeCritica =
    [Unidade Mais Critica]

VAR QtdUnidadeCritica =
    [Qtd Alerta Unidade Mais Critica]

VAR TextoUnidade =
    IF (
        NOT ISBLANK ( UnidadeCritica ),
        UnidadeCritica & " exige atenção imediata",
        "Nenhuma unidade crítica identificada"
    )

VAR TextoResumo =
    IF (
        NOT ISBLANK ( UnidadeCritica ),
        "Dentro da regional analisada, esta unidade concentra <strong>" &
            FORMAT ( QtdUnidadeCritica, "#,0" ) &
            " turmas em alerta</strong>, considerando os níveis N3 e N2.",
        "No contexto atual do filtro, não houve unidade com concentração relevante de alertas."
    )

RETURN
"
<div style='
    font-family:Segoe UI, Arial, sans-serif;
    color:#261f19;
    background:
        radial-gradient(circle at top left, rgba(173,118,0,0.08), transparent 24%),
        radial-gradient(circle at top right, rgba(159,56,43,0.08), transparent 22%),
        linear-gradient(180deg, #f8f4ed 0%, #f4efe7 100%);
    border:1px solid #dfd2be;
    border-radius:28px;
    padding:28px;
    box-shadow:0 18px 40px rgba(76,51,28,0.08);
'>

    <div style='display:flex; justify-content:space-between; align-items:flex-start; gap:18px; flex-wrap:wrap;'>
        <div style='flex:1; min-width:320px;'>
            <div style='font-size:12px; letter-spacing:2px; text-transform:uppercase; color:#7b6a58;'>
                Monitoramento de evasão
            </div>

            <div style='font-size:52px; line-height:0.98; font-weight:800; letter-spacing:-1.6px; margin-top:10px; color:#261f19;'>
                Painel ARED 2027.1
            </div>

            <div style='font-size:18px; margin-top:16px; color:#584c3f;'>
                Regional selecionada: <strong style=""color:#312720;"">" & RegionalSelecionada & "</strong>
            </div>

            <div style='max-width:760px; margin-top:16px; font-size:16px; line-height:1.6; color:#564a3e;'>
                A leitura executiva desta abertura prioriza turmas em alerta, distribuição de risco e a unidade que exige acompanhamento imediato. A ideia é levar o olhar para ação, não apenas para volume.
            </div>
        </div>

        <div style='width:260px; background:rgba(255,247,237,0.95); border:1px solid #e6d7c4; border-radius:24px; padding:20px 22px;'>
            <div style='font-size:12px; letter-spacing:1.5px; text-transform:uppercase; color:#8d7c67;'>
                Turmas em alerta
            </div>

            <div style='margin-top:10px; font-size:64px; line-height:0.95; font-weight:800; color:#9f382b;'>
                " & FORMAT ( TurmasAlerta, "#,0" ) & "
            </div>

            <div style='margin-top:10px; font-size:14px; color:#625648; line-height:1.5;'>
                Considera níveis N3 e N2 no contexto atual do filtro.
            </div>
        </div>
    </div>

    <div style='display:flex; gap:14px; flex-wrap:wrap; margin-top:28px;'>
        <div style='flex:1; min-width:170px; background:rgba(255,255,255,0.78); border:1px solid rgba(223,210,190,0.9); border-radius:16px; padding:16px 18px 18px; box-shadow: inset 0 4px 0 #9f382b;'>
            <div style='font-size:12px; letter-spacing:1.3px; text-transform:uppercase; color:#7c6d5e;'>
                N3 Crítico
            </div>
            <div style='margin-top:10px; font-size:42px; line-height:1; font-weight:760; color:#9f382b;'>
                " & FORMAT ( N3, "#,0" ) & "
            </div>
            <div style='margin-top:10px; font-size:15px; color:#5d5146;'>
                Alto nível de evasão
            </div>
        </div>

        <div style='flex:1; min-width:170px; background:rgba(255,255,255,0.78); border:1px solid rgba(223,210,190,0.9); border-radius:16px; padding:16px 18px 18px; box-shadow: inset 0 4px 0 #ad7600;'>
            <div style='font-size:12px; letter-spacing:1.3px; text-transform:uppercase; color:#7c6d5e;'>
                N2 Atenção
            </div>
            <div style='margin-top:10px; font-size:42px; line-height:1; font-weight:760; color:#ad7600;'>
                " & FORMAT ( N2, "#,0" ) & "
            </div>
            <div style='margin-top:10px; font-size:15px; color:#5d5146;'>
                Médio nível de evasão
            </div>
        </div>

        <div style='flex:1; min-width:170px; background:rgba(255,255,255,0.78); border:1px solid rgba(223,210,190,0.9); border-radius:16px; padding:16px 18px 18px; box-shadow: inset 0 4px 0 #4f7257;'>
            <div style='font-size:12px; letter-spacing:1.3px; text-transform:uppercase; color:#7c6d5e;'>
                N1 Estável
            </div>
            <div style='margin-top:10px; font-size:42px; line-height:1; font-weight:760; color:#4f7257;'>
                " & FORMAT ( N1, "#,0" ) & "
            </div>
            <div style='margin-top:10px; font-size:15px; color:#5d5146;'>
                Baixo nível de evasão
            </div>
        </div>

        <div style='flex:1; min-width:170px; background:rgba(255,255,255,0.78); border:1px solid rgba(223,210,190,0.9); border-radius:16px; padding:16px 18px 18px; box-shadow: inset 0 4px 0 #6e7884;'>
            <div style='font-size:12px; letter-spacing:1.3px; text-transform:uppercase; color:#7c6d5e;'>
                N0 Sem evasão
            </div>
            <div style='margin-top:10px; font-size:42px; line-height:1; font-weight:760; color:#6e7884;'>
                " & FORMAT ( N0, "#,0" ) & "
            </div>
            <div style='margin-top:10px; font-size:15px; color:#5d5146;'>
                Situação preservada
            </div>
        </div>
    </div>

    <div style='display:flex; gap:20px; flex-wrap:wrap; margin-top:24px;'>
        <div style='flex:1.3; min-width:340px; background:rgba(255,251,245,0.92); border:1px solid #dfd2be; border-radius:22px; padding:24px;'>
            <div style='font-size:12px; letter-spacing:1.4px; text-transform:uppercase; color:#7b6a58;'>
                Prioridade de acompanhamento
            </div>

            <div style='margin-top:10px; font-size:38px; line-height:1.08; font-weight:800; letter-spacing:-0.8px; color:#7f2f23;'>
                " & TextoUnidade & "
            </div>

            <div style='margin-top:14px; font-size:16px; line-height:1.65; color:#55483c;'>
                " & TextoResumo & "
            </div>

            <div style='margin-top:14px; font-size:16px; line-height:1.65; color:#55483c;'>
                Para uma rotina de gestão, este bloco ajuda a abrir a conversa pela urgência antes da leitura detalhada por curso e turma.
            </div>
        </div>

        <div style='flex:0.9; min-width:260px; background:rgba(255,251,245,0.92); border:1px solid #dfd2be; border-radius:22px; padding:22px;'>
            <div style='padding-bottom:14px; border-bottom:1px solid #eadfce;'>
                <div style='font-size:12px; letter-spacing:1.2px; text-transform:uppercase; color:#7b6a58;'>
                    Unidade mais crítica
                </div>
                <div style='margin-top:8px; font-size:18px; line-height:1.35; font-weight:700; color:#2d241d;'>
                    " & IF ( NOT ISBLANK ( UnidadeCritica ), UnidadeCritica, "Sem destaque no filtro atual" ) & "
                </div>
            </div>

            <div style='padding-top:14px; padding-bottom:14px; border-bottom:1px solid #eadfce;'>
                <div style='font-size:12px; letter-spacing:1.2px; text-transform:uppercase; color:#7b6a58;'>
                    Alertas na unidade
                </div>
                <div style='margin-top:8px; font-size:34px; line-height:1; font-weight:760; color:#2d241d;'>
                    " & FORMAT ( QtdUnidadeCritica, "#,0" ) & "
                </div>
            </div>

            <div style='padding-top:14px;'>
                <div style='font-size:12px; letter-spacing:1.2px; text-transform:uppercase; color:#7b6a58;'>
                    Leitura recomendada
                </div>
                <div style='margin-top:8px; font-size:14px; line-height:1.55; color:#5f5347;'>
                    Use a tabela HTML abaixo para descer ao nível de curso e turma dentro da regional selecionada.
                </div>
            </div>
        </div>
    </div>
</div>
"
```

Como configurar:

- `Values`: `HTML Visao Geral Premium`
- sem `Granularity`

## 3. Tabela HTML Completa

Crie esta medida:

```DAX
HTML Tabela Cursos =
VAR Base =
    ADDCOLUMNS (
        FILTER (
            'ARED2027 1',
            NOT ISBLANK ( 'ARED2027 1'[Habilitação/Curso] )
        ),
        "__NivelSort",
            SWITCH (
                'ARED2027 1'[Nível],
                "N3", 4,
                "N2", 3,
                "N1", 2,
                "N0", 1,
                0
            ),
        "__StatusTexto",
            SWITCH (
                'ARED2027 1'[Nível],
                "N3", "Crítico",
                "N2", "Atenção",
                "N1", "Estável",
                "N0", "Sem evasão",
                "Sem classificação"
            ),
        "__StatusCor",
            SWITCH (
                'ARED2027 1'[Nível],
                "N3", "#9f382b",
                "N2", "#ad7600",
                "N1", "#4f7257",
                "N0", "#6e7884",
                "#6e7884"
            ),
        "__StatusBg",
            SWITCH (
                'ARED2027 1'[Nível],
                "N3", "rgba(159,56,43,0.10)",
                "N2", "rgba(173,118,0,0.12)",
                "N1", "rgba(79,114,87,0.10)",
                "N0", "rgba(110,120,132,0.12)",
                "rgba(110,120,132,0.12)"
            )
    )

VAR BaseOrdenada =
    TOPN (
        COUNTROWS ( Base ),
        Base,
        [__NivelSort], DESC,
        'ARED2027 1'[Unidades do CEETEPS], ASC,
        'ARED2027 1'[Habilitação/Curso], ASC,
        'ARED2027 1'[Turma], ASC
    )

VAR Linhas =
    CONCATENATEX (
        BaseOrdenada,
        "<tr>
            <td>" & COALESCE ( 'ARED2027 1'[Núcleo Regional], "-" ) & "</td>
            <td>" & COALESCE ( 'ARED2027 1'[Unidades do CEETEPS], "-" ) & "</td>
            <td>" & COALESCE ( 'ARED2027 1'[Habilitação/Curso], "-" ) & "</td>
            <td>" & COALESCE ( 'ARED2027 1'[Turma], "-" ) & "</td>
            <td>" & FORMAT ( COALESCE ( 'ARED2027 1'[Vagas], 0 ), "#,0" ) & "</td>
            <td>" & FORMAT ( COALESCE ( 'ARED2027 1'[Inscritos], 0 ), "#,0" ) & "</td>
            <td>" & FORMAT ( COALESCE ( 'ARED2027 1'[Demanda], 0 ), "0.00" ) & "</td>
            <td>
                <span style='
                    display:inline-flex;
                    align-items:center;
                    gap:8px;
                    border-radius:999px;
                    padding:7px 10px;
                    font-size:12px;
                    font-weight:600;
                    background:" & [__StatusBg] & ";
                    color:" & [__StatusCor] & ";
                '>
                    <span style='
                        width:8px;
                        height:8px;
                        border-radius:50%;
                        background:" & [__StatusCor] & ";
                        display:inline-block;
                    '></span>
                    " & [__StatusTexto] & "
                </span>
            </td>
        </tr>",
        ""
    )

VAR QuantidadeLinhas =
    COUNTROWS ( BaseOrdenada )

RETURN
"<div style='
    font-family:Segoe UI, Arial, sans-serif;
    background:rgba(255,251,245,0.90);
    border:1px solid #dfd2be;
    border-radius:24px;
    box-shadow:0 18px 40px rgba(76,51,28,0.08);
    overflow:hidden;
'>
    <div style='
        display:flex;
        justify-content:space-between;
        align-items:center;
        gap:16px;
        padding:20px 22px;
        border-bottom:1px solid #ebdecc;
        background:rgba(255,250,243,0.88);
    '>
        <div>
            <div style='font-size:12px; letter-spacing:1.4px; text-transform:uppercase; color:#7b6a58;'>
                Leitura tabular
            </div>
            <div style='font-size:20px; font-weight:700; color:#2b221c; margin-top:6px;'>
                Turmas e cursos em acompanhamento
            </div>
        </div>
        <div style='
            border-radius:999px;
            padding:8px 12px;
            background:#f4ebdf;
            color:#6d5e4e;
            font-size:13px;
            white-space:nowrap;
        '>
            " & FORMAT ( QuantidadeLinhas, "#,0" ) & " linhas no filtro atual
        </div>
    </div>

    <div style='overflow:auto;'>
        <table style='width:100%; border-collapse:collapse;'>
            <thead>
                <tr>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Núcleo Regional</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Unidade</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Curso</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Turma</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Vagas</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Inscritos</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Demanda</th>
                    <th style='text-align:left; padding:14px 16px; border-bottom:1px solid #efe5d8; font-size:12px; text-transform:uppercase; letter-spacing:1.25px; color:#7d6c59; background:rgba(251,247,241,0.96);'>Diagnóstico</th>
                </tr>
            </thead>
            <tbody>" & Linhas & "</tbody>
        </table>
    </div>
</div>"
```

Como configurar:

- `Values`: `HTML Tabela Cursos`
- sem `Granularity`

## 4. Ordem De Montagem Na Pagina

1. remova os cards nativos de teste
2. mantenha somente HTML Content
3. crie 3 visuais `HTML Content`

Visual 1:

- nome mental: `Filtro Regional`
- `Values`: `HTML Filtro Regional Item`
- `Granularity`: `Núcleo Regional`

Visual 2:

- nome mental: `Resumo Premium`
- `Values`: `HTML Visao Geral Premium`

Visual 3:

- nome mental: `Tabela HTML`
- `Values`: `HTML Tabela Cursos`

## 5. O Que Esperar

Esse pacote deve entregar:

- pagina quase toda em HTML
- filtro lateral em HTML clicavel
- topo premium em HTML
- tabela em HTML reagindo ao filtro

## 6. O Que Pode Dar Errado

Mesmo que funcione, estes riscos sao reais:

- a tabela HTML pode ficar pesada com muitas linhas
- o filtro HTML pode precisar de ajuste de interatividade no visual
- se o visual estiver em modo `lite`, alguns comportamentos podem ficar mais restritos

Se travar:

- primeiro reduza a altura da tabela
- depois teste com uma regional menor
- se ainda pesar, troque `COUNTROWS ( Base )` por um `TOPN` com limite

## 7. Se Quiser Continuar Comigo

Voce pode me chamar assim:

- `vamos montar o filtro html`
- `vamos colar a tabela html`
- `deu erro no HTML Tabela Cursos`
- `quero ajustar o visual premium`

E eu continuo a partir deste pacote.
