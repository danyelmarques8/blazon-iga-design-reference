# 02 — Troca de senha em autosserviço (contas fora do Entra ID)

## Contexto

O Entra ID resolve a troca de senha para as contas que vivem nele. Mas toda empresa tem sistemas que **não participam** do Entra ID: legados, aplicações com base própria de usuários, sistemas de terceiros. Para esses, a plataforma de IGA pode funcionar como ponto único de troca de senha, com fatores de autenticação próprios.

## Fluxo proposto

1. O usuário acessa o portal de troca de senha e informa o nome de usuário.
2. A plataforma avalia as **políticas de troca de senha** em ordem. A primeira cuja elegibilidade bater é aplicada. Se nenhuma bater, vale a política padrão.
3. O usuário comprova a identidade com os fatores da política: perguntas secretas e token. Se faltar o canal do token (e-mail pessoal ou telefone cadastrado), o fluxo para aqui.
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

## O que testei na plataforma

Estes pontos não estão explícitos na documentação pública. Testei na prática e confirmei:

- **Perguntas secretas e token na mesma política:** funciona. Dá para exigir as perguntas junto com o token por e-mail ou SMS, e também junto com o aplicativo autenticador. Isso sustenta o desenho das três políticas acima.
- **Usuário sem canal cadastrado:** se a política exige e-mail pessoal ou telefone e o usuário não tem isso preenchido, o token não é enviado e a troca de senha não é autorizada.

### O que isso muda no desenho

Como a troca depende desses dados de contato, o cadastro deixa de ser detalhe e vira pré-requisito: quem não tem canal cadastrado não consegue trocar a senha sozinho. Por isso proponho:

- Um relatório periódico de usuários sem e-mail pessoal ou telefone cadastrado.
- Preenchimento desses dados já no onboarding, antes do primeiro uso.
- Um caminho de contingência pelo service desk, com verificação de identidade e aprovação registrada, porque esse caminho é alvo comum de engenharia social.
- Aplicativo autenticador configurado com antecedência como alternativa ao canal pessoal.

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

O painel de controle da plataforma tem telas para bloqueios de segurança e para acompanhar as trocas de senha. Pontos de revisão periódica: picos de bloqueios (indício de ataque), usuários bloqueados recorrentemente e usuários sem canal de token cadastrado.

## Riscos e mitigações

| Risco | Mitigação |
|---|---|
| Enumeração de usuários | Bloqueio por origem, mensagens de erro genéricas |
| Conta de e-mail pessoal comprometida | Privilegiados usam apenas o aplicativo autenticador |
| SIM swap | Evitar SMS em contas privilegiadas |
| Respostas descobertas em redes sociais | Banco de perguntas de baixo risco e uso sempre combinado com token |
| Usuário sem canal de token fica sem conseguir trocar a senha | Relatório de cadastros incompletos, preenchimento no onboarding e contingência pelo service desk com verificação de identidade |

## Pontos a validar numa instância real

- Se o campo de expressão da pergunta secreta aceita expressão regular para validar o formato da resposta.
