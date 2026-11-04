# 01 — Políticas de senha

## Como o Blazon trata o tema (resumo)

Uma política de senha é um conjunto de configurações que define como uma senha é **validada** (na troca) ou **gerada** (por exemplo, para contas de aplicação com gerenciamento de senha ligado). Pode haver várias políticas, mas **uma precisa ser a padrão**, usada quando nenhuma outra for indicada no processo.

Cada política tem cinco seções: complexidade, dicionário de palavras, histórico, expiração e notificações.

## Desenho proposto para a Acme Fictícia

Os valores abaixo são **ilustrativos**. Em um projeto real, saem da política corporativa de segurança.

| Seção | POL-PADRAO (colaboradores) | POL-PRIVILEGIADA (contas administrativas) | POL-APLICACAO (contas de aplicação) |
|---|---|---|---|
| Tamanho mínimo / máximo | 12 / 64 | 16 / 64 | 24 / 64 |
| Minúsculas, maiúsculas e números | Exigidos | Exigidos | Exigidos |
| Caracteres especiais | Não exigidos | Exigidos | Exigidos |
| Sem sequências (abc, 123) | Sim | Sim | Sim |
| Sem dados pessoais | Sim | Sim | Não se aplica |
| Dicionário | DIC-GERAL | DIC-PRIVILEGIADO (inclui o DIC-GERAL) | Não necessário (senha gerada) |
| Histórico | 5 últimas | 12 últimas | 3 últimas |
| Expiração | 30 dias | 30 dias | 90 dias |
| Notificações de expiração | Todas as opções | Todas as opções | Todas as opções |

Os mesmos valores estão em [`data/password-policies.csv`](../data/password-policies.csv).

## Registro de decisões

**D1 — Expiração periódica.** A ISO 27001 (controle 5.17) exige um processo formal de gestão das informações de autenticação, mas não define um intervalo de troca. Já o NIST SP 800-63B recomenda não impor trocas periódicas sem indício de comprometimento. *Decisão:* a Acme Fictícia adota 30 dias por política interna, em linha com seu apetite de risco. O trade-off (rotação frequente pode induzir senhas previsíveis) fica registrado para reavaliação, com a mitigação de tamanho mínimo de 12 caracteres e dicionário de palavras proibidas.

**D2 — Tamanho e dicionário antes de complexidade.** Na política padrão, não exigi caractere especial, porque a exigência tende a gerar padrões previsíveis como `Senha@123`. Em vez disso, combinei tamanho mínimo de 12 com um dicionário de palavras proibidas (senhas comuns, termos da empresa, padrões sazonais e de teclado), que bloqueia justamente as escolhas que a regra de complexidade deixaria passar. O dicionário é revisado a cada 6 meses.

**D3 — Contas de aplicação.** A senha é gerada pela própria política e não digitada por pessoa, então o tamanho pode ser alto. A expiração de 90 dias, maior que a dos usuários, só faz sentido se existir rotação controlada (cofre de senhas ou processo equivalente).

**D4 — Política padrão obrigatória.** Defini a POL-PADRAO como padrão. Pela documentação, a política padrão não pode ser removida enquanto estiver nessa condição, então qualquer troca de padrão exige antes designar outra.

## Dicionários de palavras proibidas

Pelo que a documentação de políticas descreve, cada política seleciona um dicionário já cadastrado. Por isso o DIC-PRIVILEGIADO é um superconjunto do DIC-GERAL, em vez de dois dicionários somados.

| Categoria | Exemplos fictícios | Dicionário |
|---|---|---|
| Senhas comuns | senha, mudar123, brasil, qwerty, euteamo | DIC-GERAL |
| Nomes próprios e times | Lucas, Pedro, flamengo, corinthians | DIC-GERAL |
| Termos da empresa | acme, acmeficticia, nomes dos sistemas (erp, crm) | DIC-GERAL |
| Padrões sazonais | meses do ano, verao, inverno, anos recentes | DIC-GERAL |
| Termos técnicos | admin, root, domain, servidor, suporte, teste, temp | DIC-PRIVILEGIADO |

Nomes próprios entram porque um levantamento da NordPass, divulgado pela imprensa brasileira, apontou nomes como Lucas e Pedro entre os mais encontrados em senhas vazadas no Brasil em 2023 ([fonte](https://www.agazeta.com.br/comentaristas/cbn-e-a-tecnologia/erros-comuns-de-senha-que-os-hackers-adoram-explorar-1223)).

Pontos de cuidado:

- Substituições de letras por símbolos (`S3nh@`) só são barradas se a plataforma normalizar a senha ou se as variações mais comuns forem listadas.
- O dicionário deve ser revisado a cada 6 meses, junto com a política.
- A lista é um ponto de partida. Em projeto real, ela se alimenta de bases de senhas vazadas e de termos específicos do cliente.

## Operação e governança

- A plataforma tem alertas para política de senha fraca ou média e um lembrete de revisão periódica das políticas a cada 6 meses. Vale ligar esses alertas e registrar a revisão como evidência de controle.
- Mudanças em política de senha devem passar por aprovação formal e ficar registradas, pois são parâmetros que auditoria de acesso lógico costuma testar.

## Pontos a validar numa instância real

- Quais atributos entram na verificação de "dados pessoais".
- Se o dicionário aceita importação em lote.
- Uma ou várias listas por política, correspondência exata ou parcial e diferença entre maiúsculas e minúsculas.
- Se a plataforma trata variações com símbolos (`S3nh@`).
- Como a política de senha interage com a política de troca de senha (documento 02).
