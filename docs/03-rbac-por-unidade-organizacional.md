# 03 — RBAC com atribuição automática por unidade organizacional

## Fonte de verdade e ciclo de vida

O ponto de partida é o **ERP/RH**, que é a fonte autoritativa dos dados do colaborador. O IGA consome esses dados e age a partir deles:

1. O RH cadastra o colaborador no ERP, com departamento, tipo de vínculo e demais atributos.
2. O IGA recebe o cadastro e cria o usuário.
3. O IGA cria a conta no **Active Directory já na OU do departamento**, sem ninguém mexer manualmente no AD.
4. As políticas de atribuição concedem os papéis base automaticamente.
5. Papéis sensíveis entram por solicitação, com aprovação e checagem de SoD.

Quando o RH altera um dado no ERP, o IGA reavalia o usuário e os papéis acompanham a mudança. Ninguém precisa pedir ou revogar acesso na mão.

## Modelo de camadas

| Camada | O que é | Como entra |
|---|---|---|
| 1. Acesso básico corporativo | E-mail, intranet | Automática, para todo colaborador |
| 2. Papel base do departamento | O mínimo que todo membro da área precisa | Automática, por unidade organizacional |
| 3. Papéis de função | Acessos sensíveis (lançar, aprovar, administrar) | **Por solicitação**, com aprovação e checagem de SoD |

Regra de ouro: **automático só para o que é básico**. Papel sensível concedido por regra automática é risco de SoD e dificulta a revisão.

## Unidade organizacional, departamento e OU do AD

O Blazon trabalha com unidades organizacionais de quatro tipos (centro de custo, departamento, grupo de trabalho e organização). Os usuários são ligados a elas pelos atributos de centro de custo, departamento e organização.

Proposta: o **departamento vem do ERP** e define dois destinos, a unidade organizacional no IGA e a OU onde o IGA cria a conta no AD.

| Departamento (origem: ERP) | Unidade organizacional no IGA | OU no AD (fictícia) |
|---|---|---|
| Financeiro | Financeiro | OU=Financeiro,OU=Acme |
| RH | RH | OU=RH,OU=Acme |
| TI | TI | OU=TI,OU=Acme |
| Comercial | Comercial | OU=Comercial,OU=Acme |
| Jurídico | Jurídico | OU=Juridico,OU=Acme |

## Convenção de nomes

Papéis: `ROL-<DEPARTAMENTO>-<FUNÇÃO>`, por exemplo `ROL-FIN-APROVADOR`. Políticas de atribuição: `PA-<NÚMERO>`. Catálogo completo em [`data/roles-catalog.csv`](../data/roles-catalog.csv) e direitos em [`data/role-entitlements.csv`](../data/role-entitlements.csv).

## Políticas de atribuição

Uma política de atribuição define uma **elegibilidade** baseada em atributos do usuário e os **papéis** concedidos a quem se encaixa. O processo roda continuamente sobre os usuários ativos do diretório.

| Política | Condição (combinação "Todas") | Papéis concedidos |
|---|---|---|
| PA-001 | Tipo de vínculo diferente de Terceiro | ROL-CORP-BASE |
| PA-010 | Departamento = Financeiro e vínculo diferente de Terceiro | ROL-FIN-BASE |
| PA-020 | Departamento = RH e vínculo diferente de Terceiro | ROL-RH-BASE |
| PA-030 | Departamento = TI e vínculo diferente de Terceiro | ROL-TI-BASE |
| PA-040 | Departamento = Comercial e vínculo diferente de Terceiro | ROL-COM-BASE |
| PA-050 | Departamento = Jurídico e vínculo diferente de Terceiro | ROL-JUR-BASE |

Versão completa em [`data/assignment-policies.csv`](../data/assignment-policies.csv).

Exemplo de elegibilidade avançada (script BeanShell, criado para este estudo):

```java
// Elegível: colaboradores CLT do Comercial -> ROL-COM-BASE
String dept = user.getDepartment();
Object vinculo = user.get("employeeType");   // atributo customizado fictício

if (dept != null && dept.equals("Comercial")
        && vinculo != null && vinculo.equals("CLT")) {
    return true;
}
return false;
```

### Armadilhas conhecidas

- **Política habilitada sem elegibilidade atribui todos os usuários ativos** aos papéis dela. É o erro mais perigoso. Checklist: nunca habilitar sem revisar a elegibilidade.
- **Mudar a elegibilidade de uma política vigente pode revogar membros** que ela gerencia. Mudança de política deve passar por aprovação e teste prévio.
- Política recém-criada nasce desabilitada. Habilitar é um passo deliberado.

## Cenário de Mover: Financeiro para Comercial

1. O RH altera o departamento do colaborador **no ERP**, que é a fonte de verdade.
2. O IGA recebe a mudança e atualiza o atributo do usuário.
3. As políticas são reavaliadas: a PA-010 deixa de valer e o papel `ROL-FIN-BASE` é revogado. A PA-040 passa a valer e o papel `ROL-COM-BASE` é concedido.
4. Na minha experiência, esse ciclo leva cerca de 2 dias, e o tempo varia conforme a configuração e o ambiente. A concessão dos novos acessos e a revogação dos antigos acontecem em sequência, sem deixar o colaborador sem nenhum acesso no meio da troca.
5. **Papéis concedidos por solicitação (ex.: `ROL-FIN-APROVADOR`) não mudam sozinhos.** Ficam ativos até alguém revisar.

O passo 5 é o risco real de Mover: o acúmulo vem dos papéis manuais.

**Por que o atributo precisa vir do ERP.** Se o departamento fosse alterado direto no AD ou no IGA, haveria duas fontes de verdade e a mudança dependeria de alguém lembrar de fazer. Com o RH alterando no ERP, a mudança de papéis acontece como consequência da mudança de cargo, e o registro do RH fica como evidência da origem.

## Impacto para UAR e SOX

Pela documentação, membros de papéis gerenciados por política **não entram na certificação**. Então:

- Campanhas de UAR focam nos papéis concedidos **por solicitação**, que é onde mora o risco.
- A **política de atribuição em si** precisa ser revisada periodicamente (quem aprovou, qual a elegibilidade, quais papéis concede). Essa revisão vira a evidência de controle para os papéis automáticos.
- Conflitos de SoD ficam em [`data/sod-conflicts.csv`](../data/sod-conflicts.csv) e devem ser checados no momento da solicitação do papel sensível.
- A qualidade do dado no ERP vira controle de acesso: departamento errado no cadastro gera papel errado. Vale um controle de revisão dos cadastros pelo RH.

## Pontos a validar numa instância real

- Quais atributos de usuário a elegibilidade enxerga, incluindo os campos de departamento e organização.
- Se, quando o departamento muda, a conta é movida automaticamente para a nova OU no AD.
