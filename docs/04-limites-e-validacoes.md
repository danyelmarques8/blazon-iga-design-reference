# 04 — Limites do projeto e validações

## O que este projeto é

Um estudo de **design** baseado na documentação pública do Blazon e em comportamentos que testei na plataforma, com dados fictícios. Mostra como eu estruturaria política de senha, troca de senha e RBAC numa plataforma IGA.

## O que este projeto não é

- Não é uma implementação completa nem uma configuração de produção. O que testei foi o comportamento de pontos específicos, indicados na tabela abaixo.
- Não reproduz configurações, políticas, matrizes ou nomes de qualquer empresa onde trabalhei.
- Não substitui a documentação oficial nem a validação com o fabricante.

## Como ler a tabela

| Status | Significado |
|---|---|
| **Testado** | Comportamento que testei na plataforma e confirmei |
| **Confirmado** | Capacidade que sei que existe, por uso da plataforma |
| **Observado** | O que vi na prática, mas que pode variar conforme a configuração do ambiente |
| **A validar** | Premissa de design ainda sem confirmação |

## Validações

| # | Item | Documento | Status |
|---|---|---|---|
| 1 | Perguntas secretas combinadas com token (e-mail, SMS ou aplicativo autenticador) na mesma política | 02 | Testado |
| 2 | Usuário sem e-mail pessoal ou telefone cadastrado: o token não é enviado e a troca de senha não é autorizada | 02 | Testado |
| 3 | Criação do usuário no AD pelo IGA, já na OU do departamento | 03 | Confirmado |
| 4 | Tempo entre a mudança de atributo no ERP e a revogação/concessão dos papéis: cerca de 2 dias no que observei, variando conforme a configuração | 03 | Observado |
| 5 | Campo de expressão da pergunta secreta: aceita expressão regular para validar o formato da resposta? | 02 | A validar |
| 6 | Atributos visíveis na elegibilidade das políticas de atribuição, incluindo departamento e organização | 03 | A validar |
| 7 | Conta movida automaticamente para a nova OU quando o departamento muda | 03 | A validar |
| 8 | Atributos usados na verificação de "sem dados pessoais" e importação de dicionário em lote | 01 | A validar |
| 9 | Dicionário: uma ou várias listas por política, correspondência exata ou parcial, diferença entre maiúsculas e minúsculas, variações com símbolos | 01 | A validar |
| 10 | Interação entre a política de senha e a política de troca de senha | 01 e 02 | A validar |

## Evolução prevista

- Consultas SQL de auditoria sobre os CSVs (papéis órfãos, conflitos de SoD, contas sem dono).
- Versão do catálogo de papéis com ABAC para comparar com o RBAC puro.
- CSV com a lista completa do DIC-GERAL, para o projeto mostrar o dicionário em si.
