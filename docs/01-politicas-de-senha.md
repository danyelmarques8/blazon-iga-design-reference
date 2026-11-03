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
| Dicionário | Senhas comuns em português | Senhas comuns + termos da empresa | Não necessário (senha gerada) |
| Histórico | 5 últimas | 12 últimas | 3 últimas |
| Expiração | 90 dias | 60 dias | 365 dias |
| Notificações de expiração | Todas as opções | Todas as opções | Todas as opções |

Os mesmos valores estão em [`data/password-policies.csv`](../data/password-policies.csv).

## Registro de decisões

**D1 — Expiração periódica.** Auditorias de TI costumam testar os parâmetros de senha, e 90 dias é um valor frequente em políticas corporativas. Por outro lado, o NIST SP 800-63B recomenda não impor trocas periódicas sem indício de comprometimento. *Decisão:* manter expiração por exigência de conformidade e registrar o trade-off para reavaliação.

**D2 — Tamanho antes de complexidade.** Na política padrão, não exigi caractere especial para evitar senhas previsíveis do tipo `Senha@123`. O tamanho de 12 e o dicionário compensam.

**D3 — Contas de aplicação.** A senha é gerada pela própria política e não digitada por pessoa, então o tamanho pode ser alto. A expiração longa só faz sentido se existir rotação controlada (cofre de senhas ou processo equivalente).

**D4 — Política padrão obrigatória.** Defini a POL-PADRAO como padrão. Pela documentação, a política padrão não pode ser removida enquanto estiver nessa condição, então qualquer troca de padrão exige antes designar outra.

## Operação e governança

- A plataforma tem alertas para política de senha fraca ou média e um lembrete de revisão periódica das políticas a cada 6 meses. Vale ligar esses alertas e registrar a revisão como evidência de controle.
- Mudanças em política de senha devem passar por aprovação formal e ficar registradas, pois são parâmetros que auditoria de acesso lógico costuma testar.

## Pontos a validar numa instância real

- Quais atributos entram na verificação de "dados pessoais".
- Se o dicionário aceita importação em lote.
- Como a política de senha interage com a política de troca de senha (documento 02).
