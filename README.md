# Blazon IGA

Estudo de governança de identidades (IGA) baseado na **documentação pública do Blazon** e em comportamentos que testei na plataforma, aplicado a uma empresa **fictícia** (Acme Fictícia S.A.).

> **Aviso:** todos os dados são fictícios. Este material é um estudo de IGA, não uma configuração de produção, e não tem vínculo com a Blazon. Os nomes de produto pertencem aos seus donos.

## O que este projeto cobre

| Frente | Documento |
|---|---|
| Políticas de senha (complexidade, dicionário, histórico, expiração, notificações) | [docs/01-politicas-de-senha.md](docs/01-politicas-de-senha.md) |
| Troca de senha em autosserviço com perguntas secretas e token, para contas fora do Entra ID | [docs/02-troca-de-senha-autosservico.md](docs/02-troca-de-senha-autosservico.md) |
| RBAC com atribuição automática de papéis por unidade organizacional | [docs/03-rbac-por-unidade-organizacional.md](docs/03-rbac-por-unidade-organizacional.md) |
| O que foi testado, o que é observação e o que ainda precisa ser validado | [docs/04-limites-e-validacoes.md](docs/04-limites-e-validacoes.md) |

Os dados de apoio estão na pasta [`data/`](data/) em CSV (catálogo de papéis, direitos por papel, políticas de atribuição, conflitos de SoD, políticas de senha e de troca de senha).

## Cenário fictício

A Acme Fictícia S.A. tem 5 departamentos (Financeiro, RH, TI, Comercial e Jurídico), um Active Directory organizado por OUs e sistemas fictícios (ERP, CRM, sistema de RH e repositório de contratos). O ERP/RH é a fonte autoritativa dos dados do colaborador. Parte dos sistemas não usa o Entra ID, por isso a troca de senha precisa de um fluxo próprio.

## Principais decisões de design

1. **Três políticas de senha** (padrão, privilegiada e de aplicação), em vez de uma única regra para todo mundo.
2. **Tamanho e dicionário antes de complexidade.** Na política padrão, não exigi caractere especial. Combinei tamanho mínimo com um dicionário de palavras proibidas, que bloqueia as senhas previsíveis que a regra de complexidade deixaria passar.
3. **A verificação de dados pessoais tem limite.** Ela só enxerga o que o cadastro enviou ao IGA, então o que não está lá (como o nome do pet) não é barrado. Esse ponto está documentado no projeto.
4. **Perguntas secretas nunca sozinhas.** São o fator mais fraco, então entram sempre combinadas com token e com bloqueios de segurança. Como o token depende de e-mail pessoal ou telefone cadastrado, o cadastro desses dados vira pré-requisito de segurança.
5. **O ERP é a fonte autoritativa.** O RH altera o cadastro, o IGA cria a conta no AD já na OU certa e os papéis acompanham a mudança, sem pedido manual.
6. **Atribuição automática só para acessos básicos.** Papéis sensíveis entram por solicitação, com aprovação e verificação de SoD.
7. **A política de atribuição também precisa de revisão.** Membros de papéis gerenciados por política não entram nas campanhas de certificação, então a revisão passa a ser da política em si.

## Fontes

Documentação pública do Blazon: https://docs.blazon.im

Texto e estrutura deste repositório são originais. Não reproduzo a documentação do fabricante nem imagens dela.

## Como fiz

Trabalhei com o Blazon e quis registrar como eu estruturaria essas frentes do zero, usando a documentação pública, o que testei na plataforma e dados fictícios. Usei IA como apoio para organizar o material e revisei tudo contra a documentação. Os pontos testados na prática estão indicados no documento 04.
