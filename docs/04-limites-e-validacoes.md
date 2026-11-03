# 04 — Limites do projeto e validações pendentes

## O que este projeto é

Um estudo de **design** baseado na documentação pública, com dados fictícios. Mostra como eu estruturaria política de senha, troca de senha e RBAC numa plataforma IGA.

## O que este projeto não é

- Não é uma implementação testada. Não houve execução numa instância do Blazon neste estudo.
- Não reproduz configurações, políticas, matrizes ou nomes de qualquer empresa onde trabalhei.
- Não substitui a documentação oficial nem a validação com o fabricante.

## Lista de validações para uma instância real

| # | Item | Documento |
|---|---|---|
| 1 | Combinação obrigatória de perguntas secretas e token na mesma política | 02 |
| 2 | Formato aceito no campo de expressão da pergunta secreta | 02 |
| 3 | Comportamento quando o usuário não tem e-mail pessoal ou telefone | 02 |
| 4 | Fluxo de preenchimento do departamento a partir da OU do AD | 03 |
| 5 | Atributos visíveis na elegibilidade das políticas de atribuição | 03 |
| 6 | Tempo de reavaliação após mudança de atributo (Mover) | 03 |
| 7 | Interação entre política de senha e política de troca de senha | 01 e 02 |

## Evolução prevista

- Consultas SQL de auditoria sobre os CSVs (papéis órfãos, conflitos de SoD, contas sem dono).
- Versão do catálogo de papéis com ABAC para comparar com o RBAC puro.
