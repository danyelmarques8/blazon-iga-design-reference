# 02 — Troca de senha em autosserviço (contas fora do Entra ID)

## Contexto

O Entra ID resolve a troca de senha para contas que vivem nele. Mas toda empresa tem sistemas que **não participam** do Entra ID: legados, aplicações com base própria de usuários, sistemas de terceiros. Para esses, a plataforma de IGA pode funcionar como ponto único de troca de senha, com fatores de autenticação próprios.

## Fluxo proposto

1. O usuário acessa o portal de troca de senha e informa o nome de usuário.
2. A plataforma avalia as **políticas de troca de senha** em ordem. A primeira cuja elegibilidade bater é aplicada. Se nenhuma bater, vale a política padrão.
3. O usuário comprova identidade com os fatores da política (perguntas secretas e token).
4. A nova senha é validada pela **política de senha** (documento 01).
5. A senha é propagada ao sistema de destino pelo conector (resource adapter) e o evento fica registrado.

## Políticas de troca de senha da Acme Fictícia

A ordem importa: da mais restritiva para a mais geral.

| Ordem | Política | Elegibilidade | Perguntas secretas | Token |
|---|---|---|---|---|
| 1 | TSP-PRIVILEGIADO | Usuário com papel de administrador | 3 | Aplicativo autenticador apenas |
| 2 | TSP-RISCO | Rede de origem desconhecida **ou** horário fora de 06:00–20:00 | 3 | SMS |
| 3 | TSP-PADRAO (padrão) | Demais casos | 2 | E-mail corporativo ou pessoal |

Detalhes em [`data/password-reset-policies.csv`](../data/password-reset-policies.csv).

**Por que sem SMS para privilegiados:** SMS é vulnerável a troca de chip (SIM swap). Para contas de alto impacto, o aplicativo autenticador é mais seguro.

## Bloqueios de segurança

A plataforma permite bloquear por tentativa inválida de **usuário**, **token** ou **resposta**, com bloqueio por **origem** (rede) ou por **usuário**, por tempo definido ou até liberação manual do administrador.

| Evento | Bloqueio por | Limite proposto | Duração |
|---|---|---|---|
| Resposta inválida | Usuário | 3 tentativas | 30 min |
| Token inválido | Usuário | 3 tentativas | 30 min |
| Username inválido | Origem | 10 tentativas | 60 min |

O bloqueio por origem para username inválido ajuda contra enumeração de usuários.

## Perguntas secretas: como desenhar

Perguntas secretas são o fator mais fraco do fluxo. Muitas respostas são descobertas em redes sociais ou adivinhadas. Por isso:

- Nunca usar sozinhas, sempre combinadas com token.
- Evitar perguntas com resposta pública ou previsível (nome da mãe, cidade de nascimento, nome do pet).
- Preferir perguntas cuja resposta só o usuário conhece e que não mudam com o tempo.

Banco de perguntas de exemplo:

| Pergunta | Risco de a resposta ser descoberta |
|---|---|
| Qual era o apelido que só a sua família usava na infância? | Baixo |
| Qual foi o primeiro filme que você assistiu no cinema? | Médio |
| Qual o nome do primeiro livro que você leu até o fim? | Médio |
| Qual o nome do seu primeiro chefe? | Alto (pode estar no LinkedIn) |
| Qual o nome do seu animal de estimação? | Alto (pode estar nas redes sociais) |

Recomendação: manter só as de risco baixo e médio no banco.

## Monitoramento

O painel de controle da plataforma tem telas para bloqueios de segurança e para acompanhar as trocas de senha. Pontos de revisão periódica: picos de bloqueios (indício de ataque) e usuários bloqueados recorrentemente.

## Riscos e mitigações

| Risco | Mitigação |
|---|---|
| Enumeração de usuários | Bloqueio por origem, mensagens de erro genéricas |
| Conta de e-mail pessoal comprometida | Privilegiados usam apenas o aplicativo autenticador |
| SIM swap | Evitar SMS em contas privilegiadas |
| Respostas descobertas em redes sociais | Banco de perguntas de baixo risco e uso sempre combinado com token |

## Pontos a validar numa instância real

- Se a plataforma permite **exigir** perguntas **e** token ao mesmo tempo na mesma política.
- Se o campo de expressão da pergunta aceita expressão regular para validar o formato da resposta.
- Como o fluxo se comporta quando o usuário não tem e-mail pessoal ou telefone cadastrado.
