# Blazon IGA

Estudo de design de governança de identidades (IGA) baseado na **documentação pública do Blazon**, aplicado a uma empresa **fictícia** (Acme Fictícia S.A.).

> **Aviso:** todos os dados são fictícios. Este material é um estudo de design, **não** uma configuração testada em produção, e não tem vínculo com a Blazon. Os nomes de produto pertencem aos seus donos.

## O que este projeto cobre

| Frente | Documento |
|---|---|
| Políticas de senha (complexidade, dicionário, histórico, expiração, notificações) | [docs/01-politicas-de-senha.md](docs/01-politicas-de-senha.md) |
| Troca de senha em autosserviço com perguntas secretas e token, para contas fora do Entra ID | [docs/02-troca-de-senha-autosservico.md](docs/02-troca-de-senha-autosservico.md) |
| RBAC com atribuição automática de papéis por unidade organizacional | [docs/03-rbac-por-unidade-organizacional.md](docs/03-rbac-por-unidade-organizacional.md) |
| O que ainda precisa ser validado numa instância real | [docs/04-limites-e-validacoes.md](docs/04-limites-e-validacoes.md) |

Os dados de apoio estão na pasta [`data/`](data/) em CSV (catálogo de papéis, direitos por papel, políticas de atribuição, conflitos de SoD, políticas de senha e de troca de senha).

## Cenário fictício

A Acme Fictícia S.A. tem 5 departamentos (Financeiro, RH, TI, Comercial e Jurídico), um Active Directory organizado por OUs e sistemas fictícios (ERP, CRM, sistema de RH e repositório de contratos). Parte dos sistemas não usa o Entra ID, por isso a troca de senha precisa de um fluxo próprio.

## Principais decisões de design

1. **Três políticas de senha** (padrão, privilegiada e de aplicação), em vez de uma única regra para todo mundo.
2. **Perguntas secretas nunca sozinhas.** São o fator mais fraco, então entram sempre combinadas com token e com bloqueios de segurança.
3. **Atribuição automática só para acessos básicos** (o que todo colaborador do departamento precisa). Papéis sensíveis entram por solicitação com aprovação e verificação de SoD.
4. **Política de atribuição também precisa de revisão.** Membros de papéis gerenciados por política não entram nas campanhas de certificação, então a revisão passa a ser da política em si.

## Fontes

Documentação pública do Blazon: https://docs.blazon.im

Texto e estrutura deste repositório são originais. Não reproduzo a documentação do fabricante nem imagens dela.

## Apoio de IA

Usei IA como apoio para estruturar o material. O conteúdo foi conferido contra a documentação pública acima.
