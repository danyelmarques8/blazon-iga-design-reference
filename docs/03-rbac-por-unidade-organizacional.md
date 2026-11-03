# 03 — RBAC com atribuição automática por unidade organizacional

## Modelo de camadas

| Camada | O que é | Como entra |
|---|---|---|
| 1. Acesso básico corporativo | E-mail, intranet | Automática, para todo colaborador |
| 2. Papel base do departamento | O mínimo que todo membro da área precisa | Automática, por unidade organizacional |
| 3. Papéis de função | Acessos sensíveis (lançar, aprovar, administrar) | **Por solicitação**, com aprovação e checagem de SoD |

Regra de ouro: **automático só para o que é básico**. Papel sensível concedido por regra automática é risco de SoD e dificulta a revisão.

## De OU do AD para unidade organizacional

O Blazon trabalha com unidades organizacionais de quatro tipos (centro de custo, departamento, grupo de trabalho e organização). Os usuários são ligados a elas pelos atributos de centro de custo, departamento e organização.

Proposta: cada OU do AD que representa uma área vira um **departamento** no IGA, preenchido por mapeamento na sincronização.

| OU no AD (fictícia) | Departamento no IGA |
|---|---|
| OU=Financeiro,OU=Acme | Financeiro |
| OU=RH,OU=Acme | RH |
| OU=TI,OU=Acme | TI |
| OU=Comercial,OU=Acme | Comercial |
| OU=Juridico,OU=Acme | Jurídico |

> Premissa de design: o departamento chega ao IGA pela sincronização do AD. Precisa ser confirmado numa instância real (ver documento 04).

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

1. O RH altera o departamento do colaborador.
2. O atributo muda no IGA e as políticas são reavaliadas continuamente.
3. A política PA-010 deixa de valer: o papel `ROL-FIN-BASE` é revogado.
4. A política PA-040 passa a valer: o papel `ROL-COM-BASE` é concedido.
5. **Papéis concedidos por solicitação (ex.: `ROL-FIN-APROVADOR`) não mudam sozinhos.** Ficam ativos até alguém revisar.

O passo 5 é o risco real de Mover: o acúmulo vem dos papéis manuais.

## Impacto para UAR e SOX

Pela documentação, membros de papéis gerenciados por política **não entram na certificação**. Então:

- Campanhas de UAR focam nos papéis concedidos **por solicitação**, que é onde mora o risco.
- A **política de atribuição em si** precisa ser revisada periodicamente (quem aprovou, qual a elegibilidade, quais papéis concede). Essa revisão vira a evidência de controle para os papéis automáticos.
- Conflitos de SoD ficam em [`data/sod-conflicts.csv`](../data/sod-conflicts.csv) e devem ser checados no momento da solicitação do papel sensível.

## Pontos a validar numa instância real

- Quais atributos de usuário a elegibilidade enxerga, incluindo os campos de departamento e organização.
- Como o Mover é tratado quando o atributo muda por sincronização com o AD.
- Quanto tempo leva a reavaliação contínua após uma mudança de atributo.
